# Backup and Restore

## 1. Confirm cluster state
Ensure all pods are running before taking a backup.

```bash
k get pods
k get all
```

## 2. Check etcd manifest and certs
Inspect the etcd manifest and confirm the certificate file paths.

```bash
cat /etc/kubernetes/manifests/etcd.yaml
cat /etc/kubernetes/manifests/etcd.yaml | grep file
```

## 3. Create backup
Use `etcdctl snapshot save` to create a snapshot.

```bash
etcdctl snapshot save --help
```

Example command:

```bash
etcdctl snapshot save \
  --endpoints=127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  /opt/snapshot-pre-boot.db
```

## 4. Restore

### a) Stop kube-apiserver and back up the manifest
Stop the apiserver to prevent writes to etcd during restore, then move the manifest.

```bash
ls /etc/kubernetes/manifests/
mv /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/kube-apiserver.yaml
cat /tmp/kube-apiserver.yaml
```

### b) Confirm kube-apiserver is stopped

```bash
crictl ps | grep kube-apiserver
crictl ps
```

### c) Restore the snapshot

```bash
etcdctl snapshot restore /opt/snapshot-pre-boot.db --data-dir=/var/lib/etcd-backup
```

This creates `/var/lib/etcd-backup/member/` on the host filesystem.
It does not touch the running etcd process yet.

### d) Update `etcd.yaml`
Only change `hostPath.path` in `/etc/kubernetes/manifests/etcd.yaml`.

```yaml
# BEFORE:
volumes:
  - hostPath:
      path: /var/lib/etcd   # OLD
    name: etcd-data

# AFTER:
volumes:
  - hostPath:
      path: /var/lib/etcd-backup   # NEW (matches --data-dir)
    name: etcd-data
```

Important:
- Do not change the `--data-dir` flag inside the manifest; it stays `/var/lib/etcd` (container internal path).
- Do not change `volumeMounts.mountPath`; it stays `/var/lib/etcd` (container internal path).

### e) Bring kube-apiserver back

```bash
mv /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/
ls /etc/kubernetes/manifests/
```

## 5. Final verification
Wait for kube-apiserver to restart (~30-60s), then verify the cluster is healthy.

```bash
crictl ps -a
crictl ps | grep kube-apiserver
k get pods
k get all
```
