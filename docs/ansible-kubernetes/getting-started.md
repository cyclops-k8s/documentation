# Getting Started

## Prerequisites

### Ansible Collections

Install the required collections from the `requirements.yaml` file:

```bash
ansible-galaxy collection install -r requirements.yaml
```

This installs:

- `cloud.terraform`
- `community.general`
- `ansible.posix` (required for RPM-based systems)
- `community.dns`

### Python Packages

```bash
pip install dnspython
```

### Supported Operating Systems

**Debian-based:**

- Ubuntu 24.04 LTS
- Ubuntu 25.10

**RPM-based:**

- CentOS Stream 9
- CentOS Stream 10

The playbook automatically detects the OS family and uses the appropriate package manager.

## Inventory Setup

Create an inventory with four host groups:

| Group | Purpose |
|-------|---------|
| `proxies` | Nodes running HAProxy + Keepalived to load-balance the control plane |
| `kubernetes` | All Kubernetes nodes (control planes + workers) |
| `control_planes` | Control plane nodes (minimum 3 recommended for HA) |
| `worker_nodes` | Worker nodes |

### Example Inventory

```yaml
all:
  children:
    proxies:
      hosts:
        proxy-1:
          ansible_host: 10.0.0.10
          vrrp_state: MASTER
        proxy-2:
          ansible_host: 10.0.0.11
          vrrp_state: BACKUP
    kubernetes:
      children:
        control_planes:
          hosts:
            cp-1:
              ansible_host: 10.0.0.20
            cp-2:
              ansible_host: 10.0.0.21
            cp-3:
              ansible_host: 10.0.0.22
        worker_nodes:
          hosts:
            worker-1:
              ansible_host: 10.0.0.30
            worker-2:
              ansible_host: 10.0.0.31
```

## Required Configuration

Set these variables in `group_vars/all.yml` or pass them via `-e`:

```yaml
# IP for HAProxy/Keepalived floating VIP
kubernetes_control_plane_ip: 10.0.0.100

# FQDN for API access — DNS must resolve before running
kubernetes_api_endpoint: k8s-api.example.com

# 32-byte base64-encoded encryption key for etcd at-rest encryption
# Generate: head -c 32 /dev/urandom | base64
kubernetes_encryption_key: "YOUR_BASE64_KEY_HERE"

# OIDC configuration
kubernetes_oidc_issuer_url: https://login.example.com/
kubernetes_oidc_client_id: your-client-id

# Keepalived / VRRP
vrrp_interface: eth0
vrrp_password: "your-vrrp-password"
vrrp_virtual_router_id: 51
```

## Running the Playbook

### Install

```bash
ansible-playbook -i inventory install.yaml
```

### Adding Nodes

Add new nodes to your inventory and re-run the install playbook. Existing nodes are not disrupted:

```bash
ansible-playbook -i inventory install.yaml
```

!!! tip
    Your hooks should be idempotent — check whether they've already been applied before taking action.

## Install Playbook Flow

The `install.yaml` playbook executes in this order:

1. **Configure proxies** — installs HAProxy and Keepalived on proxy nodes
2. **Install container runtime** — installs containerd on all Kubernetes nodes
3. **Install Kubernetes prerequisites** — kernel modules, sysctl, packages, swap disable
4. **Pre-control-plane hooks** — user-defined tasks before control plane setup
5. **Initialize/join control planes** — `kubeadm init` on first node, `kubeadm join` on others (serial:1)
6. **Post-control-plane hooks** — user-defined tasks after all control planes are ready
7. **Join worker nodes** — `kubeadm join` for workers
8. **Post-worker hooks** — user-defined tasks after all workers are ready
9. **Post-finalization** — CoreDNS rebalancing, etcd encryption verification

## Next Steps

- [Configuration Reference](configuration.md) — full list of all configuration variables
- [Hooks](hooks.md) — extend the playbook with custom tasks
- [Upgrading](upgrading.md) — rolling cluster upgrades
- [Compliance](compliance.md) — CIS Benchmark and STIG details
