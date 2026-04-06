# crun

Installs the crun OCI runtime to replace the default runc. Supports zero-downtime upgrades by keeping previous versions.

**Hook points:** `pre_prerequisites` (install), `post_upgrade` (upgrade)

```yaml
kubernetes_hookfiles:
  pre_prerequisites:
    - /path/to/example-hooks/install-crun/containerd-hook.yaml
  post_upgrade:
    - /path/to/example-hooks/install-crun/containerd-hook.yaml
```

## Variables

| Variable | Required | Default | Type | Description |
|----------|----------|---------|------|-------------|
| `crun_version` | No | `latest` | string | GitHub release version (e.g. `"1.14.1"`). See [crun releases](https://github.com/containers/crun/releases) |
| `crun_install_path` | No | `/opt/crun` | path | Directory to install crun binaries |

## Notes

- Supports architectures: x86_64, aarch64, ppc64le, riscv64, s390x
- Links to `/usr/local/bin/runc` as the default runtime
- Removes `/usr/bin/runc` to avoid conflicts
- Previous versions are kept for zero-downtime upgrades
- Linux only
