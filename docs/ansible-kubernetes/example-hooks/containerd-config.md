# Containerd Config Customization

Enables `device_ownership_from_security_context` in containerd for CDI/KubeVirt support.

**Hook points:** `pre_prerequisites` (install), `post_upgrade` (upgrade)

```yaml
kubernetes_hookfiles:
  pre_prerequisites:
    - /path/to/example-hooks/containerd-config-customization/hook.yaml
  post_upgrade:
    - /path/to/example-hooks/containerd-config-customization/hook.yaml
```

## Variables

None.

## What It Does

- Adds `device_ownership_from_security_context = true` to `/etc/containerd/config.toml`
- Restarts containerd (safe to do without draining)
- Waits 10 seconds for the restart to complete

## Recommended Hook Points

- `pre_prerequisites` — after containerd config is written during install, before control plane setup
- `post_upgrade` — after containerd is upgraded and reconfigured, before the node is uncordoned

Containerd can safely be restarted on a node without draining, so this hook can be placed at either point.
