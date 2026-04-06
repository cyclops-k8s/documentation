# Registry Mirrors

Deploys container registry mirrors on proxy nodes using the Docker `registry` image.

**Hook point:** `post_proxies`

```yaml
kubernetes_hookfiles:
  post_proxies:
    - /path/to/example-hooks/registry-mirrors/post-proxies/add-containerd-mirrors.yaml
```

## Variables

| Variable | Required | Default | Type | Description |
|----------|----------|---------|------|-------------|
| `registry_mirrors` | Yes | — | list | Registry mirror definitions (see below) |
| `registry_mirror_config_path` | Yes | — | path | Directory on proxies where mirror config files are stored |
| `registry_mirror_port` | No | `5000` | int | Port passed to the mirror containers |
| `proxy_mirrors` | No | — | list | Upstream mirror for the Distribution Registry image itself |
| `kubernetes_proxy_haproxy_config_file` | No | (override) | path | Must be overridden to use the included `haproxy.cfg` for routing |

## `registry_mirrors` Structure

```yaml
registry_mirrors:
  - registry: docker.io
    data_path: /data/registry/docker.io    # Full path for data storage
    port: 5001                             # Must be unique per mirror
    remote_url: https://registry-1.docker.io  # Upstream URL (default: https://<registry>)
    username: myuser                       # Optional auth
    password: mypassword                   # Optional auth
    ttl: 336h                              # Cache TTL; 0 to disable pruning (default: 336h / 2 weeks)
```

## `proxy_mirrors` Structure

Optional — for mirroring the Distribution Registry image itself:

```yaml
proxy_mirrors:
  - registry: docker.io
    endpoints:
      - https://my.localserver.me:5001
```

## Notes

- You **must** override `kubernetes_proxy_haproxy_config_file` to use the included HAProxy config that routes requests to the correct mirror
- This hook can also be used with containerd 2.2.0+, which no longer supports registry credentials in the config file — the mirror handles upstream authentication instead
