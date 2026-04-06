# Calico

Installs Calico CNI from the upstream manifest. Includes an optional pre-requisites hook for RHEL-based systems.

**Hook points:** `post_cluster_init`, `pre_prerequisites` (RHEL only)

```yaml
kubernetes_hookfiles:
  pre_prerequisites:
    - /path/to/example-hooks/install-calico/pre-prerequisites/configure-rhel-for-calico.yaml  # RHEL only
  post_cluster_init:
    - /path/to/example-hooks/install-calico/post-cluster-init/install-calico.yaml
```

## Variables

| Variable | Required | Default | Type | Description |
|----------|----------|---------|------|-------------|
| `calico_manifest_url` | No | Calico v3.31.3 manifest | URL | Override manifest URL for a different Calico version |

## RHEL Pre-requisites

The `configure-rhel-for-calico.yaml` hook (used in `pre_prerequisites`) performs RHEL-specific configuration:

- Disables `firewalld` if installed
- Configures NetworkManager to ignore Calico interfaces (`cali*`, `tunl*`, `vxlan.calico`, etc.)
