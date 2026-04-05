# Configuration Reference

All configuration variables are defined in `roles/kubernetes-defaults/defaults/main.yml`. Variables marked **Required** must be set before running any playbook.

## Required Variables

| Variable | Type | Description |
|----------|------|-------------|
| `kubernetes_control_plane_ip` | IP address | VIP for HAProxy/Keepalived load balancer |
| `kubernetes_api_endpoint` | FQDN | API endpoint FQDN — DNS must resolve before running the playbook |
| `kubernetes_encryption_key` | Base64 string | 32-byte key for etcd encryption at rest. Generate with `head -c 32 /dev/urandom \| base64` |
| `kubernetes_oidc_issuer_url` | URL | OIDC provider issuer URL (without `.well-known/openid-configuration`) |
| `kubernetes_oidc_client_id` | string | OIDC provider client ID |
| `vrrp_interface` | string | Network interface for the Keepalived floating VIP |
| `vrrp_password` | string | Keepalived authentication password |
| `vrrp_virtual_router_id` | integer | VRRP virtual router ID |

## Kubernetes Settings

| Variable | Default | Description |
|----------|---------|-------------|
| `kubernetes_version` | `"1.35"` | Minor version to install (e.g. `"1.34"`, not `"1.34.2"`) |
| `kubernetes_supported_versions` | `["1.33", "1.34", "1.35"]` | Supported minor versions |
| `kubernetes_cluster_name` | `"cluster"` | Cluster name |
| `kubernetes_dns_domain` | `"cluster.local"` | In-cluster DNS domain |
| `kubernetes_pod_subnet` | `"10.244.0.0/16"` | Pod CIDR |
| `kubernetes_service_subnet` | `"10.96.0.0/16"` | Service CIDR |
| `kubernetes_node_cidr_mask_size` | `24` | Per-node pod CIDR mask |
| `kubernetes_api_port` | `6443` | Load-balanced API port |
| `kubernetes_api_server_port` | `6443` | API server bind port |
| `kubernetes_api_server_bind_address` | `null` | API server bind address (`null` = auto-lookup) |
| `kubernetes_cloud_provider` | `null` | External cloud provider (if any) |
| `kubernetes_init_skip_phases` | `[]` | Phases to skip in `kubeadm init` |
| `kubernetes_kubeadm_init_extra_args` | `""` | Extra args appended to `kubeadm init` |
| `kubernetes_kubeadm_join_extra_args` | `""` | Extra args appended to `kubeadm join` |
| `skip_checks` | `false` | Skip default validation checks (not recommended) |

## Container Runtime

| Variable | Default | Description |
|----------|---------|-------------|
| `container_runtime` | `"containerd"` | Container runtime to use |
| `kubernetes_containerd_version` | `"latest"` | Containerd version (`"latest"` or specific like `"1.7.29"`) |
| `kubernetes_containerd_registry_mirrors` | `[]` | Registry mirror configuration (see below) |
| `kubernetes_containerd_credentials` | `[]` | Registry credentials (deprecated in containerd 2.0+) |
| `kubernetes_containerd_mirror_apt` | Docker default | APT repository URL for containerd |
| `kubernetes_containerd_mirror_rpm` | Docker default | RPM repository URL for containerd |

### Registry Mirror Configuration

For containerd < 2.2.0:

```yaml
kubernetes_containerd_registry_mirrors:
  - registry: docker.io
    endpoints:
      - https://dockermirror.example.com:5001
  - registry: quay.io
    endpoints:
      - https://quaymirror.example.com:5002
```

For containerd >= 2.2.0:

```yaml
kubernetes_containerd_registry_mirrors:
  - registry: docker.io
    hosts:
      - host: "https://dockermirror.example.com:5001"
        capabilities: '["pull", "resolve"]'
        skip_verify: false
    extra_files:
      - name: ca.pem
        content: |
          -----BEGIN CERTIFICATE-----
          ...
          -----END CERTIFICATE-----
```

