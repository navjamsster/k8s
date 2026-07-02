https://killercoda.com/cka-mock-practice/scenario/fix-rbac

Namespace: qa-tools
├── Deployment: pod-explorer
│   └── Uses ServiceAccount: sa-explorer
├── ServiceAccount: sa-explorer
├── Roles (predefined by security team)
│   ├── config-reader (ConfigMaps only)
│   ├── secret-reader (Secrets only)
│   ├── pod-reader ← Selected ✓
│   └── deployment-viewer (Deployments only)
└── RoleBinding: explorer-rolebinding
    ├── Binds: pod-reader → sa-explorer
    └── Grants: get, list, watch on pods


https://killercoda.com/cka-mock-practice/scenario/GitLab-CICD-RBAC


kubectl get componentstatuses
sudo crictl ps | grep kube-apiserver
sudo crictl logs container id

heck component health
kubectl get componentstatuses

kube-apiserver
    ↓ (connects on port 2379)
etcd cluster
    ↓ (stores cluster state)
All Kubernetes objects


Correct Configuration:
---------------------
┌─────────────────┐
│ kube-apiserver  │
└────────┬────────┘
         │ Port 2379
         │ (Client API)
         ↓
┌─────────────────┐
│  etcd cluster   │
└─────────────────┘
         ↑
    Port 2380
(Peer communication)
         ↑
┌─────────────────┐
│  etcd members   │
│  talking to     │
│  each other     │
└─────────────────┘

Incorrect Configuration (The Problem):
--------------------------------------
┌─────────────────┐
│ kube-apiserver  │
└────────┬────────┘
         │ Port 2380 ❌
         │ (Wrong - Peer port!)
         ↓
┌─────────────────┐
│  etcd cluster   │
│ refusesconnection
│ (not a peer)    │
└─────────────────┘

Result: Connection refused
        API server down
        Cluster inaccessible

Static Pod Lifecycle:
--------------------
1. Edit manifest in /etc/kubernetes/manifests/
2. Kubelet detects file change
3. Kubelet stops old container
4. Kubelet starts new container with updated config
5. Pod runs with new configuration

(No kubectl ap

ply needed - automatic!)


https://killercoda.com/cka-mock-practice/scenario/troubleshoot-kube-api-server