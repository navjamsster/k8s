# Question 13 — NetworkPolicy Reference

## Exact question

There are two deployments, Frontend and Backend. Frontend is in the frontend namespace, Backend is in the backend namespace.

Task:

Look at the NetworkPolicy YAML files in /root/network-policies.

Decide which of the policies provides the functionality to allow interaction between the frontend and backend deployments in the least permissive way.

Deploy that YAML — do NOT delete or modify any existing deny-all policies.

## Source / related exam dump references

- [k8s/backup-restore/q13.md](k8s/backup-restore/q13.md)
- [CKA-PREP-2025-v2/Question-13 Network-Policy/Questions.bash](CKA-PREP-2025-v2/Question-13%20Network-Policy/Questions.bash)

## How to replicate in KillerKoda

1. Create the namespaces and deployments:
   - namespace: frontend
   - namespace: backend
   - deployment: frontend-deployment in frontend
   - deployment: backend-deployment in backend
   - expose backend as a ClusterIP service named backend-service

2. Create three NetworkPolicy YAML files under /root/network-policies:
   - policy-x: allows all ingress to all pods in backend namespace
   - policy-y: allows ingress from frontend namespace and an extra IP range
   - policy-z: allows ingress only from frontend namespace and frontend pods on TCP 80

3. Apply the policy that is the least permissive while still allowing the required frontend-to-backend communication.

## Correct choice

Use policy-z.

It is the most restrictive option that still allows the frontend deployment in the frontend namespace to reach the backend deployment on port 80.

## Reference YAML

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: policy-z
  namespace: backend
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: frontend
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 80
  policyTypes:
  - Ingress
```

## Apply command

```bash
kubectl apply -f /root/network-policies/network-policy-3.yaml
```

## Why this is the correct answer

- policy-x is too open because it allows all ingress.
- policy-y is still too open because it allows an extra IP block in addition to the frontend namespace.
- policy-z restricts ingress to the frontend namespace and specifically to pods labeled app=frontend, while only allowing TCP 80.

## Exam-style takeaway

When a question asks for the least permissive policy that still allows a specific service interaction, choose the policy that:

- targets the correct pod selector,
- uses namespace or pod selectors rather than broad allow-all rules,
- allows only the required port and protocol.
