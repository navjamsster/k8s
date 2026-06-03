# K8s NetworkPolicy Lab — `project-snake`

> **CKA Practice | Question 24 | Security — NetworkPolicy**

A hands-on lab to replicate a real CKA exam scenario: restricting outgoing traffic from a hacked `backend` pod using a Kubernetes `NetworkPolicy`.

---

## 📖 Scenario

There was a security incident where an intruder was able to access the whole cluster from a single hacked **backend Pod**.

To prevent this, create a `NetworkPolicy` called `np-backend` in Namespace `project-snake`. It should allow the `backend-*` Pods **only** to:

- Connect to `db1-*` Pods on **port 1111**
- Connect to `db2-*` Pods on **port 2222**

After implementation, connections from `backend-*` Pods to `vault-*` Pods on port **3333** should **no longer work**.

Use the `app` label of Pods in your policy.

---

## 🗂 Files

| File | Description |
|---|---|
| `networkpolicy-lab.yaml` | All-in-one YAML: namespace, 4 pods, and `np-backend` NetworkPolicy |
| `lab-guide.html` | Interactive visual guide with architecture diagram and copy-able commands |
| `README.md` | This file |

---

## 🏗 Architecture

```
Namespace: project-snake
┌──────────────────────────────────────────────────────────┐
│                                                          │
│   [backend-pod]  ──── port 1111 ──→  [db1-pod]   ✅    │
│   app: backend   ──── port 2222 ──→  [db2-pod]   ✅    │
│                  ──── port 3333 ──→  [vault-pod]  ❌    │
│                                                          │
│   NetworkPolicy: np-backend                              │
│   policyType: Egress                                     │
└──────────────────────────────────────────────────────────┘
```

---

## ✅ Prerequisites

- [kind](https://kind.sigs.k8s.io/) installed
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) running
- `kubectl` installed and configured

---

## 🚀 Quick Start

### Step 1 — Create kind cluster

```bash
kind create cluster --name cka-lab
kubectl get nodes
```

### Step 2 — Apply the full lab YAML

```bash
kubectl apply -f networkpolicy-lab.yaml
```

This creates:
- Namespace `project-snake`
- 4 pods: `backend-pod`, `db1-pod`, `db2-pod`, `vault-pod`
- NetworkPolicy `np-backend`

### Step 3 — Verify everything is running

```bash
kubectl get all -n project-snake
kubectl get networkpolicy -n project-snake
```

### Step 4 — Test the policy

```bash
# Get pod IPs
kubectl get pods -n project-snake -o wide

# Store vault IP
VAULT_IP=$(kubectl get pod vault-pod -n project-snake -o jsonpath='{.status.podIP}')
DB1_IP=$(kubectl get pod db1-pod -n project-snake -o jsonpath='{.status.podIP}')
DB2_IP=$(kubectl get pod db2-pod -n project-snake -o jsonpath='{.status.podIP}')

# ✅ Should SUCCEED — backend → db1:1111
kubectl exec backend-pod -n project-snake -- nc -zv $DB1_IP 1111 --wait=5

# ✅ Should SUCCEED — backend → db2:2222
kubectl exec backend-pod -n project-snake -- nc -zv $DB2_IP 2222 --wait=5

# ❌ Should FAIL/TIMEOUT — backend → vault:3333
kubectl exec backend-pod -n project-snake -- nc -zv $VAULT_IP 3333 --wait=5
# Expected: Connection timed out → NetworkPolicy is working!
```

---

## 📄 Solution: NetworkPolicy YAML

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np-backend
  namespace: project-snake
spec:
  podSelector:
    matchLabels:
      app: backend          # Applies to backend-* pods

  policyTypes:
  - Egress                  # Restrict outgoing traffic only

  egress:
  # Allow backend → db1 on port 1111 ✅
  - to:
    - podSelector:
        matchLabels:
          app: db1
    ports:
    - protocol: TCP
      port: 1111

  # Allow backend → db2 on port 2222 ✅
  - to:
    - podSelector:
        matchLabels:
          app: db2
    ports:
    - protocol: TCP
      port: 2222

  # No rule for vault:3333 → BLOCKED by default ❌
```

---

## 🧠 Key Concepts

| Concept | Explanation |
|---|---|
| `podSelector` | Selects which pods the policy applies to using labels. Empty `{}` = all pods in namespace |
| `policyTypes: Egress` | Restricts **outgoing** traffic from the selected pods |
| Default deny | Once a NetworkPolicy selects a pod, **all traffic not explicitly allowed is denied** |
| Egress rule | Defines where the selected pod CAN send traffic — combine `to` (destination) with `ports` |
| `app` label | Standard convention to select pods in CKA exam questions |
| CNI required | NetworkPolicies only work with a supporting CNI: **Calico**, **Cilium**, **Weave**. Flannel does NOT support them |

---

## ⚠️ Important Note for kind

`kind` uses `kindnet` as its default CNI, which has **basic NetworkPolicy support**. If tests don't behave as expected, install Calico for full support:

```bash
# Create kind cluster without default CNI
cat <<EOF | kind create cluster --name cka-lab --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  disableDefaultCNI: true
nodes:
- role: control-plane
- role: worker
EOF

# Install Calico CNI
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

# Wait for Calico to be ready
kubectl wait --for=condition=ready pod -l k8s-app=calico-node -n kube-system --timeout=120s
```

---

## 🔍 Troubleshooting

```bash
# Check NetworkPolicy details
kubectl describe networkpolicy np-backend -n project-snake

# Check pod labels (must match policy selectors)
kubectl get pods -n project-snake --show-labels

# Check if nc (netcat) is available in nginx image
kubectl exec backend-pod -n project-snake -- which nc
# If not available, use wget:
kubectl exec backend-pod -n project-snake -- wget -T 3 $VAULT_IP:3333

# Check events for issues
kubectl get events -n project-snake --sort-by='.lastTimestamp'
```

---

## 🧹 Cleanup

```bash
# Delete namespace and all resources inside it
kubectl delete namespace project-snake

# Delete kind cluster entirely
kind delete cluster --name cka-lab
```

---

## 📚 References

- [Kubernetes NetworkPolicy Docs](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [kind Quick Start](https://kind.sigs.k8s.io/docs/user/quick-start/)
- [NetworkPolicy Editor (visual tool)](https://editor.networkpolicy.io/)

---

## 🎯 CKA Exam Tips

1. **Use `kubectl explain networkpolicy.spec`** to check field names during the exam — no need to memorize
2. **Generate a base YAML**: `kubectl get networkpolicy -o yaml` from an existing policy, then modify
3. **Always check `policyTypes`** — forgetting to set `Egress` means only Ingress is restricted
4. **No rule = blocked** — you only need to write ALLOW rules, never DENY rules
5. **Test with `nc -zv <ip> <port>`** — much faster than `curl` for port connectivity checks

---

*Namespace: `project-snake` | NetworkPolicy: `np-backend` | Kind: CKA Security Practice*
