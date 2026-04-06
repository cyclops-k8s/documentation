# Kubelet CSR Approver

Installs kubelet-csr-approver via Helm to automatically approve kubelet serving certificate CSRs. Helps with CIS Benchmark 1.2.5 (securing API server to kubelet communication).

**Hook point:** `post_control_planes`

```yaml
kubernetes_hookfiles:
  post_control_planes:
    - /path/to/example-hooks/kubelet-csr-approver/post-control-planes/install-kubelet-csr-approver.yml
```

## Variables

| Variable | Required | Default | Type | Description |
|----------|----------|---------|------|-------------|
| `kubernetes_kubelet_csr_approver_regex` | Yes | — | regex string | Provider regex pattern to match allowed CSR DNS names |
| `kubernetes_kubelet_csr_approver_ips` | No | `[]` | list of strings | List of allowed IP prefixes for CSR validation |
| `kubernetes_kubelet_csr_approver_bypass_dns_checks` | No | `false` | boolean | Bypass DNS resolution checks when approving CSRs |
