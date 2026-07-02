https://killercoda.com/cka-mock-practice/scenario/schedule-pod-with-labels

nodeSelector is the simplest way to schedule Pods on nodes that meet specific label requirements.

nodeName directly assigns the Pod to a specific node, bypassing the Kubernetes scheduler (manual scheduling).

All label key-value pairs defined under nodeSelector must match (logical AND condition).

Unlike nodeAffinity , nodeSelector does not support complex expressions or soft preferences.


Using nodeName:
----------------
You → Directly tell kubelet:
  Pod.spec.nodeName = "node-01"
        │
        ▼
  Scheduler: ❌ Skipped
  Kubelet on node-01 → Creates Pod directly


  Using nodeSelector:
-------------------
You → Add constraint:
  Pod.spec.nodeSelector:
    disktype: ssd
        │
        ▼
  Scheduler → Finds nodes with label disktype=ssd
        │
        ▼
  Chooses one node → Assigns nodeName internally
  Kubelet on that node → Creates Pod


Hardcode Pod to a node (manual override)	nodeName
Let scheduler choose node based on labels	nodeSelector
Use complex rules (AND/OR, weights, ranges)	nodeAffinity