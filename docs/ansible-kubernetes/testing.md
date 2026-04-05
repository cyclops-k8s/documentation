# Testing

The ansible-kubernetes project includes a full test environment using QEMU virtual machines.

## Local Test Environment

### Requirements

- **Memory**: 24 GB minimum (swap is acceptable)
- **CPU**: QEMU with KVM acceleration
- **Software**: QEMU, cloud-init utilities, Terraform/OpenTofu

### VM Layout

| Name | IP | Role |
|------|-----|------|
| `px.k8s.local` | 10.255.254.11 | Proxy (HAProxy + Keepalived) |
| `cp1.k8s.local` | 10.255.254.12 | Control plane 1 |
| `cp2.k8s.local` | 10.255.254.13 | Control plane 2 |
| `cp3.k8s.local` | 10.255.254.14 | Control plane 3 |
| `w1.k8s.local` | 10.255.254.15 | Worker node 1 |
| `w2.k8s.local` | 10.255.254.16 | Worker node 2 |

### Setting Up

1. Spin up the QEMU VMs:

    ```bash
    cd test/
    ./spin-up-test-environment.sh
    ```

    This creates VMs using cloud-init for initial configuration. Default OS is Ubuntu 24.04; Ubuntu 25.10 and CentOS 9/10 are also supported.

2. Run the install:

    ```bash
    ./install.sh
    ```

    This runs `terraform apply` to create the Ansible inventory, then executes `install.yaml`.

3. Test an upgrade:

    ```bash
    ./upgrade.sh
    ```

4. Reset the cluster:

    ```bash
    ./reset.sh
    ```

### Test Configuration

Test-specific variables are in `test/vars.yaml`:

- Calico CNI hook configuration
- Registry mirror definitions
- Kubelet config patches
- `unsafe-no-fsync` for etcd (test performance only — never use in production)

### Terraform Configuration

`test/main.tf` uses the `ansible/ansible` Terraform provider to generate the Ansible inventory. It automatically:

- Creates host definitions for all VMs
- Generates random encryption keys and VRRP passwords
- Configures hook file references

## CI/CD Test Environment

The CI/CD pipeline uses KubeVirt-based VMs running on an existing Kubernetes cluster.

### CI VM Naming

VMs are prefixed with `gh-` followed by the GitHub Actions run number and a random suffix.

### CI Scripts

| Script | Purpose |
|--------|---------|
| `ci-cd/test/install-dependencies.sh` | Install Ansible, collections, OpenTofu, kubectl, jq, yq |
| `ci-cd/test/install.sh` | Run Terraform and the install playbook |
| `ci-cd/test/verify-cluster-health.sh` | Cluster health checks |
| `ci-cd/test/run-smoke-tests.sh` | Basic cluster validation |
| `ci-cd/test/run-conformance-tests.sh` | Sonobuoy CNCF conformance tests |
| `ci-cd/test/collect-logs.sh` | Collect logs for debugging |
| `ci-cd/test/shutdown-test-environment.sh` | Clean up VMs |

### CI Infrastructure

- **Storage class**: `cyclops-block`
- **Image cache**: `http://assets.cyclops-assets/os-images/`
- **Conformance testing**: Sonobuoy (optional)
