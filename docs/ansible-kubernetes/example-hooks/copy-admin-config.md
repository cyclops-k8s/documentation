# Copy Admin Config

Copies the cluster `admin.conf` kubeconfig to your local `~/.kube/config` for direct kubectl access.

**Hook point:** `post_cluster_init`

```yaml
kubernetes_hookfiles:
  post_cluster_init:
    - /path/to/example-hooks/copy-admin-config/post-cluster-init/copy-admin-config.yaml
```

## Variables

None — copies from the first control plane to `~/.kube/config` with mode `0600`.
