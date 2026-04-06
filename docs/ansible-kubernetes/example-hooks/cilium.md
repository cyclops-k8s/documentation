# Cilium

Installs the Cilium CNI via Helm with kube-proxy replacement, WireGuard encryption, Hubble observability, and optional BGP support.

**Hook point:** `post_cluster_init`

```yaml
kubernetes_hookfiles:
  post_cluster_init:
    - /path/to/example-hooks/cilium/post-cluster-init/install-cilium.yaml
```

## Variables

| Variable | Required | Default | Type | Description |
|----------|----------|---------|------|-------------|
| `kubernetes_api_endpoint` | Yes | (inherited) | FQDN | Kubernetes API server host |
| `kubernetes_api_port` | Yes | (inherited) | int | Kubernetes API server port |
| `kubernetes_cilium_version` | No | latest | semver | Helm chart version. Leave empty for latest |
| `kubernetes_cilium_bgpControlPlane_enabled` | No | `false` | boolean | Enable BGP CRDs for virtual BGP routers |
| `kubernetes_cilium_clusterPoolIPv4PodCIDR` | No | `10.0.0.0/8` | CIDR | IPv4 pod CIDR range for IPAM |
| `kubernetes_cilium_clusterPoolIPv4MaskSize` | No | `24` | int (0-32) | IPv4 per-node CIDR mask size |
| `kubernetes_cilium_clusterPoolIPv6PodCIDR` | No | `fd00::/104` | CIDR | IPv6 pod CIDR range for IPAM |
| `kubernetes_cilium_clusterPoolIPv6MaskSize` | No | `120` | int (0-128) | IPv6 per-node CIDR mask size |
| `kubernetes_cilium_devices` | No | — | string | Space-separated network interfaces for eBPF datapath (e.g. `"br0 br1"`) |
| `kubernetes_cilium_hubble_fqdn` | No | `chart-example.local` | FQDN | Hubble UI FQDN. Setting this enables the Hubble ingress |
| `kubernetes_cilium_hubble_ingressClassName` | No | `cilium` | string | Ingress class for Hubble UI |

## Recommendations

1. Leave `kubernetes_cilium_version` empty to always get the latest chart version
2. Set `kubernetes_cilium_bgpControlPlane_enabled: true` to install BGP CRDs
3. Set `kubernetes_cilium_devices` to the bond or bridge interface on your nodes (must be the same name across all nodes)

## Enabled Features

- kube-proxy replacement
- IPAM cluster-pool mode
- Hubble metrics (DNS, drop, flow, ICMP, port-distribution, TCP), relay, and UI
- WireGuard node-to-node encryption
- PMTU discovery
- Prometheus metrics