## OIDC Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `kubernetes_oidc_issuer_url` | **Required** | OIDC issuer URL |
| `kubernetes_oidc_client_id` | **Required** | OIDC client ID |
| `kubernetes_oidc_username_claim` | `"upn"` | Token claim for username |
| `kubernetes_oidc_username_expression` | `null` | CEL expression for username (overrides claim) |
| `kubernetes_oidc_username_prefix` | `"oidc:"` | Prefix added to usernames |
| `kubernetes_oidc_group_claim` | `"roles"` | Token claim for group membership |
| `kubernetes_oidc_group_prefix` | `"oidc:"` | Prefix added to group names |
| `kubernetes_oidc_uid_claim` | `"oid"` | Token claim for unique user ID |

## Security & Compliance

| Variable | Default | CIS/STIG Reference | Description |
|----------|---------|---------------------|-------------|
| `kubernetes_audit_log_path` | `/var/log/apiserver/audit.log` | CIS 1.2.16, STIG V-242465 | Audit log file path |
| `kubernetes_audit_log_max_age` | `30` | CIS 1.2.17, STIG V-242464 | Days to retain audit logs |
| `kubernetes_audit_log_max_backup` | `10` | CIS 1.2.18, STIG V-242463 | Maximum audit log backups |
| `kubernetes_audit_log_max_size` | `100` | CIS 1.2.19, STIG V-242462 | Maximum audit log file size (MB) |
| `kubernetes_profiling` | `false` | CIS 1.2.15, STIG V-242409 | Disable profiling endpoints |
| `kubernetes_podpidslimit` | `1000` | CIS 4.2.13 | Pod PID limit |
| `kubernetes_seccomp_default` | `true` | CIS 4.2.14 | Enable seccomp by default |
| `kubernetes_streamingconnectionidletimeout` | `"5m"` | CIS 4.2.5, STIG V-245541 | Streaming connection idle timeout |
| `kubernetes_terminated_pod_gc_threshold` | `10` | CIS 1.3.1 | Terminated pod garbage collection threshold |
| `kubernetes_service_account_extend_token_expiration` | `false` | CIS 1.2.30 | Disable extended token expiration |
| `kubernetes_cluster_signing_duration` | `"8760h"` | — | Certificate signing duration |
| `kubernetes_manage_cert_renewal` | `true` | — | Automatic cert renewal via systemd timer |

### Encryption at Rest

| Variable | Default | Description |
|----------|---------|-------------|
| `kubernetes_encryption_key` | **Required** | Primary AES-GCM encryption key (base64 32 bytes) |
| `kubernetes_encryption_key_name` | `"key1"` | Name identifier for the primary key |
| `kubernetes_encryption_additional_keys` | `[]` | Additional keys for key rotation |
| `kubernetes_encryption_configuration_file` | `null` | Custom encryption config template |

**Key rotation procedure:**

1. Add the current `kubernetes_encryption_key` / `kubernetes_encryption_key_name` to `kubernetes_encryption_additional_keys`
2. Set `kubernetes_encryption_key` and `kubernetes_encryption_key_name` to the new values
3. Re-run the playbook
4. After all secrets are re-encrypted, remove the old key from `kubernetes_encryption_additional_keys`

### Admission Control

| Variable | Default | Description |
|----------|---------|-------------|
| `kubernetes_admission_control_plugins` | See defaults | Enabled admission controllers |
| `kubernetes_admission_configuration.enforce` | `"privileged"` | Pod Security Standards enforcement level |
| `kubernetes_admission_configuration.audit` | `"baseline"` | Pod Security Standards audit level |
| `kubernetes_admission_configuration.warn` | `"privileged"` | Pod Security Standards warning level |
| `kubernetes_admission_configuration.exemptions.namespaces` | `["kube-system"]` | Namespaces exempt from PSS |

### TLS Cipher Suites

| Variable | Default | CIS/STIG Reference |
|----------|---------|---------------------|
| `kubernetes_strong_cypher_suites` | See below | CIS 4.2.12, STIG V-242418 |

Default cipher suites:

- `TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256`
- `TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256`
- `TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384`
- `TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA`
- `TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA`

### Authorization Modes

