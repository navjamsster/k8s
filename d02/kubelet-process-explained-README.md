# Kubelet Process Deep Dive
> **CKA Reference | `ps aux | grep kubelet` — Understanding the Kubelet Process**

A complete reference for understanding every part of the `kubelet` process output on a Kubernetes worker node — and how it connects to TLS bootstrapping.

---

## The Command

```bash
# Run inside the worker node
ps aux | grep kubelet

# Cleaner version (suppresses grep itself from output)
ps aux | grep [k]ubelet
```

---

## Sample Output (kind worker node)

```
root  219  1.5  1.0  2394980  81968  ?  Ssl  10:03  1:03 \
  /usr/bin/kubelet \
  --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf \
  --kubeconfig=/etc/kubernetes/kubelet.conf \
  --config=/var/lib/kubelet/config.yaml \
  --node-ip=172.18.0.5 \
  --node-labels= \
  --provider-id=kind://docker/k8s-c2/k8s-c2-worker \
  --runtime-cgroups=/system.slice/containerd.service
```

---

## Part 1 — Process Metadata Columns

| Column | Value | Meaning |
|---|---|---|
| `USER` | `root` | Kubelet runs as **root** — needs full system access |
| `PID` | `219` | Process ID of kubelet |
| `%CPU` | `1.5` | Currently using 1.5% CPU |
| `%MEM` | `1.0` | Currently using 1.0% RAM |
| `VSZ` | `2394980` | Virtual memory allocated (KB) |
| `RSS` | `81968` | Actual RAM in use (~80 MB) |
| `TTY` | `?` | No terminal — runs as a background daemon |
| `STAT` | `Ssl` | **S**=Sleeping, **s**=Session leader, **l**=Multi-threaded |
| `START` | `10:03` | Kubelet started at 10:03 |
| `TIME` | `1:03` | Total CPU time consumed = 1 min 3 sec |

---

## Part 2 — Kubelet Flags Explained

### `--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf`

**Purpose:** Used **only once** during first node startup — TLS bootstrapping

- When a new node joins the cluster, it has no certificate yet
- Uses a **bootstrap token** to temporarily authenticate with the API server
- Says: *"Hey API server — I'm new, please issue me a real certificate"*
- After the real cert is issued, this file is no longer actively used
- **This is the core of CKA Q23** — understanding this flow

```
bootstrap-kubelet.conf
    ↓
Sends CSR to kube-apiserver
    ↓
controller-manager approves + signs with Cluster CA
    ↓
Real client cert issued → /var/lib/kubelet/pki/kubelet-client-current.pem
```

---

### `--kubeconfig=/etc/kubernetes/kubelet.conf`

**Purpose:** The **real** kubeconfig used after bootstrapping is complete

- Contains the kubelet **client certificate** 
- Used for all ongoing communication: kubelet → kube-apiserver
- The client cert embedded here = `/var/lib/kubelet/pki/kubelet-client-current.pem`

```bash
# View kubeconfig content
cat /etc/kubernetes/kubelet.conf

# The client-certificate-data field points to the client cert
```

---

### `--config=/var/lib/kubelet/config.yaml`

**Purpose:** Kubelet's main configuration file (replaces individual CLI flags in modern K8s)

```bash
# View kubelet config
cat /var/lib/kubelet/config.yaml
```

Common settings inside:
```yaml
kind: KubeletConfiguration
staticPodPath: /etc/kubernetes/manifests   # Where static pods live
clusterDNS:
  - 10.96.0.10                             # CoreDNS IP
clusterDomain: cluster.local
authentication:
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
evictionHard:
  memory.available: "100Mi"
  nodefs.available: "10%"
```

---

### `--node-ip=172.18.0.5`

**Purpose:** The IP address of this node

- `172.18.0.5` = Docker bridge network IP assigned to this kind container
- Tells other components how to reach this node
- In cloud clusters this would be the private IP of the VM/EC2 instance

---

### `--node-labels=`

**Purpose:** Custom labels to apply to this node at startup

- Empty here (no custom labels)
- In production:
  ```
  --node-labels=topology.kubernetes.io/zone=eu-west-1a,node-role=worker
  ```

---

### `--provider-id=kind://docker/k8s-c2/k8s-c2-worker`

**Purpose:** Unique identifier for this node in the infrastructure provider

