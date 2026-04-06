# Proxy on Control Planes

Runs HAProxy and Keepalived directly on control plane nodes instead of dedicated proxy nodes. Eliminates the need for separate proxy infrastructure.

**Hook point:** `pre_control_planes`

```yaml
kubernetes_hookfiles:
  pre_control_planes:
    - /path/to/example-hooks/proxy-on-control-planes/pre_control_planes/proxy-on-control-planes.yaml
```

## Variables

| Variable | Required | Default | Type | Description |
|----------|----------|---------|------|-------------|
| `vrrp_interface` | Yes | (inherited) | string | Network interface for Keepalived VIP |
| `vrrp_password` | Yes | (inherited) | string | VRRP authentication password |
| `vrrp_virtual_router_id` | Yes | (inherited) | int | VRRP virtual router ID |
| `vrrp_state` | No | `BACKUP` | string | Initial Keepalived state (`MASTER` or `BACKUP`) |
| `kubernetes_control_plane_ip` | Yes | (inherited) | IP | Virtual IP address (bound as /24) |
| `kubernetes_proxy_port` | No | `6443` | int | HAProxy listen port |
| `kubernetes_proxy_bind_address` | No | `""` | IP | HAProxy bind address |
| `kubernetes_control_plane_check_interval` | No | `250ms` | duration | Health check interval |
| `kubernetes_api_server_proxy_image` | No | `haproxy` | string | HAProxy container image |
| `kubernetes_api_server_proxy_image_tag` | No | `lts` | string | HAProxy image tag |
| `kubernetes_api_server_proxy_image_pull_policy` | No | `Always` | string | Image pull policy |

## What It Does

1. Runs `pre_proxies` hooks (if defined)
2. Validates required VRRP variables
3. Deploys HAProxy as a static pod at `/etc/kubernetes/manifests/kube-apiserver-proxy.yaml`
4. Installs and configures Keepalived for VIP failover
5. Sets `net.ipv4.ip_nonlocal_bind=1` sysctl
6. Runs `post_proxies` hooks (if defined)

## HAProxy Static Pod

- **CPU:** 250m request / 500m limit
- **Memory:** 100Mi request / 200Mi limit
- **Health checks:** `:1936/healthz` (liveness and readiness)
- **Stats:** Enabled on `:1936`
- **Backend:** TCP mode load balancing to all control plane nodes, health checking `/readyz`
