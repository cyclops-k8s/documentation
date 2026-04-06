# Local Kubeconfig (int128 OIDC)

Configures a local `~/.kube/config` with generic OIDC authentication via `kubelogin` (int128/kubelogin).

**Hook point:** `post_control_planes`

```yaml
kubernetes_hookfiles:
  post_control_planes:
    - /path/to/example-hooks/local-kubeconfig-int128/post-control-planes/local-kubeconfig-int128.yml
```

## Variables

| Variable | Required | Default | Type | Description |
|----------|----------|---------|------|-------------|
| `kubernetes_first_kube_control_plane` | Yes | (inherited) | hostname | First control plane for cert retrieval |
| `kubernetes_cluster_name` | Yes | (inherited) | string | Cluster name for kubeconfig context |
| `kubernetes_api_endpoint` | Yes | (inherited) | FQDN | API server endpoint |
| `kubernetes_api_port` | Yes | (inherited) | int | API server port |
| `kubernetes_oidc_issuer_url` | Yes | (inherited) | URL | OIDC issuer URL |
| `kubernetes_oidc_client_id` | Yes | (inherited) | string | OIDC client ID |

## Prerequisites

Requires `kubelogin` ([int128/kubelogin](https://github.com/int128/kubelogin)) installed locally.

## What It Does

1. Reads `ca.crt` from the first control plane
2. Creates `~/.kube/` directory locally
3. Writes the CA certificate to `~/.kube/<cluster-name>-ca.crt`
4. Configures kubectl cluster, credentials (via kubelogin exec with OIDC args), and context
5. Sets the new context as the current context
