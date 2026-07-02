
https://killercoda.com/cka-mock-practice/scenario/vpa-policies

Vertical Pod Autoscaler Components
VPA Recommender: Monitors resource usage and provides recommendations
VPA Updater: Evicts pods that need to be updated with new resource requests
VPA Admission Controller: Sets correct resource requests on new pods
VPA Custom Resource: Defines the autoscaling policy

How VPA Works
1. VPA Recommender analyzes historical resource usage
2. Generates recommendations for CPU and memory
3. VPA Updater identifies pods needing updates
4. Pods are evicted (in Recreate mode)
5. VPA Admission Controller intercepts pod creation
6. Applies recommended resource requests/limits
7. New pods start with optimized resources

VPA Architecture:
----------------
                    ┌─────────────────┐
                    │  VPA Resource   │
                    │  (app-vpa)      │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
         ┌────▼────┐   ┌────▼────┐   ┌────▼────┐
         │Recomm-  │   │ Updater │   │Admission│
         │ender    │   │         │   │Controlle│
         └────┬────┘   └────┬────┘   └────┬────┘
              │             │              │
              │             │              │
         Analyzes      Evicts Pods    Modifies New
         Metrics       Needing        Pod Specs
                       Updates
              │             │              │
              └─────────────┼──────────────┘
                            │
                    ┌───────▼────────┐
                    │ app-deployment │
                    │  (2 replicas)  │
                    └────────────────┘

Resource Policy Flow:
--------------------
Current Usage → Recommender → Calculate Target
                                     ↓
                    ┌────────────────┴────────────────┐
                    │   Apply Constraints             │
                    │   - Min: 100m CPU / 128Mi RAM   │
                    │   - Max: 2 CPU / 2Gi RAM        │
                    └────────────────┬────────────────┘
                                     ↓
                    ┌────────────────▼────────────────┐
                    │ Final Recommendation            │
                    │ (within min/max bounds)         │
                    └────────────────┬────────────────┘
                                     ↓
                    Update Mode: Recreate → Evict Pod
                                     ↓
                    New Pod with optimized resources

https://killercoda.com/cka-mock-practice/scenario/HHPA-with-CPU-Memory

HPA 

 Conceptual Diagram
Without HPA:
-----------
Fixed replicas (15) → High cost during low traffic
                   → Insufficient capacity during peak traffic

With HPA (minReplicas: 2, maxReplicas: 8):
------------------------------------------
Low traffic    → Scales down to 2 replicas ✅ (cost optimization)
Moderate load  → Maintains 3-5 replicas ✅ (balanced)
High traffic   → Scales up to 8 replicas ✅ (performance)
Stabilization  → Waits 5s before scaling down ✅ (prevents flapping)


How HPA Makes Scaling Decisions
1. Metrics Collection:
   - HPA queries metrics-server every 15 seconds (default)
   - Retrieves current CPU and memory usage for all Pods

2. Calculation (for each metric):
   desiredReplicas = ceil[currentReplicas × (currentMetric / targetMetric)]
   
   Example (CPU):
   - Current: 3 replicas using 240m CPU total (80m each)
   - Target: 80% of 100m request = 80m per pod
   - Current usage: 80m / 80m = 100%
   - Desired: ceil[3 × (100 / 80)] = ceil[3.75] = 4 replicas

3. Decision:
   - Takes the MAX of all metric calculations
   - If CPU suggests 4 and memory suggests 3 → scales to 4
   - Respects minReplicas (2) and maxReplicas (8) bounds

4. Stabilization:
   - Waits 5 seconds before scaling down (prevents rapid changes)
   - Scales up immediately when needed (default behavior)