# RadosGW Usage Exporter

A Prometheus exporter for Ceph RADOS Gateway (RGW) usage metrics.

## How It Works

The exporter authenticates to the RGW Admin API using AWS SigV4, parses usage logs, and outputs Prometheus-format metrics.

### Metrics

| Metric | Type | Labels | Description |
|--------|------|--------|-------------|
| `radosgw_usage_ops_total` | counter | bucket, owner, category, store | Total operation count per bucket/owner |

## Configuration

The `scrape.sh` script accepts configuration via CLI arguments or environment variables (CLI takes precedence):

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `RGW_ENDPOINT` | Yes | — | RGW Admin API URL |
| `ACCESS_KEY` | Yes | — | S3 credentials with admin capabilities |
| `SECRET_KEY` | Yes | — | S3 secret key |
| `STORE` | No | `"default"` | Store name label for metrics |
| `SCRAPE_INTERVAL` | No | `60` | Polling interval in seconds |
| `METRICS_DIR` | No | `/metrics` | Output directory for metrics files |
| `ADMIN_PATH` | No | `"admin"` | Admin API path |
| `TIMEOUT` | No | `60` | Request timeout in seconds |

## Docker Image

Based on Alpine Linux with `curl`, `jq`, and `bash`.

## Kubernetes Deployment

The `kubernetes/` directory contains manifests for deploying the exporter:

| Resource | Details |
|----------|---------|
| **Deployment** | 2 containers: scraper (Alpine + scrape.sh) and web server (Python HTTP on port 9099) |
| **Service** | ClusterIP on port 9099 |
| **ServiceMonitor** | Prometheus scrape config, 120s interval |
| **ConfigMap** | Store name, scrape interval, admin path, timeout |
| **Secrets** | Pulled from Rook Ceph object user secret |
| **Shared Volume** | `emptyDir` (Memory-backed, 64Mi limit) mounted at `/metrics` |

### Security

- Runs as non-root (UID 65534)
- No Linux capabilities
- Read-only root filesystem
- Metrics served at `/metrics/metrics.prom`
