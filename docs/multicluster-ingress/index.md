# MultiCluster Ingress

A .NET application that provides multi-cluster Kubernetes DNS resolution and Global Server Load Balancing (GSLB).

## Overview

Vecc.K8s.MultiCluster is a Kubernetes operator/orchestrator that watches Ingress, Service, and custom GSLB resources across multiple clusters and provides unified DNS resolution via a CoreDNS gRPC plugin.

## Architecture

The application runs in four modes, each deployable independently:

| Mode | Flag | Purpose | Leader Election |
|------|------|---------|-----------------|
| **Operator** | `--operator` | Watches K8s resources (Ingress, Service, EndpointSlice, GSLB) | Single leader |
| **Orchestrator** | `--orchestrator` | Watches V1ClusterCache, coordinates multi-cluster state | Single leader per namespace |
| **DNS Server** | `--dns-server` | gRPC DNS service on port 1153, watches V1HostnameCache | No (stateless) |
| **Front End** | `--front-end` | REST API with Swagger/OpenAPI docs | No |

### Data Flow

```
Cluster A (Operator) ──┐
                       ├── Orchestrator ── HostnameCache ── DNS Server ── CoreDNS
Cluster B (Operator) ──┘
```

## Custom Resources

### V1Gslb

Global Server Load Balancing resource (`multicluster.veccsolutions.io/v1alpha`):

| Field | Required | Default | Description |
|-------|----------|---------|-------------|
| `ObjectReference` | Yes | — | Reference to an Ingress or Service |
| `Hostnames` | Yes | — | DNS names to expose |
| `IPOverrides` | No | — | Alternate IP addresses |
| `Priority` | No | `0` | Higher value wins in failover |
| `Weight` | No | `50` | Round-robin weight (calculated as `weight / sum * 100`) |

### Other Resources

| Resource | Purpose |
|----------|---------|
| `V1ClusterCache` | Cached cluster state for orchestration |
| `V1HostnameCache` | DNS hostname-to-IP mappings |
| `V1ResourceCache` | Cached K8s resource state |
| `V1ServiceCache` | Cached service endpoints |

## DNS Configuration

| Setting | Default |
|---------|---------|
| DNS server name | `dns.vecck8smulticlusteringress.com` |
| Default TTL | 5 seconds |
| Refresh interval | 30 seconds |
| gRPC port | 1153 |

## Helm Chart

**Chart**: `multicluster-ingress` (v0.1.0)

### Components

| Component | Default Replicas | Port |
|-----------|-----------------|------|
| API Server | 1 | 8080 |
| DNS Server | 2 | 5000 |
| Operator | 2 | 8080 |
| Orchestrator | 2 | 8080 |

### Installation

```bash
helm install multicluster-ingress charts/multicluster-ingress/ \
  --namespace multicluster \
  --create-namespace \
  --values my-values.yaml
```

### Key Values

```yaml
# API server configuration
api:
  replicas: 1
  resources:
    limits:
      cpu: 300m      # 100m after startup
      memory: 128Mi
    requests:
      cpu: 100m
      memory: 64Mi

# DNS server with CoreDNS
dns:
  replicas: 2
  coredns:
    image: coredns/coredns:1.12.0
  resources:
    limits:
      cpu: 500m
      memory: 256Mi
    requests:
      cpu: 100m
      memory: 128Mi

# Operator watches K8s resources
operator:
  replicas: 2

# Orchestrator coordinates clusters
orchestrator:
  replicas: 2
```

### Generated Resources

- Deployments (API, DNS, Operator, Orchestrator)
- ServiceAccounts and RBAC (3 ClusterRoles + bindings)
- Services (ClusterIP for API, LoadBalancer for DNS)
- Optional Ingress for API with TLS
- ConfigMaps (CoreDNS config, API settings)
- Secrets (API keys, cluster authentication)
- CRDs (optional, can be managed externally)

## Authentication

Cluster-to-cluster authentication uses API keys:

- Each cluster has a unique salt and API key
- Remote peers are configured with their endpoint URL and API key
- Heartbeat monitoring with configurable intervals (check: 1s, set: 10s, timeout: 90s)

## Docker Image

- **Runtime**: `mcr.microsoft.com/dotnet/aspnet:10.0`
- **SDK**: `mcr.microsoft.com/dotnet/sdk:10.0`
- **Port**: 80 (HTTP/1.1)
- **Debug mode**: Set `DEBUG=1` build arg to include diagnostic tools (procps, net-tools, dnsutils, curl)
