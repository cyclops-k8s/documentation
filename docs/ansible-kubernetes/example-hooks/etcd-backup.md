# Etcd Backup

Creates a Kubernetes CronJob for daily etcd snapshots.

**Hook point:** `post_control_planes`

```yaml
kubernetes_hookfiles:
  post_control_planes:
    - /path/to/example-hooks/etcd-backup/post-control-planes/etcd-backup.yaml
```

## Variables

None — the CronJob runs daily at 00:00 UTC using `bitnami/etcd:3.5.16`, saving snapshots to `/var/lib/etcd/snapshots/` on control-plane nodes.

!!! warning
    The snapshots are stored locally on the node. You should move them to external storage (NFS, S3, etc.) for production use.