| Part | Value | Meaning |
|---|---|---|
| Provider | `kind` | Running in kind |
| Runtime | `docker` | Using Docker |
| Cluster | `k8s-c2` | Cluster name |
| Node | `k8s-c2-worker` | This node's name |

Cloud equivalents:
```
AWS EKS  → aws:///eu-west-1a/i-1234567890abcdef0
GKE      → gce://project-id/europe-west1-b/node-name
Azure    → azure:///subscriptions/.../virtualMachines/node-name
```

---

### `--runtime-cgroups=/system.slice/containerd.service`

**Purpose:** Tells kubelet where the container runtime (containerd) cgroup lives

- kind uses **containerd** as its container runtime
- Kubelet needs to co-manage resources with containerd
- Cgroups control CPU/memory limits per process

---

## Part 3 — The grep Line (Ignore This)

```
root  2188  0.0  0.0  3332  1652  pts/1  S+  11:12  0:00  grep kubelet
```

This is just the `grep` command itself showing up in the process list. It means nothing — ignore it.

**Tip:** Use the `[k]` trick to suppress it:
```bash
ps aux | grep [k]ubelet
```

---

## The Full TLS Bootstrapping Flow (CKA Q23 Connection)

```
Node joins cluster
      ↓
kubelet reads --bootstrap-kubeconfig (one-time token)
      ↓
Sends CSR (Certificate Signing Request) to kube-apiserver
      ↓
kube-controller-manager approves + signs with Cluster CA
      ↓
kubelet receives CLIENT cert, saves to:
      /var/lib/kubelet/pki/kubelet-client-current.pem
      Issuer     : CN = kubernetes        ← Cluster CA signed it
      EKU        : TLS Web Client Authentication
      ↓
kubelet also generates its own SERVER cert:
      /var/lib/kubelet/pki/kubelet.crt
      Issuer     : CN = k8s-c2-worker-ca@<timestamp>  ← Self-signed
      EKU        : TLS Web Server Authentication
      ↓
kubelet switches to --kubeconfig for all future API communication
```

---

## CKA Exam Tips

### Find kubelet config paths dynamically

On the exam, the paths may differ per cluster. Always find them via:

```bash
# Find exact paths used by kubelet
ps aux | grep [k]ubelet
```

Then pick the exact `--config`, `--kubeconfig`, `--bootstrap-kubeconfig` paths from the output.

### Inspect certificates after bootstrapping

```bash
# List all kubelet PKI files
ls /var/lib/kubelet/pki/

# Inspect CLIENT cert (outgoing → apiserver)
openssl x509 -noout -text \
  -in /var/lib/kubelet/pki/kubelet-client-current.pem \
  | grep -E "Issuer|Extended Key Usage" -A1

# Inspect SERVER cert (incoming ← apiserver)
openssl x509 -noout -text \
  -in /var/lib/kubelet/pki/kubelet.crt \
  | grep -E "Issuer|Extended Key Usage" -A1
```

### Check kubelet status

```bash
systemctl status kubelet
journalctl -u kubelet --no-pager | tail -30
```

---

## Quick Reference — Key File Paths

| File | Purpose |
|---|---|
| `/usr/bin/kubelet` | Kubelet binary |
| `/etc/kubernetes/bootstrap-kubelet.conf` | Bootstrap token (first join only) |
| `/etc/kubernetes/kubelet.conf` | Real kubeconfig (ongoing use) |
| `/var/lib/kubelet/config.yaml` | Kubelet configuration |
| `/var/lib/kubelet/pki/kubelet-client-current.pem` | CLIENT certificate |
| `/var/lib/kubelet/pki/kubelet.crt` | SERVER certificate |
| `/var/lib/kubelet/pki/kubelet.key` | Server private key |
| `/etc/kubernetes/pki/ca.crt` | Cluster CA certificate |

---

## Troubleshooting

```bash
# Kubelet not starting?
systemctl status kubelet
journalctl -u kubelet -f

# Certificate issues?
openssl x509 -noout -dates -in /var/lib/kubelet/pki/kubelet-client-current.pem

# Check if kubelet can reach API server
curl -k https://<apiserver-ip>:6443/healthz

# Restart kubelet
systemctl restart kubelet
```

---

*Node: `k8s-c2-worker` | Cluster: `k8s-c2` | Topic: Kubelet Process & TLS Bootstrapping*
