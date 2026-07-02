https://killercoda.com/cka-mock-practice/scenario/crt-audit-image-verification

Certificate Lifecycle Flow
Certificate Created (day 0)
    ↓
Valid Period: 1 year (365 days)
    ↓
Warning Period: Last 30 days (kubeadm recommends proactive renewal)
    ↓
kubeadm certs renew all
    ↓
New Certificate (New 1-year validity)
    ↓
Control-plane components restart with new certs
    ↓
Worker node kubelets restart
    ↓
Cluster stability verified 


CA Certificate Renewal Flow (Critical Understanding!)
CA Certificate (day 0) - 10 years validity
    ↓
CA approaching expiration (year 9-10)
    ↓
Step 1: kubeadm certs renew ca
    ↓
New CA Certificate Generated (keeping same private key)
    ↓
⚠️ CRITICAL: ALL component certificates now INVALID!
    ↓
Step 2: kubeadm certs renew all (re-sign with new CA)
    ↓
Step 3: Restart all control-plane components
    ↓
Step 4: Restart all worker node kubelets
    ↓
Cluster stability verified ✅

kubeadm certs renew ca
kubeadm certs renew all


If you renew the CA → you MUST re-renew all other certificates!

CA renewed → All certs invalid → renew all → restart everything