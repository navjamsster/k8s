https://killercoda.com/cka-mock-practice/scenario/multi-label-pod-netpol

NetworkPolicy Architecture
NetworkPolicy provides Layer 3/4 filtering for pod-to-pod communication in Kubernetes:


Pod Traffic Flow:
----------------
Source Pod → CNI Plugin → NetworkPolicy Evaluation → Target Pod
                              ↓
                       iptables/eBPF Rules
                              ↓
                        ALLOW or DENY


https://killercoda.com/cka-mock-practice/scenario/egress-or-netpol-dns

https://killercoda.com/cka-mock-practice/scenario/create-egress-with-dns

important 
https://killercoda.com/cka-mock-practice/scenario/calico-setup


CNI Plugin Architecture
A Container Network Interface (CNI) plugin is essential for Kubernetes networking:

Pod Creation
    ↓
Kubelet calls CNI Plugin
    ↓
CNI Plugin allocates IP from IPAM
    ↓
Creates veth pair (pod ↔ node)
    ↓
Configures routing and iptables
    ↓
Pod has network connectivity


Calico Components
Calico consists of several key components:

calico-node: DaemonSet running on each node, handles routing and policy enforcement
calico-kube-controllers: Deployment that watches Kubernetes API for changes
calico-typha: Optional component for scaling to large clusters
Tigera Operator: Manages Calico lifecycle and configuration

How It Works Together
Tigera Operator
    ↓
Monitors Installation CR
    ↓
Deploys Calico Components
    ↓
calico-node (on each node)
    ├─ Felix: Policy enforcement and routing
    ├─ BIRD: BGP routing daemon
    └─ confd: Configuration management
    ↓
CNI Plugin (/opt/cni/bin/calico)
    ↓
Pod networking and policy enforcement


NetworkPolicy Flow:
------------------
NetworkPolicy Created
    ↓
API Server stores policy
    ↓
calico-kube-controllers detects change
    ↓
Syncs to Calico datastore
    ↓
calico-node (Felix) on each node
    ↓
Converts to iptables/eBPF rules
    ↓
Enforces traffic filtering at kernel level