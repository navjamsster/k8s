# Create with specific Kubernetes version
kind create cluster --config \cluster\kind-cluster.yaml \
  --image kindest/node:v1.32.0

# List all clusters
kind get clusters

# Delete the kind cluster
kind delete cluster --name k8s-c2

# Verify
kubectl config get-contexts
# The row with * is the current context
kubectl config current-context

kubectl config view --minify


kind create cluster --config kind-cluster.yaml --name k8s-c2
          │
          ▼
┌─────────────────────────────────────────────────────────┐
│  STEP 1: Pull node image                                │
│  docker pull kindest/node:v1.32.0                       │
└─────────────────────────┬───────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  STEP 2: Create Docker containers (one per node)        │
│  docker run ... --name k8s-c2-control-plane             │
│  docker run ... --name k8s-c2-worker                    │
└─────────────────────────┬───────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  STEP 3: Run kubeadm inside control-plane container     │
│  kubeadm init → generates ALL cluster certificates:     │
│    /etc/kubernetes/pki/ca.crt      ← Cluster CA         │
│    /etc/kubernetes/pki/apiserver.*  ← API server cert   │
│    /etc/kubernetes/admin.conf       ← admin kubeconfig  │
└─────────────────────────┬───────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  STEP 4: Worker joins via kubeadm join                  │
│  TLS bootstrapping happens → kubelet gets its certs     │
│    /var/lib/kubelet/pki/kubelet-client-current.pem      │
│    /var/lib/kubelet/pki/kubelet.crt                     │
└─────────────────────────┬───────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  STEP 5: kind extracts kubeconfig from control-plane    │
│  docker exec k8s-c2-control-plane                       │
│    cat /etc/kubernetes/admin.conf                       │
│  → Copies it to your local machine                      │
└─────────────────────────┬───────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│  STEP 6: kind MERGES into ~/.kube/config                │
│  Adds 3 entries:                                        │
│    clusters:  → k8s-c2  (API server URL + CA cert)      │
│    users:     → k8s-c2  (client cert + key)             │
│    contexts:  → kind-k8s-c2  (links cluster + user)     │
│  Sets kind-k8s-c2 as CURRENT context                    │
└─────────────────────────┬───────────────────────────────┘
                          │
                          ▼
           Prints: "Set kubectl context to kind-k8s-c2"

The Relationship Diagram

 ~/.kube/config
│
├── clusters:
│     └── k8s-c2 ──────────────────── API: https://127.0.0.1:PORT
│                                      CA cert: (cluster CA)
│
├── users:
│     └── k8s-c2 ──────────────────── admin client cert + key
│
└── contexts:
      └── kind-k8s-c2
            ├── cluster ──────────→  k8s-c2  (which cluster)
            ├── user    ──────────→  k8s-c2  (which credentials)
            └── namespace ────────→  (empty = default)

kubectl config use-context kind-k8s-c2
→ kubectl now uses k8s-c2 cluster + k8s-c2 credentials          
           


ssh node01 ( worker-node)
ps aux 

⚡ Step 1 — Get the Real Error (Most Important Command)
bash
# Best command — last 50 lines, no pager
journalctl -u kubelet -n 50 --no-pager

# OR follow live as it crashes/restarts
journalctl -u kubelet -f

# OR since last boot only
journalctl -u kubelet --since today --no-pager



# 1. See why it failed
journalctl -u kubelet -n 50 --no-pager

# 2. Check key config files
cat /var/lib/kubelet/config.yaml
cat /etc/kubernetes/kubelet.conf
cat /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf

# 3. Check dependencies
systemctl status containerd

# 4. Fix → reload → restart
systemctl daemon-reload
systemctl restart kubelet

# 5. Verify
systemctl status kubelet
kubectl get nodes   # should show Ready


https://harshitsahu2311.hashnode.dev/troubleshooting-control-plane-failure-cka