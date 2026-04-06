# vSphere CPI

Installs the vSphere Cloud Provider Interface via Helm.

**Hook point:** `post_cluster_init`

```yaml
kubernetes_hookfiles:
  post_cluster_init:
    - /path/to/example-hooks/vsphere-cpi/post-cluster-init/install-vsphere-cpi.yaml
```

## Variables

| Variable | Required | Default | Type | Description |
|----------|----------|---------|------|-------------|
| `vsphere_cpi_server` | Yes | — | string | vCenter server hostname |
| `vsphere_cpi_username` | Yes | — | string | vCenter username |
| `vsphere_cpi_password` | Yes | — | string | vCenter password |
| `vsphere_cpi_datacenter` | Yes | — | string | vCenter datacenter name |
| `vsphere_cpi_region_category` | Yes | — | string | vSphere tag category for region |
| `vsphere_cpi_zone_category` | Yes | — | string | vSphere tag category for zone |
