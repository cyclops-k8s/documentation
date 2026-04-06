# Helm

Installs the latest Helm CLI on control plane nodes.

**Hook points:** `pre_configure_control_planes` (install), `pre_upgrade_control_planes` (upgrade)

```yaml
kubernetes_hookfiles:
  pre_configure_control_planes:
    - /path/to/example-hooks/install-helm/pre-configure-control-planes/install-helm.yaml
  pre_upgrade_control_planes:
    - /path/to/example-hooks/install-helm/pre-upgrade-control-planes/upgrade-helm.yaml
```

## Variables

None — always installs the latest version from the official Helm install script.
