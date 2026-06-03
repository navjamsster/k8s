# 30-second diagnosis flow:
kubectl get pods -n kube-system | grep scheduler           # status?
kubectl logs <scheduler-pod> -n kube-system --previous     # why?
vi /etc/kubernetes/manifests/kube-scheduler.yaml           # fix it
kubectl get pods -n kube-system -w | grep scheduler        # verify


Symptom	Most Likely Cause	Fix Location

CrashLoopBackOff	Wrong file path in flag	/etc/kubernetes/manifests/kube-scheduler.yaml
Pending	CPU request too high	/etc/kubernetes/manifests/kube-scheduler.yaml
Error	Typo in binary name	/etc/kubernetes/manifests/kube-scheduler.yaml
Pods stuck Pending	Scheduler not running at all	Check all above


Static pod manifest 
/etc/kubernetes/manifest
