In the VS Code terminal (PowerShell), run:
Step 1 — Open Your PowerShell Profile

# Check if profile file exists
Test-Path $PROFILE

# Create it if it doesn't exist
New-Item -ItemType File -Path $PROFILE -Force

# Open it in VS Code
code $PROFILE

Step 2 — Paste These kubectl Aliases

# ─── kubectl Core Alias ──────────────────────────────────────
function k { kubectl $args }

# ─── Get Resources ───────────────────────────────────────────
function kg   { kubectl get $args }
function kgp  { kubectl get pods $args }
function kgpa { kubectl get pods --all-namespaces $args }
function kgn  { kubectl get nodes $args }
function kgs  { kubectl get svc $args }
function kgd  { kubectl get deploy $args }
function kgns { kubectl get namespaces $args }
function kgcm { kubectl get configmap $args }
function kgsec{ kubectl get secret $args }
function kgpv { kubectl get pv $args }
function kgpvc{ kubectl get pvc $args }
function kging{ kubectl get ingress $args }
function kgnp { kubectl get networkpolicy $args }

# ─── Describe ────────────────────────────────────────────────
function kd   { kubectl describe $args }
function kdp  { kubectl describe pod $args }
function kdn  { kubectl describe node $args }
function kds  { kubectl describe svc $args }

# ─── Apply / Delete ──────────────────────────────────────────
function kaf  { kubectl apply -f $args }
function kdf  { kubectl delete -f $args }
function krm  { kubectl delete $args }

# ─── Logs & Exec ─────────────────────────────────────────────
function kl   { kubectl logs $args }
function klf  { kubectl logs -f $args }
function kex  { kubectl exec -it $args }

# ─── Context / Namespace ─────────────────────────────────────
function kuc  { kubectl config use-context $args }
function kgc  { kubectl config get-contexts }
function kcc  { kubectl config current-context }
function kns  { kubectl config set-context --current --namespace=$args }

# ─── Useful Shortcuts ────────────────────────────────────────
function krun { kubectl run $args }
function kdry { kubectl $args --dry-run=client -o yaml }
function keti { kubectl exec -it $args -- /bin/bash }

# ─── kubectl autocompletion ──────────────────────────────────
kubectl completion powershell | Out-String | Invoke-Expression

Step 3 — Reload Profile
powershell

. $PROFILE

Step 4 — Test It
k get nodes
kgp -n kube-system
kgc
kd node k8s-c2-worker