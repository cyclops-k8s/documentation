# Ansible Kubernetes Devcontainer

A VS Code devcontainer image for working with the [ansible-kubernetes](../ansible-kubernetes/index.md) project.

**Registry**: `quay.io/cyclops-k8s/ansible-kubernetes-devcontainer`

## What's Included

The devcontainer provides a complete development environment with all tools needed to deploy and manage Kubernetes clusters.

### Infrastructure Tools

| Tool | Purpose |
|------|---------|
| Ansible (via pipx) | Configuration management |
| Terraform / OpenTofu | Infrastructure provisioning |
| kubectl | Kubernetes CLI |
| kubectx | Context/namespace switching |
| kustomize | Kubernetes manifest customization |
| Helm | Package manager for Kubernetes |

### Networking & Diagnostics

| Tool | Purpose |
|------|---------|
| bind9 / dnsutils | DNS utilities |
| bridge-utils | Network bridge management |
| iproute2 | IP routing utilities |
| mtr / traceroute | Network path analysis |
| curl / wget | HTTP clients |

### Virtualization

| Tool | Purpose |
|------|---------|
| QEMU | VM management for test environments |
| OVMF | UEFI firmware for VMs |
| cloud-utils | Cloud-init image tools |

### Additional CLIs

Installed via the `update-binaries` script (always fetches latest releases):

- Argo CD CLI
- Calicoctl
- Cilium CLI
- cert-manager CLI
- Kubeseal
- jq / yq

### Ansible Collections

Installed via `ansible-galaxy-requirements.yaml`:

- `ansible.posix`
- `cloud.terraform`
- `community.dns`
- `community.general`

## Entrypoint

On container startup, the entrypoint script:

1. Sets up SSH keys and configuration
2. Configures SSH access to cluster nodes on ports 2021–2026
3. Prepares the environment for Ansible execution

## Build

The image uses a multi-stage Docker build on Ubuntu 24.04:

1. **Builder stage** — installs Ansible, collects galaxy collections, downloads binary tools
2. **Runtime stage** — copies built artifacts into the final image with all system dependencies
