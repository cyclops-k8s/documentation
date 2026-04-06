# ArgoCD

Bootstraps an ArgoCD HA installation with Azure OIDC authentication via kustomize.

**Hook point:** `post_workers`

```yaml
kubernetes_hookfiles:
  post_workers:
    - /path/to/example-hooks/argocd/post-workers/install-argocd.yaml
```

## Variables

| Variable | Required | Default | Type | Description |
|----------|----------|---------|------|-------------|
| `argocd_fqdn` | Yes | — | URL | ArgoCD server URL (used in ConfigMap and Ingress) |
| `argocd_oidc_azure_tenant_id` | Yes | — | string | Azure AD tenant ID for OIDC |
| `argocd_oidc_azure_client_id` | Yes | — | string | Azure AD application client ID |
| `argocd_oidc_azure_client_secret` | Yes | — | string | Azure AD client secret |
| `argocd_admin_password` | Yes | — | string | ArgoCD admin password (base64-encoded in secret) |

## What It Installs

- ArgoCD v2.12.3 HA cluster install via kustomize
- Insecure server mode (for TLS termination at ingress)
- RBAC with default read-only role, admin/user group mappings
- Nginx ingress with TLS for `argocd_fqdn`
