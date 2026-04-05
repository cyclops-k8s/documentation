# CIS & STIG Compliance

The ansible-kubernetes playbook is hardened against two security frameworks:

- **CIS Kubernetes Benchmark 1.12**
- **DOD STIG Kubernetes V2 Release 1** (24 July 2024)

## CIS Benchmark 1.12

Most CIS benchmarks are handled automatically by kubeadm and the playbook. The playbook addresses additional items through configuration and post-installation hardening.

### Control Plane Components

| CIS ID | Description | Status | Notes |
|--------|-------------|--------|-------|
| 1.1.12 | Etcd data directory ownership set to etcd:etcd | ✅ Automated | Init container sets ownership (UID/GID 500) |
| 1.2.6 | Authorization mode not set to AlwaysAllow | ✅ Automated | Node + RBAC configured |
| 1.2.7 | Authorization mode includes Node | ✅ Automated | |
| 1.2.8 | Authorization mode includes RBAC | ✅ Automated | |
| 1.2.11 | AlwaysPullImages admission plugin | ✅ Automated | Enabled by default |
| 1.2.14 | NodeRestriction admission plugin | ✅ Automated | Included by default |
| 1.2.15 | Profiling disabled | ✅ Automated | `kubernetes_profiling: false` |
| 1.2.16 | Audit log path set | ✅ Automated | `/var/log/apiserver/audit.log` |
| 1.2.17 | Audit log max age ≥ 30 | ✅ Automated | 30 days |
| 1.2.18 | Audit log max backup ≥ 10 | ✅ Automated | 10 backups |
| 1.2.19 | Audit log max size ≥ 100 | ✅ Automated | 100 MB |
| 1.2.30 | Service account token expiration not extended | ✅ Automated | `false` |
| 1.3.1 | Terminated pod GC threshold set | ✅ Automated | 10 |
| 1.3.2 | Profiling disabled (controller manager) | ✅ Automated | |
| 1.4.1 | Profiling disabled (scheduler) | ✅ Automated | |

### Worker Nodes

| CIS ID | Description | Status | Notes |
|--------|-------------|--------|-------|
| 4.2.5 | Streaming connection idle timeout not 0 | ✅ Automated | 5 minutes |
| 4.2.12 | Strong cryptographic ciphers only | ✅ Automated | See [TLS Cipher Suites](configuration.md#tls-cipher-suites) |
| 4.2.13 | Pod PID limit set | ✅ Automated | 1000 |
| 4.2.14 | Seccomp enabled by default | ✅ Automated | |

### Items Requiring Manual Action

Some CIS items cannot be fully automated and require administrator attention during cluster operation:

- **Default service account** — ensure workloads do not use the `default` service account with auto-mounted tokens
- **Network policies** — must be defined per-namespace by the administrator
- **RBAC policies** — must be configured per-application

!!! info "Anonymous Auth (CIS 1.2.1)"
    Anonymous authentication is left enabled because it is required for nodes to join the cluster via kubeadm. RBAC restricts what the anonymous user can access.

## DOD STIG Compliance

The Kubernetes STIG Version 2 Release 1 (24 July 2024) has been applied. A STIG Viewer checklist file (`Stig checklist - Kubernetes.cklb`) is included in the repository.

### Key STIG Controls

| STIG ID | Description | Status |
|---------|-------------|--------|
| V-242382 | Authorization mode configured | ✅ |
| V-242402 | Audit logging enabled | ✅ |
| V-242409 | Profiling disabled | ✅ |
| V-242418 | Strong cipher suites | ✅ |
| V-242462 | Audit log size limit | ✅ |
| V-242463 | Audit log backup count | ✅ |
| V-242464 | Audit log retention | ✅ |
| V-242465 | Audit log path configured | ✅ |
| V-245541 | Streaming connection timeout | ✅ |
| V-254800 | Pod Security admission configured | ✅ |

### STIG Viewer

The DOD STIG Viewer is available at: [https://public.cyber.mil/stigs/srg-stig-tools/](https://public.cyber.mil/stigs/srg-stig-tools/)

Use it to open the `Stig checklist - Kubernetes.cklb` file included in the repository to review the full compliance status.

## Encryption at Rest

Etcd data is encrypted at rest using **AES-GCM** with a 32-byte key. The playbook:

1. Templates the encryption provider configuration
2. Configures the API server with `--encryption-provider-config`
3. Enables automatic reload of the encryption configuration
4. Verifies encryption after cluster creation by writing and reading a test secret

See [Configuration Reference — Encryption at Rest](configuration.md#encryption-at-rest) for key rotation procedures.
