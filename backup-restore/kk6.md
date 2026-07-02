https://killercoda.com/cka-mock-practice/scenario/configure-kubernetes-gateway-api


Internet Traffic (HTTPS)
         ↓
    [Gateway]
    (TLS Termination)
         ↓
    [HTTPRoute]
    (Path-based Routing)
         ↓
    ┌────────┬──────────┬──────────┐
    │        │          │          │
/available  /books  /travellers
    │        │          │          │
    ↓        ↓          ↓          ↓
[Service]  [Service]  [Service]
available   books    travellers
    │        │          │          │
    ↓        ↓          ↓          ↓
[Pods]     [Pods]     [Pods]