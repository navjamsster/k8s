in worker node ( Kubelet installed which communicate with Kube API server  )
1. PATH    → /var/lib/kubelet/pki/
               kubelet-client-current.pem  (CLIENT)
               kubelet.crt                 (SERVER)
2. COMMAND → openssl x509 -noout -text -in <file> | grep "Extended Key Usage" -A1

3. RESULTS → CLIENT: Issuer=kubernetes        EKU=TLS Web Client Authentication
             SERVER: Issuer=node-ca@timestamp  EKU=TLS Web Server Authentication

openssl	The SSL/TLS toolkit
x509	The certificate format standard (X.509 is the universal cert format)

openssl x509     → Open the certificate safe
  -noout         → Don't dump the raw junk
  -text          → Show me readable text
  -in <file>     → From this file
| grep -E        → Filter: find these words
  -A1            → Show 1 line after (to catch the value)

  
# Get ONLY the Issuer (no grep needed!)
openssl x509 -noout -issuer -in /var/lib/kubelet/pki/kubelet-client-current.pem

# Get ONLY the dates
openssl x509 -noout -dates -in /var/lib/kubelet/pki/kubelet-client-current.pem

# Get ONLY the subject
openssl x509 -noout -subject -in /var/lib/kubelet/pki/kubelet-client-current.pem