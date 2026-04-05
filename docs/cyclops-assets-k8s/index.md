# Cyclops Assets K8s

Kubernetes manifests for asset management, container registry mirroring, and OS image synchronization.

**Namespace**: `cyclops-assets`

## Components

### Asset Server

An nginx-based HTTP/HTTPS asset server deployed as a DaemonSet on nodes labeled `cyclops-k8s.io/ansible-kubernetes=amd64`.

| Resource | Purpose |
|----------|---------|
| `assets-daemonset.yaml` | Nginx DaemonSet (ports 80, 443) |
| `assets-cm.yaml` | Nginx configuration |
| `assets-pvc.yaml` | Persistent storage for assets |
| `assets-service.yaml` | Service exposure |
| `assets-certificate.yaml` | TLS certificate |
| `cert-issuer.yaml` | Certificate issuer |

Health checks are served at `/healthz`.

### Container Registry Mirrors

Pull-through cache mirrors for three major container registries, each consisting of a DaemonSet, ConfigMap, PVC, and Service:

| Registry | Manifests | Port |
|----------|-----------|------|
| Docker Hub (`docker.io`) | `mirror-docker-io-*` | 5000 |
| Quay.io (`quay.io`) | `mirror-quay-io-*` | 5000 |
| Kubernetes Registry (`registry.k8s.io`) | `mirror-registry-k8s-io-*` | 5000 |

Each mirror runs the `registry:3` image with persistent storage for cached layers.

An nginx proxy router (`registry-mirrors-daemonset.yaml` + `registry-mirrors-proxy-cm.yaml`) routes requests to the appropriate mirror.

### OS Image Sync

| Resource | Purpose |
|----------|---------|
| `os-image-sync-cronjob.yaml` | Daily CronJob to download OS images |
| `os-images-cm.yaml` | Configuration for which images to sync |

### Squid Proxy

| Resource | Purpose |
|----------|---------|
| `squid-*.yaml` | Squid caching proxy manifests |

## Resource Limits

Each mirror and asset service is configured with:

- **CPU**: 10m request / 500m limit
- **Memory**: 100Mi request / 500Mi limit
