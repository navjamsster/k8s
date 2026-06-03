In kind, nodes are Docker containers — there is no SSH. You use docker exec instead.

kubectl get nodes or docker ps --format "table {{.Names}}\t{{.Status}}"

login to worker node 

docker exec -it k8s-c2-worker bash
docker exec -it k8s-c2-worker sh
docker exec -it k8s-c2-control-plane bash

# Check hostname
hostname

# Check kubelet status
systemctl status kubelet

# Inspect kubelet PKI certs
ls /var/lib/kubelet/pki/

# Inspect client cert
openssl x509 -noout -text \
  -in /var/lib/kubelet/pki/kubelet-client-current.pem \
  | grep -E "Issuer|Extended Key Usage" -A1

# Exit the node
exit