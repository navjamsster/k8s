Securing cluster 

Enduser 
   Admin 
   Developers 
   Service Account 


1. Create 5 family member user key ,csr , kubernetese certificate sining request object  approve and use 


The Full Flow — Creating User (alice) from Scratch
text
STEP 1 → Generate private key        (alice's key)
STEP 2 → Create CSR                  (alice requests a certificate)
STEP 3 → Sign with cluster CA        (cluster trusts alice)
STEP 4 → Build kubeconfig for alice  (alice can now connect)
STEP 5 → Create Role                 (define what is allowed)
STEP 6 → Create RoleBinding          (give alice that role)
STEP 7 → Verify                      (test everything works)

# Step 1 
openssl genrsa -out alice.key 2048
#  Output: alice.key, This is alice's PRIVATE KEY

# Step 2 Create CSR with alice's identity embedded
openssl req -new -key alice.key -subj "/CN=alice/O=dev-team" -out alice.csr

alice.csr = certificate request = "please sign my identity"


# Step 3a — Base64 encode the CSR
cat alice.csr | base64 | tr -d "\n"
# Copy the output ← used in the YAML below

# Step 3b — Create the CSR object in Kubernetes
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: alice
spec:
  request: $(cat alice.csr | base64 | tr -d "\n")
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400     # 1 day
  usages:
  - client auth
EOF

# Step 3c — Approve the CSR (as admin)
kubectl get csr
# NAME    AGE   SIGNERNAME                            REQUESTOR   CONDITION
# alice   10s   kubernetes.io/kube-apiserver-client   admin       Pending

kubectl certificate approve alice
# → certificatesigningrequest.certificates.k8s.io/alice approved ✅

# Step 3d — Retrieve the signed certificate
kubectl get csr alice \
  -o jsonpath='{.status.certificate}' \
  | base64 --decode > alice.crt

# You now have:
#   alice.key  ← alice's private key
#   alice.crt  ← signed certificate (identity proof)


Step 4 — Build KubeConfig for Alice
bash
# Step 4a — Set cluster details (API server + CA)
kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/pki/ca.crt \
  --embed-certs=true \
  --server=https://$(kubectl get svc kubernetes \
    -o jsonpath='{.spec.clusterIP}'):443 \
  --kubeconfig=alice.kubeconfig

# Step 4b — Set user credentials (cert + key)
kubectl config set-credentials alice \
  --client-certificate=alice.crt \
  --client-key=alice.key \
  --embed-certs=true \
  --kubeconfig=alice.kubeconfig

# Step 4c — Create context linking cluster + user
kubectl config set-context alice-context \
  --cluster=kubernetes \
  --namespace=dev \
  --user=alice \
  --kubeconfig=alice.kubeconfig

# Step 4d — Set as current context
kubectl config use-context alice-context \
  --kubeconfig=alice.kubeconfig

# View the result
kubectl config view --kubeconfig=alice.kubeconfig


Step 5 — Create Role (Authorization)
bash
# Create namespace first
kubectl create namespace dev

# Create Role for alice in namespace dev
kubectl create role dev-role \
  --verb=get,list,watch,create,delete \
  --resource=pods,deployments,services \
  -n dev

Step 6 — Create RoleBinding (Attach Role to Alice)
bash
# Option A — Bind to user "alice" (matched by CN in certificate)
kubectl create rolebinding alice-dev-binding \
  --role=dev-role \
  --user=alice \
  -n dev

# Option B — Bind to group "dev-team" (matched by O in certificate)
# All users with O=dev-team get this role automatically
kubectl create rolebinding devteam-binding \
  --role=dev-role \
  --group=dev-team \
  -n dev

Step 7 — Test and Verify
bash
# Test as admin — can alice list pods in dev?
kubectl auth can-i list pods \
  --as=alice -n dev
# → yes ✅

# Test as admin — can alice access kube-system?
kubectl auth can-i list pods \
  --as=alice -n kube-system
# → no ✅

# Test as admin — can alice delete nodes?
kubectl auth can-i delete nodes --as=alice
# → no ✅

# Test as admin — list all alice's permissions in dev
kubectl auth can-i --list --as=alice -n dev
# Resources    Verbs
# pods         [get list watch create delete]
# deployments  [get list watch create delete]
# services     [get list watch create delete]

# Alice actually uses her kubeconfig
kubectl get pods -n dev \
  --kubeconfig=alice.kubeconfig
# → (lists pods or "No resources found") ✅

kubectl get pods -n kube-system \
  --kubeconfig=alice.kubeconfig
# → Error: Forbidden ✅ (correctly blocked)


Full Picture — Authentication vs Authorization
text
AUTHENTICATION (Who are you?)
─────────────────────────────
Alice presents:  alice.crt + alice.key
API server checks:
  1. Is this cert signed by our cluster CA? ✅
  2. Is the cert expired? No ✅
  3. Extract identity: CN=alice, O=dev-team
  4. Identity confirmed → alice is authenticated ✅

AUTHORIZATION (What can you do?)
─────────────────────────────────
API server now checks RBAC:
  1. Request: alice wants to list pods in namespace dev
  2. Check all RoleBindings in namespace dev for subject=alice
  3. Found: alice-dev-binding → role=dev-role
  4. dev-role allows: get,list,watch pods ✅
  5. Request allowed → returns pod list ✅

ANOTHER REQUEST:
  1. Request: alice wants to delete nodes
  2. Check ClusterRoleBindings for subject=alice → none
  3. Check RoleBindings in namespace dev for delete nodes → none
  4. No match found → DENIED ❌
  5. Returns: 403 Forbidden