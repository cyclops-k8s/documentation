# Local Kubeconfig (Azure OIDC)

Configures a local `~/.kube/config` with Azure OIDC authentication via `kubelogin`.

**Hook point:** `post_control_planes`

```yaml
kubernetes_hookfiles:
  post_control_planes:
    - /path/to/example-hooks/local-kubeconfig-azure/post-control-planes/local-kubeconfig-azure.yml
```

## Variables

| Variable | Required | Default | Type | Description |
|----------|----------|---------|------|-------------|
| `kubernetes_first_kube_control_plane` | Yes | (inherited) | hostname | First control plane for cert retrieval |
| `kubernetes_cluster_name` | Yes | (inherited) | string | Cluster name for kubeconfig context |
| `kubernetes_api_endpoint` | Yes | (inherited) | FQDN | API server endpoint |
| `kubernetes_api_port` | Yes | (inherited) | int | API server port |
| `kubernetes_oidc_client_id` | Yes | (inherited) | string | OIDC server ID (`--server-id` for kubelogin) |
| `kubernetes_kubelogin_azure_client_id` | Yes | — | string | Azure AD application client ID |
| `kubernetes_kubelogin_azure_tenant` | Yes | — | string | Azure AD tenant ID |
| `kubernetes_kubelogin_azure_login_arg` | Yes | — | string | kubelogin login argument (e.g. `devicecode`, `interactive`) |

## Prerequisites

Requires `kubelogin` installed locally.

## What It Does

1. Reads `ca.crt` from the first control plane
2. Creates `~/.kube/` directory locally
3. Writes the CA certificate to `~/.kube/<cluster-name>-ca.crt`
4. Configures kubectl cluster, credentials (via kubelogin exec), and context
5. Sets the new context as the current context
