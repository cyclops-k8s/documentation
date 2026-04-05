# Hooks

Hooks are Ansible task files imported at specific lifecycle points during cluster installation and upgrades. They allow you to extend the playbook without modifying the core roles.

!!! note
    Hooks are intended for **bootstrapping** tasks only. For ongoing cluster management, use GitOps tools like ArgoCD or Flux.

## Hook Configuration

Define hooks in your inventory or group_vars via the `kubernetes_hookfiles` dictionary:

```yaml
kubernetes_hookfiles:
  post_cluster_init:
    - "{{ inventory_dir }}/hooks/install-cilium.yaml"
    - "{{ inventory_dir }}/hooks/add-admin-rolebinding.yaml"
  post_workers:
    - "{{ inventory_dir }}/hooks/bootstrap-argocd.yaml"
```

## Hook Lifecycle Points

### Installation Hooks

| Hook | When it runs | Typical use |
|------|-------------|-------------|
| `pre_proxies` | Before proxy configuration | Pre-proxy setup |
| `post_proxies` | After proxy configuration | Post-proxy validation |
| `default_configuration` | Beginning of every major section | Default variables, validations |
| `pre_prerequisites` | Before installing packages | Custom package sources |
| `post_install_packages` | After K8s packages installed | Package customization |
| `pre_configure_control_planes` | Before control plane config files | Run proxies on control planes |
| `post_configure_control_planes` | After control plane config files | Config file customization |
| `post_configure_workers` | After worker config files | Worker config customization |
| `pre_control_planes` | Before kubeadm init/join | Pre-cluster tasks |
| `post_cluster_init` | After cluster init, before security hardening | **CNI, CPI installation** |
| `post_security` | After security policies applied | Post-security tasks |
| `post_control_plane_join` | After each control plane joins | Per-node control plane tasks |
| `post_control_planes` | After all control planes joined | **Kubeconfig setup, Helm charts** |
| `post_worker_join` | After each worker joins | Per-node worker tasks |
| `post_workers` | After all workers joined | **ArgoCD, app bootstrapping** |

### Upgrade Hooks

| Hook | When it runs | Typical use |
|------|-------------|-------------|
| `pre_upgrade_control_planes` | Before any control plane upgrade | Pre-upgrade checks |
| `post_upgrade_control_planes` | After control plane kubeadm upgrade | Post-upgrade validation |
| `pre_upgrade_workers` | Before any worker upgrade | Pre-upgrade checks |
| `post_upgrade_workers` | After worker kubeadm upgrade | Post-upgrade validation |
| `post_upgrade` | After kubelet/kubectl/containerd upgraded | Final upgrade tasks |

## Execution Context

- **Post-cluster-init hooks** run on the first control plane node
- **Post-control-plane-join hooks** run on the control plane that was just added
- **Post-control-planes hooks** run on each control plane node
    - Use `run_once: true` to run only once
    - Use `delegate_to: "{{ first_kube_control_plane }}"` for tasks needing `helm` or `kustomize` (installed on the first control plane)
- **Post-workers hooks** run across worker nodes

## Example Hooks

The `example-hooks/` directory contains ready-to-use hooks:

### CNI Plugins

| Hook | Directory | When | Description |
|------|-----------|------|-------------|
| **Cilium** | `cilium/` | `post_cluster_init` | Cilium CNI with optional BGP control plane and Hubble |
| **Calico** | `install-calico/` | `post_cluster_init` | Calico CNI |

### Cluster Components

| Hook | Directory | When | Description |
|------|-----------|------|-------------|
| **ArgoCD** | `argocd/` | `post_workers` | Bootstrap ArgoCD for GitOps |
| **Sealed Secrets** | `sealed-secrets/` | `post_workers` | Bitnami Sealed Secrets controller |
| **Kube-VIP** | `kube-vip/` | `post_workers` | Virtual IP for services |
| **vSphere CPI** | `vsphere-cpi/` | `post_cluster_init` | vSphere Cloud Provider Interface |
| **Kubelet CSR Approver** | `kubelet-csr-approver/` | `post_control_planes` | Automated CSR approval |

### Tools

| Hook | Directory | When | Description |
|------|-----------|------|-------------|
| **Helm** | `install-helm/` | varies | Install Helm CLI |
| **Kustomize** | `install-kustomize/` | varies | Install Kustomize CLI |
| **crun** | `install-crun/` | varies | Install crun OCI runtime |

### Configuration

| Hook | Directory | When | Description |
|------|-----------|------|-------------|
| **Admin Role Binding** | `add-adminrolebinding/` | `post_cluster_init` | OIDC admin ClusterRoleBinding |
| **Copy Admin Config** | `copy-admin-config/` | `post_cluster_init` | Copy kubeconfig locally |
| **Local Kubeconfig (Azure)** | `local-kubeconfig-azure/` | `post_control_planes` | Azure OIDC kubeconfig |
| **Local Kubeconfig (int128)** | `local-kubeconfig-int128/` | `post_control_planes` | int128 OIDC kubeconfig |
| **Containerd Config** | `containerd-config-customization/` | `pre_prerequisites` | Custom containerd settings (e.g. KubeVirt) |
| **Registry Mirrors** | `registry-mirrors/` | `post_proxies` | Registry mirror configuration |
| **Proxy on Control Planes** | `proxy-on-control-planes/` | `pre_configure_control_planes` | Run HAProxy/Keepalived on control plane nodes |
| **Etcd Backup** | `etcd-backup/` | `post_control_planes` | Etcd snapshot backups |

## Writing Custom Hooks

A hook is a standard Ansible tasks file:

```yaml
# hooks/my-custom-hook.yaml
- name: Install my application
  ansible.builtin.shell: |
    helm install my-app my-repo/my-app --namespace my-app --create-namespace
  delegate_to: "{{ first_kube_control_plane }}"
  run_once: true
  when: "'my-app' not in existing_releases.stdout"
```

Register it in your configuration:

```yaml
kubernetes_hookfiles:
  post_workers:
    - "{{ inventory_dir }}/hooks/my-custom-hook.yaml"
```

!!! tip "Idempotency"
    Always check whether your hook has already been applied before taking action. This ensures re-running the playbook (e.g. to add nodes) doesn't break anything.
