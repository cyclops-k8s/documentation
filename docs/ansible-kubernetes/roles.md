# Roles

The playbook is organized into Ansible roles, each responsible for a specific stage of the cluster lifecycle.

## Role Execution Order

```
kubernetes-proxy          → HAProxy + Keepalived on proxy nodes
container-runtime         → containerd on all K8s nodes
kubernetes                → Prerequisites (kernel, packages, swap)
  └─ kubernetes-defaults  → Default variable definitions
pre-kubernetes-control-plane  → Pre-control-plane hooks
kubernetes-control-plane  → kubeadm init / join for control planes
post-kubernetes-control-plane → Post-control-plane hooks
kubernetes-worker         → kubeadm join for workers
post-kubernetes-worker    → Post-worker hooks
post-kubernetes-finalization  → CoreDNS rebalance, encryption verification
```

## kubernetes-defaults

Defines all default configuration variables used by the other roles. This role has no tasks — it only provides `defaults/main.yml`.

See the full [Configuration Reference](configuration.md).

## kubernetes-proxy

Installs and configures HAProxy and Keepalived on the `proxies` group nodes.

**Tasks:**

- Install HAProxy and Keepalived packages
- Template `haproxy.cfg` — load balances TCP traffic to all control plane nodes on port 6443
- Template `keepalived.conf` — manages the floating VIP (`kubernetes_control_plane_ip`)
- Configure SELinux on RPM-based systems (when `kubernetes_configure_selinux: true`)

**Handlers:**

- Restart HAProxy
- Restart Keepalived

!!! tip "Running proxies on control planes"
    If you don't want separate proxy nodes, use the `proxy-on-control-planes` example hook to run HAProxy/Keepalived directly on the control plane nodes.

## container-runtime

Installs and configures the container runtime on all nodes in the `kubernetes` group.

**Tasks:**

- Add Docker's APT/RPM repository for containerd
- Install containerd at the specified version (or latest)
- Generate `/etc/containerd/config.toml` with `SystemdCgroup = true`
- Configure registry mirrors (supports both containerd < 2.2.0 and >= 2.2.0 formats)
- Hold/lock containerd package version to prevent accidental upgrades

**Key configuration:**

- `kubernetes_containerd_version` — pin a specific version or use `"latest"`
- `kubernetes_containerd_registry_mirrors` — configure pull-through caches

## kubernetes

Prepares all Kubernetes nodes by installing prerequisites.

**Sub-tasks:**

| Task file | Purpose |
|-----------|---------|
| `configure-kernel.yml` | Loads `overlay` and `br_netfilter` modules, sets sysctl (`net.bridge.bridge-nf-call-iptables`, `net.ipv4.ip_forward`) |
| `configure-swap.yml` | Disables swap (required by kubelet) |
| `install-packages.yml` | Installs `kubelet`, `kubectl`, `kubeadm` with version locking |
| `define-first-kube-control.yml` | Auto-detects or sets the first control plane node for `kubeadm init` |

**Hooks executed:**

- `pre_prerequisites`
- `post_install_packages`

## kubernetes-control-plane

Initializes and joins control plane nodes.

### First Control Plane

1. Templates kubeadm configuration files:
    - `kubeadm-config.yaml` — ClusterConfiguration and InitConfiguration
    - `encryption-config.yaml` — etcd encryption at rest
    - `audit-policy.yaml` — API server audit rules
    - `admission-configuration.yaml` — Pod Security Standards
    - `authentication-config.yaml` — OIDC configuration
    - `authorization-config.yaml` — authorization chain

2. Runs `kubeadm init` with retry logic

3. Post-initialization hardening:
    - Patches default service account (disables auto-mount)
    - Creates `.kube/config` symlink
    - Applies CIS hardening tasks

4. Sets up systemd timer for automatic certificate renewal (monthly)

### Secondary Control Planes

- Generates join token and certificate key on the first control plane
- Runs `kubeadm join` with `--control-plane` flag
- Approves CSRs (6 retry passes)

**Hooks executed:**

- `pre_configure_control_planes`
- `post_configure_control_planes`
- `post_cluster_init`
- `post_security`
- `post_control_plane_join`
- `post_control_planes`

## kubernetes-worker

Joins worker nodes to the cluster.

**Tasks:**

1. Generates join token on the first control plane
2. Applies kubeadm patches on the worker node
3. Runs `kubeadm join`
4. Approves CSRs (6 retry passes)

**Hooks executed:**

- `post_configure_workers`
- `post_worker_join`
- `post_workers`

## post-kubernetes-finalization

Runs after all nodes have joined the cluster.

**Tasks:**

- **CoreDNS rebalancing** — deletes CoreDNS pods so the scheduler redistributes them across the new nodes
- **Encryption verification** — creates a test secret in etcd and reads it back via `etcdctl` to confirm encryption at rest is active

## pre-kubernetes-control-plane / post-kubernetes-control-plane / post-kubernetes-worker

These roles are thin wrappers that execute the corresponding hook points (`pre_control_planes`, `post_control_planes`, `post_workers`) defined in `kubernetes_hookfiles`.
