YOU
 │
 │  kubectl apply -f pod.yaml
 ▼
┌─────────────────────────────────────────────────┐
│  STEP 1: kubectl reads ~/.kube/config            │
│  - API server address (server: https://...)      │
│  - Client certificate / token for auth           │
│  - Converts YAML → JSON REST API call            │
│  POST https://apiserver:6443/api/v1/namespaces/  │
│        default/pods                              │
└──────────────────┬──────────────────────────────┘
                   │ HTTPS + mTLS
                   ▼
┌─────────────────────────────────────────────────┐
│  STEP 2: kube-apiserver receives request         │
│                                                  │
│  ① Authentication (AuthN)                        │
│     - Validates client cert or token             │
│     - "Who are you?"                             │
│                                                  │
│  ② Authorization (AuthZ)                         │
│     - RBAC check                                 │
│     - "Are you allowed to create pods?"          │
│                                                  │
│  ③ Admission Controllers                         │
│     - MutatingAdmissionWebhook (modify object)   │
│     - ValidatingAdmissionWebhook (reject/allow)  │
│     - LimitRanger, ResourceQuota checks          │
│                                                  │
│  ④ Schema Validation                             │
│     - Is the pod spec valid YAML/JSON?           │
└──────────────────┬──────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────┐
│  STEP 3: etcd — Persist the object               │
│                                                  │
│  - API server writes pod object to etcd          │
│  - Pod status = "Pending", nodeName = ""         │
│  - KEY: /registry/pods/default/my-pod            │
│  - API server returns 201 Created to kubectl     │
│                                                  │
│  kubectl prints: "pod/my-pod created" ✅         │
└──────────────────┬──────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────┐
│  STEP 4: kube-scheduler WATCHES API server       │
│                                                  │
│  - Scheduler uses LIST+WATCH on API server       │
│  - "New pod with nodeName='' ? That's my job!"   │
│                                                  │
│  ① Filtering (Predicates)                        │
│     - Which nodes have enough CPU/RAM?           │
│     - NodeSelector, Taints/Tolerations, Affinity │
│                                                  │
│  ② Scoring (Priorities)                          │
│     - Rank remaining nodes (LeastRequested etc.) │
│                                                  │
│  ③ Binding                                       │
│     - PATCH pod → nodeName: node01               │
│     - Writes back to API server → etcd updated   │
└──────────────────┬──────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────┐
│  STEP 5: kubelet on node01 WATCHES API server    │
│                                                  │
│  - kubelet polls/watches: "Any pods for me?"     │
│  - Sees pod with nodeName: node01                │
│                                                  │
│  ① Calls containerd/CRI to pull image            │
│  ② Sets up CNI networking (eth0, IP, routes)     │
│  ③ Mounts volumes (CSI)                          │
│  ④ Starts container                              │
│                                                  │
│  ⑤ Reports back to API server:                   │
│     pod.status.phase = "Running"                 │
│     pod.status.podIP = "10.244.1.5"              │
│     API server writes status → etcd              │
└──────────────────┬──────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────┐
│  STEP 6: kube-controller-manager WATCHES too     │
│                                                  │
│  - ReplicaSet controller: "desired=3, actual=1"  │
│    → Creates 2 more pods via API server          │
│  - Node controller: watches node heartbeats      │
│  - Endpoints controller: updates Service IPs     │
│  Each controller is a separate control loop      │
└─────────────────────────────────────────────────┘