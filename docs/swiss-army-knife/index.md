# Swiss Army Knife

A collection of networking and diagnostic tools packaged as container images for Kubernetes cluster troubleshooting.

## Variants

### Lite (`Dockerfile.lite`)

Minimal image for quick debugging.

**Base**: Ubuntu 26.04

**Included tools**: `curl`, `wget`, `jq`, `openssl`, `yq`

```bash
kubectl run debug --image=quay.io/cyclops-k8s/swiss-army-knife:lite -it --rm -- bash
```

### Heavy (`Dockerfile.heavy`)

Full-featured image with comprehensive networking and Kubernetes tools.

**Base**: Ubuntu 26.04

#### Networking Tools

| Category | Tools |
|----------|-------|
| DNS | dnsutils |
| Packet capture | tcpdump, ngrep |
| Connectivity | netcat-openbsd, telnet, ssh-client, socat |
| Traffic analysis | iperf, iperf3, iftop, ifstat |
| Routing | iproute2, ipset, iptables, arptables |
| Diagnostics | mtr, arping, traceroute (via mtr) |
| Multiplexer | tmux |

#### Kubernetes Tools

| Tool | Source |
|------|--------|
| kubectl | Latest stable release |
| kustomize | Official hack install script |

#### System Tools

`ca-certificates`, `curl`, `jq`, `openssl`, `wget`, `yq`

```bash
kubectl run debug --image=quay.io/cyclops-k8s/swiss-army-knife:heavy -it --rm -- bash
```

## Use Cases

- Debug DNS resolution inside a cluster
- Capture and analyze network traffic between pods
- Test connectivity to services and external endpoints
- Benchmark network throughput with iperf/iperf3
- Inspect iptables rules and IP routing
- Run kubectl commands from within the cluster network
