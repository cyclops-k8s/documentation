# Sealed Secrets

Installs Bitnami Sealed Secrets controller via Helm.

**Hook point:** `post_workers`

```yaml
kubernetes_hookfiles:
  post_workers:
    - /path/to/example-hooks/sealed-secrets/post-workers/sealed-secrets.yaml
```

## Variables

None — installs with default Helm values into the `kube-system` namespace.
