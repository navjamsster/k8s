# Option 1 — Direct via mapped host port (if you used kind-config.yaml above)
curl http://localhost:30080

# Option 2 — kubectl port-forward (works without extra port mapping)
kubectl port-forward svc/nginx-demo-svc 8080:80 -n demo
# Then open: http://localhost:8080