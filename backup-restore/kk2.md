https://killercoda.com/cka-mock-practice/scenario/analyze-netpol

The Critical Rule: namespaceSelector + podSelector
For cross-namespace communication with least privilege, you MUST use BOTH selectors in the SAME from item:

ingress:
- from:
  - namespaceSelector:        # Condition 1: Source namespace
      matchLabels:
        name: frontend
    podSelector:              # Condition 2: Specific pods
      matchLabels:            # ← Notice: SAME indentation level
        app: frontend         #   This creates AND logic

Missing namespaceSelector:

# ❌ WRONG - Only works within same namespace
from:
- podSelector:
    matchLabels:
      app: frontend
Problem: podSelector alone only matches pods in the backend namespace (where the policy is). Frontend pods are in a different namespace!

Missing podSelector:

# ❌ TOO PERMISSIVE - Allows all pods in namespace
from:
- namespaceSelector:
    matchLabels:
      name: frontend
Problem: Allows ALL pods in frontend namespace, not just app=frontend pods. Violates least privilege!