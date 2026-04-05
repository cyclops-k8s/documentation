# Recovery

## Recovering Expired Kubelet Certificates

If kubelet certificates expire and nodes can no longer communicate with the API server, use the `recover-expired-kubelet-certs.yaml` playbook.

### Symptoms

- `kubectl get nodes` shows nodes as `NotReady`
- kubelet logs show certificate-related errors
- `Unable to connect to the server: x509: certificate has expired`

### Recovery Procedure

```bash
ansible-playbook -i inventory recover-expired-kubelet-certs.yaml
```

The playbook:

1. Generates a bootstrap token on the first control plane
2. Retrieves the CA certificate from the cluster
3. Creates a `bootstrap-kubelet.conf` file on each affected node with:
    - Bootstrap token for authentication
    - Base64-encoded CA certificate
    - API endpoint configuration
4. Restarts kubelet on each node
5. Kubelet uses the bootstrap token to request new certificates via CSR signing

!!! note "Automatic Renewal"
    The playbook configures a systemd timer for monthly certificate renewal (`kubernetes_manage_cert_renewal: true`). This should prevent certificate expiration under normal operation.

## Cluster Reset

The `reset.yaml` playbook performs a complete cluster teardown:

```bash
ansible-playbook -i inventory reset.yaml
```

!!! danger
    This is destructive and irreversible. It will completely destroy the cluster.

**What it does:**

1. Runs `kubeadm reset` on all nodes
2. Stops systemd units (kubelet, containerd)
3. Removes all containerd containers
4. Uninstalls Kubernetes packages (`kubeadm`, `kubelet`, `kubectl`)
5. Cleans up directories:
    - `/etc/kubernetes`
    - `/var/lib/etcd`
    - `/etc/cni`
    - `/var/lib/kubelet`
    - `/var/lib/containerd`
6. Removes the `etcd` user and group
7. Optionally reboots nodes
