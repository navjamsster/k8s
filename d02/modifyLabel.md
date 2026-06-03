# Label All Pods in a Namespace Lab
> **CKA Practice | Labels & Selectors | Single Imperative Command**

A hands-on lab to replicate a real CKA exam scenario: add a label `env=finance-production` to **all pods** in the `finance` namespace using a **single imperative command**.

---

## 📖 Scenario

There are multiple deployments running in the `finance` namespace.

You have to add a label `env=finance-production` to **all the pods** under this namespace using a **single imperative command**.

---

## 🗂 Files

| File | Description |
|---|---|
| `setup.yaml` | Creates namespace `finance` + 4 deployments (frontend, backend, api-gateway, payment-service) |
| `README.md` | This file — full guide with use case, setup, solution, and explanation |

---

## 🏗 Lab Architecture

```
Namespace: finance
│
├── Deployment: frontend        (2 replicas)  app=frontend
├── Deployment: backend         (2 replicas)  app=backend
├── Deployment: api-gateway     (1 replica)   app=api-gateway
└── Deployment: payment-service (2 replicas)  app=payment-service
                                              ↓
                              Goal: add env=finance-production
                              to ALL 7 pods with ONE command
```

---

## 🚀 Setup on Kind Cluster

### Step 1 — Create kind cluster (if not already running)

```bash
kind create cluster --name cka-lab
```

### Step 2 — Deploy the lab

```bash
kubectl apply -f setup.yaml
```

### Step 3 — Verify pods are running

```bash
kubectl get pods -n finance
# NAME                               READY   STATUS
# frontend-xxx-yyy                   1/1     Running
# frontend-xxx-zzz                   1/1     Running
# backend-xxx-yyy                    1/1     Running
# backend-xxx-zzz                    1/1     Running
# api-gateway-xxx-yyy                1/1     Running
# payment-service-xxx-yyy            1/1     Running
# payment-service-xxx-zzz            1/1     Running
```

### Step 4 — Confirm no label yet

```bash
kubectl get pods -n finance --show-labels
# None of them have env=finance-production yet
```

---

## ✅ Solution — Single Imperative Command

```bash
kubectl label pods --all -n finance env=finance-production
```

### That's it! 🎉

---

## 🔍 Command Breakdown

```
kubectl label pods --all -n finance env=finance-production
│       │     │    │     │          │
│       │     │    │     │          └── label KEY=VALUE to add
│       │     │    │     └───────────── namespace: finance
│       │     │    └─────────────────── target ALL pods
│       │     └──────────────────────── resource type: pods
│       └────────────────────────────── subcommand: label
└────────────────────────────────────── kubectl
```

| Part | Meaning |
|---|---|
| `kubectl label` | Imperative command to add/update/remove labels |
| `pods` | Resource type to label |
| `--all` | Target ALL resources of that type in the namespace |
| `-n finance` | In the finance namespace |
| `env=finance-production` | The label key=value to add |

---

## 🔎 Verify the Labels Were Applied

```bash
# Show all pods with their labels
kubectl get pods -n finance --show-labels

# Filter only pods that have env=finance-production
kubectl get pods -n finance -l env=finance-production

# Count how many pods have the label
kubectl get pods -n finance -l env=finance-production --no-headers | wc -l
# Expected: 7
```

---

## 📊 Expected Output After Labeling

```
NAME                               READY  STATUS   LABELS
frontend-xxx-yyy                   1/1    Running  app=frontend,env=finance-production
frontend-xxx-zzz                   1/1    Running  app=frontend,env=finance-production
backend-xxx-yyy                    1/1    Running  app=backend,env=finance-production
backend-xxx-zzz                    1/1    Running  app=backend,env=finance-production
api-gateway-xxx-yyy                1/1    Running  app=api-gateway,env=finance-production
payment-service-xxx-yyy            1/1    Running  app=payment-service,env=finance-production
payment-service-xxx-zzz            1/1    Running  app=payment-service,env=finance-production
```

---

## 🧠 Key Concepts

### Imperative vs Declarative

| Approach | Command | When to use |
|---|---|---|
| **Imperative** ✅ | `kubectl label pods --all -n finance env=finance-production` | CKA exam — fast, single command |
| **Declarative** | Edit each deployment YAML → add label → `kubectl apply` | Production — auditable, GitOps |

### Why `--all` targets pods only in the given namespace?

`--all` is always **namespace-scoped**. Combined with `-n finance` it means:
> "all pods **within** the finance namespace"

Without `-n finance`, it targets `default` namespace.

### What about newly created pods?

> ⚠️ **Important:** `kubectl label pods --all` labels **currently running pods only**.
> If a pod is replaced (e.g., deployment rolling update), the **new pod will NOT have the label**.
>
> To make labels persistent, add them to the **deployment template** instead:
> ```bash
> kubectl label deployment --all -n finance env=finance-production
> # This labels the deployment object, NOT the pods
> ```
>
> For persistent pod labels, edit the deployment spec:
> ```bash
> kubectl patch deployment frontend -n finance \
>   --type=json \
>   -p='[{"op":"add","path":"/spec/template/metadata/labels/env","value":"finance-production"}]'
> ```

---

## 🎯 Variations — Other Useful Label Commands

```bash
# Label a SINGLE pod
kubectl label pod frontend-xxx-yyy -n finance env=finance-production

# Label ALL pods across ALL namespaces
kubectl label pods --all --all-namespaces env=finance-production

# Label all pods matching a selector
kubectl label pods -l app=frontend -n finance env=finance-production

# OVERWRITE an existing label value
kubectl label pods --all -n finance env=finance-staging --overwrite

# REMOVE a label from all pods
kubectl label pods --all -n finance env-
#                                       ↑ dash at end = remove label

# Label deployments (not pods)
kubectl label deployment --all -n finance env=finance-production

# Label nodes
kubectl label nodes <node-name> env=finance-production
```

---

## ⚠️ Common Mistakes

| Mistake | Problem | Fix |
|---|---|---|
| Forgetting `-n finance` | Labels pods in `default` namespace | Always specify `-n <namespace>` |
| Using `kubectl label pod` (singular) | Requires pod name — not bulk | Use `pods --all` |
| No `--overwrite` when label exists | Error: "already has a value" | Add `--overwrite` flag |
| Labeling deployments instead of pods | Deployment object gets label, pods don't | Use `pods --all` not `deployment --all` |

---

## 🧹 Cleanup

```bash
# Remove the label from all pods
kubectl label pods --all -n finance env-

# Delete all resources
kubectl delete namespace finance

# Delete kind cluster
kind delete cluster --name cka-lab
```

---

## 📚 References

- [kubectl label — Kubernetes Docs](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_label/)
- [Labels and Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)

---

*Namespace: `finance` | Command: `kubectl label pods --all` | Topic: Imperative Labels*
