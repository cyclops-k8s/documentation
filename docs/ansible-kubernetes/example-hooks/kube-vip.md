# Kube-VIP

Installs Kube-VIP for virtual IP management of Kubernetes services.

**Hook point:** `post_workers`

```yaml
kubernetes_hookfiles:
  post_workers:
    - /path/to/example-hooks/kube-vip/post-workers/kube-vip.yaml
```

## Variables

No Ansible variables — but the manifest file `files/kube-vip.yaml` **must be edited** before use:

| Field (in manifest) | Default | Description |
|---------------------|---------|-------------|
| `cidr-global` (ConfigMap) | `10.3.0.128/25` | **Must be changed** to your VIP subnet for LoadBalancer services |

## Details

The DaemonSet runs Kube-VIP v0.8.3 on control-plane nodes with:

- ARP mode enabled
- Leader election
- DNS mode (first)
- Service LoadBalancer support
- Prometheus metrics on port 2112
