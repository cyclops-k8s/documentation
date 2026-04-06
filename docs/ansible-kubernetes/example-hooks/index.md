# Example Hooks

Ready-to-use hooks are provided in the `example-hooks/` directory. Each hook can be assigned to one or more lifecycle hook points via the `kubernetes_hookfiles` variable.

!!! note
    These hooks are intended for **bootstrapping only**. For ongoing management and version upgrades of cluster components, use GitOps tools like ArgoCD or Flux.

## CNI Plugins

| Hook | Hook Point | Description |
|------|-----------|-------------|
| [Cilium](cilium.md) | `post_cluster_init` | Cilium CNI with kube-proxy replacement, WireGuard, Hubble, and BGP |
| [Calico](calico.md) | `post_cluster_init` | Calico CNI from upstream manifest |

## Cluster Components

| Hook | Hook Point | Description |
|------|-----------|-------------|
| [ArgoCD](argocd.md) | `post_workers` | ArgoCD HA with Azure OIDC |
| [Sealed Secrets](sealed-secrets.md) | `post_workers` | Bitnami Sealed Secrets controller |
| [Kube-VIP](kube-vip.md) | `post_workers` | Virtual IP for LoadBalancer services |
| [vSphere CPI](vsphere-cpi.md) | `post_cluster_init` | vSphere Cloud Provider Interface |
| [Kubelet CSR Approver](kubelet-csr-approver.md) | `post_control_planes` | Automatic kubelet CSR approval |
| [Etcd Backup](etcd-backup.md) | `post_control_planes` | Daily etcd snapshot CronJob |

## Tools

| Hook | Hook Points | Description |
|------|------------|-------------|
| [Helm](helm.md) | `pre_configure_control_planes`, `pre_upgrade_control_planes` | Install/upgrade Helm CLI |
| [Kustomize](kustomize.md) | `pre_configure_control_planes`, `pre_upgrade_control_planes` | Install/upgrade Kustomize CLI |
| [crun](crun.md) | `pre_prerequisites`, `post_upgrade` | crun OCI runtime (replaces runc) |

## Configuration

| Hook | Hook Point | Description |
|------|-----------|-------------|
| [Admin Role Binding](admin-rolebinding.md) | `post_cluster_init` | OIDC admin ClusterRoleBinding |
| [Copy Admin Config](copy-admin-config.md) | `post_cluster_init` | Copy kubeconfig locally |
| [Local Kubeconfig (Azure)](local-kubeconfig-azure.md) | `post_control_planes` | Azure OIDC kubeconfig via kubelogin |
| [Local Kubeconfig (int128)](local-kubeconfig-int128.md) | `post_control_planes` | Generic OIDC kubeconfig via kubelogin |
| [Containerd Config](containerd-config.md) | `pre_prerequisites`, `post_upgrade` | CDI/KubeVirt containerd support |
| [Registry Mirrors](registry-mirrors.md) | `post_proxies` | Container registry pull-through mirrors |
| [Proxy on Control Planes](proxy-on-control-planes.md) | `pre_control_planes` | HAProxy + Keepalived on control planes |
