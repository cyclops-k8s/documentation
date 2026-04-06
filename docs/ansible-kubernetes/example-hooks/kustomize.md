# Kustomize

Installs the latest Kustomize CLI on control plane nodes.

**Hook points:** `pre_configure_control_planes` (install), `pre_upgrade_control_planes` (upgrade)

```yaml
kubernetes_hookfiles:
  pre_configure_control_planes:
    - /path/to/example-hooks/install-kustomize/pre-configure-control-planes/install-kustomize.yaml
  pre_upgrade_control_planes:
    - /path/to/example-hooks/install-kustomize/pre-upgrade-control-planes/upgrade-kustomize.yaml
```

## Variables

None — always installs the latest version from the official Kustomize install script.
