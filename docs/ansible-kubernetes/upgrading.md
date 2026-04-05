# Upgrading

The `upgrade.yaml` playbook performs a rolling upgrade of the Kubernetes cluster with zero downtime.

## Prerequisites

- Update `kubernetes_version` in your configuration to the target minor version
- The target version must be in `kubernetes_supported_versions` (default: `["1.33", "1.34", "1.35"]`)
- Only single minor version jumps are supported (e.g. 1.33 → 1.34, not 1.33 → 1.35)

## Running the Upgrade

```bash
ansible-playbook -i inventory upgrade.yaml
```

## Upgrade Order

The playbook upgrades nodes in a carefully orchestrated order to maintain cluster availability:

### 1. Detect Etcd Leader

The playbook queries the etcd cluster to identify the current leader. This ensures the leader is upgraded last to minimize disruption.

### 2. Control Plane Upgrade

Control planes are upgraded **one at a time** in this order:

1. **Etcd followers first** — all non-leader control planes
2. **Etcd leader last** — the current leader node

For each control plane:

1. Execute `pre_upgrade_control_planes` hooks
2. Update APT/RPM repositories to the new version
3. Upgrade `kubeadm` package
4. Run `kubeadm upgrade apply` (first node) or `kubeadm upgrade node` (subsequent nodes)
5. Verify etcd health
6. Verify API server health
7. Upgrade `kubelet` and `kubectl` packages
8. Restart kubelet
9. Execute `post_upgrade_control_planes` hooks

### 3. Worker Node Upgrade

For each worker node:

1. Execute `pre_upgrade_workers` hooks
2. **Drain the node** — evicts all pods
3. Upgrade `kubeadm`, `kubelet`, and `kubectl` packages
4. Run `kubeadm upgrade node`
5. Restart kubelet
6. **Uncordon the node** — allows scheduling again
7. Execute `post_upgrade_workers` hooks

### 4. Post-Upgrade

After all nodes are upgraded:

1. Execute `post_upgrade` hooks
2. Verify cluster health

## Etcd WAL Bug Workaround

For etcd 3.6, the upgrade playbook includes a workaround for a membership consistency issue. It forces a Raft snapshot before certain upgrade operations to ensure the write-ahead log (WAL) is consistent across all etcd members.

## Version Locking

During the upgrade, package version locks are temporarily removed, the upgrade is performed, and then new version locks are applied to prevent accidental further upgrades:

- **Debian**: `dpkg --set-selections` (hold/install)
- **RedHat**: `dnf versionlock`

## Upgrade Hooks

Use these hooks to run custom tasks during the upgrade:

```yaml
kubernetes_hookfiles:
  pre_upgrade_control_planes:
    - "{{ inventory_dir }}/hooks/pre-upgrade-backup.yaml"
  post_upgrade:
    - "{{ inventory_dir }}/hooks/post-upgrade-verify.yaml"
```

| Hook | When | Use case |
|------|------|----------|
| `pre_upgrade_control_planes` | Before any control plane upgrades | Etcd snapshots, pre-flight checks |
| `post_upgrade_control_planes` | After control plane kubeadm upgrade | Validation |
| `pre_upgrade_workers` | Before any worker upgrades | Workload migration |
| `post_upgrade_workers` | After worker kubeadm upgrade | Validation |
| `post_upgrade` | After all upgrades complete | Smoke tests, notifications |
