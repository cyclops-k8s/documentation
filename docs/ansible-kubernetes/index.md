# Ansible Kubernetes

An Ansible playbook for deploying vanilla Kubernetes clusters hardened against the **CIS Benchmark 1.12** and **DOD STIG V2R1**.

## Features

- **Vanilla kubeadm** — produces a standard cluster manageable with `kubeadm` going forward
- **High availability** — HAProxy + Keepalived load-balanced control plane
- **Security by default** — OIDC authentication, etcd encryption at rest (AES-GCM), audit logging, Pod Security Standards, RBAC, strong TLS cipher suites
- **Multi-OS support** — automatic detection of Debian (Ubuntu 24.04 LTS, 25.10) and RPM-based (CentOS Stream 9, 10) distributions
- **Extensible hooks** — 20+ lifecycle hook points for CNI, CPI, CSI, and any custom tasks
- **Automatic certificate renewal** — systemd timer-based monthly renewal
- **Version locking** — prevents accidental upgrades via `dpkg` hold (Debian) or `dnf versionlock` (RedHat)
- **Supported Kubernetes versions** — 1.33, 1.34, 1.35

!!! warning "CentOS Stream 9"
    As of February 2026, CentOS Stream 9 has stability issues with Kubernetes 1.35 on fresh installs. Upgrading from an earlier version to 1.35 works fine.

## Playbooks

| Playbook | Purpose |
|----------|---------|
| `install.yaml` | Full cluster installation |
| `upgrade.yaml` | Rolling cluster upgrade |
| `reset.yaml` | Complete cluster teardown |
| `recover-expired-kubelet-certs.yaml` | Recover from expired kubelet certificates |

## Architecture

```
                    ┌─────────────────────┐
                    │   Clients / kubectl │
                    └─────────┬───────────┘
                              │
                    ┌─────────▼───────────┐
                    │  Keepalived (VIP)   │
                    └─────────┬───────────┘
                              │
                    ┌─────────▼───────────┐
                    │  HAProxy (proxies)  │
                    └──┬──────┬───────┬───┘
                       │      │       │
              ┌────────▼┐ ┌───▼────┐ ┌▼────────┐
              │  CP #1  │ │  CP #2 │ │  CP #3  │
              │ (init)  │ │ (join) │ │ (join)  │
              └─────────┘ └────────┘ └─────────┘
                       │      │       │
              ┌────────▼──────▼───────▼────────┐
              │  Worker Nodes (worker_nodes)   │
              └────────────────────────────────┘
```

## Quick Start

1. Install the required Ansible collections:

    ```bash
    ansible-galaxy collection install -r requirements.yaml
    pip install dnspython
    ```

2. Create your inventory with four groups: `proxies`, `kubernetes`, `control_planes`, `worker_nodes`

3. Set the [required configuration variables](configuration.md#required-variables)

4. Run the install playbook:

    ```bash
    ansible-playbook -i inventory install.yaml
    ```

See [Getting Started](getting-started.md) for detailed setup instructions.
