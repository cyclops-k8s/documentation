# Admin Role Binding

Creates a ClusterRoleBinding granting `cluster-admin` to the OIDC `oidc:Admins` group.

**Hook point:** `post_cluster_init`

```yaml
kubernetes_hookfiles:
  post_cluster_init:
    - /path/to/example-hooks/add-adminrolebinding/post-cluster-init/add-adminbinding.yaml
```

## Variables

None — uses the inherited `first_kube_control_plane`, `kubernetes_config_directory`, and `kubernetes_output_directory` variables.

## Details

The hook applies a ClusterRoleBinding manifest that binds:

- **Group:** `oidc:Admins`
- **ClusterRole:** `cluster-admin`

This allows members of the `Admins` role in your OIDC provider to have full cluster access.