| Variable | Default | Description |
|----------|---------|-------------|
| `kubernetes_authorization_pre_node` | `[]` | Authorization modes before Node |
| `kubernetes_authorization_pre_rbac` | `[]` | Authorization modes between Node and RBAC |
| `kubernetes_authorization_post_rbac` | `[]` | Authorization modes after RBAC |

The final authorization chain is: `pre_node` → `Node` → `pre_rbac` → `RBAC` → `post_rbac`

## Etcd

| Variable | Default | Description |
|----------|---------|-------------|
| `kubernetes_etcd_init_container_image` | `"alpine:latest"` | Init container for etcd directory ownership (CIS 1.1.12) |
| `kubernetes_etcd_group_id` | `500` | GID for etcd data directory |
| `kubernetes_etcd_user_id` | `500` | UID for etcd data directory |
| `kubernetes_etcd_extra_args` | `[]` | Extra arguments for etcd |

## Proxy (HAProxy / Keepalived)

| Variable | Default | Description |
|----------|---------|-------------|
| `kubernetes_proxy_port` | `6443` | Proxy listen port |
| `kubernetes_proxy_bind_address` | `""` | Proxy bind address (`""` = all interfaces) |
| `kubernetes_proxy_haproxy_config_file` | `"haproxy.cfg.j2"` | HAProxy config template |
| `kubernetes_proxy_keepalived_config_file` | `"keepalived.conf.j2"` | Keepalived config template |
| `kubernetes_control_plane_check_interval` | `"100ms"` | Health check interval |
| `kubernetes_control_plane_timeout` | `"4m0s"` | Health check timeout |
| `vrrp_state` | `"BACKUP"` | Initial Keepalived state (`MASTER` or `BACKUP`) |

## API Server Extra Arguments

| Variable | Default | Description |
|----------|---------|-------------|
| `kubernetes_api_server_extra_args` | `[]` | Extra kube-apiserver arguments |
| `kubernetes_api_server_additional_mounts` | `[]` | Extra volume mounts for the API server pod |
| `kubernetes_controller_manager_extra_args` | `[]` | Extra controller-manager arguments |
| `kubernetes_scheduler_extra_args` | `[]` | Extra scheduler arguments |

Example:

```yaml
kubernetes_api_server_extra_args:
  - name: feature-gates
    value: SomeFeature=true
```

## Additional Files

| Variable | Default | Description |
|----------|---------|-------------|
| `kubernetes_control_plane_additional_files` | `[]` | Files to copy to control plane nodes |
| `kubernetes_control_plane_additional_templates` | `[]` | Templates to render on control plane nodes |
| `kubernetes_worker_additional_files` | `[]` | Files to copy to worker nodes |
| `kubernetes_worker_additional_templates` | `[]` | Templates to render on worker nodes |

## Package Mirrors

| Variable | Default | Description |
|----------|---------|-------------|
| `kubernetes_mirror_apt` | `pkgs.k8s.io` | APT repository for Kubernetes packages |
| `kubernetes_mirror_rpm` | `pkgs.k8s.io` | RPM repository for Kubernetes packages |
| `kubernetes_containerd_mirror_apt` | `download.docker.com` | APT repository for containerd |
| `kubernetes_containerd_mirror_rpm` | `download.docker.com` | RPM repository for containerd |

## SELinux (RPM-based only)

| Variable | Default | Description |
|----------|---------|-------------|
| `kubernetes_configure_selinux` | `true` | Configure SELinux on proxy nodes for HAProxy |

## Directories

| Variable | Default | Description |
|----------|---------|-------------|
| `kubernetes_config_directory` | `/etc/kubernetes/config` | Playbook/component config storage |
| `kubernetes_pki_directory` | `/etc/kubernetes/pki` | Certificate storage |
| `kubernetes_output_directory` | `/etc/kubernetes/output` | Playbook output files |
| `kubernetes_scripts_directory` | `/opt/kubernetes/scripts` | Helper scripts on nodes |
| `kubernetes_kubelet_config_dropin_directory` | `/var/lib/kubelet/conf.d` | Kubelet config drop-in files |
