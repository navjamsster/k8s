
Backup and Restore 
1. Confirms all pods running — safe to backup
	k get pods
	k get all 

2. check the url and cert files 
cat /etc/kubernetes/manifests/etcd.yaml 
cat /etc/kubernetes/manifests/etcd.yaml | grep file

3. create BACKUP

etcdctl snapshot save --help
etcdctl snapshot save --endpoints=127.0.0.1:2379 --carcert= --cert= --key=   /opt/snapshot-pre-boot.db
etcdctl snapshot save --endpoints=127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key /opt/snapshot-pre-boot.db

4. Restore 
a> STOP kube-apiserver (prevent writes to etcd during restore) , take a backup 
	ls /etc/kubernetes/manifests/ 
	mv /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/kube-apiserver.yaml
	
	cat /tmp/kube-apiserver.yaml
b> Confirm kube-apiserver stopped
	crictl ps | grep kube-apiserver
	crictl ps 

c> etcdutl snapshot restore /opt/snapshot-pre-boot.db --data-dir=/var/lib/etcd-backup
    This creates /var/lib/etcd-backup/member/ on the HOST filesystem
	Does NOT touch the running etcd yet

d>  Update etcd.yaml — ONLY change hostPath.path
 
  vi /etc/kubernetes/manifests/etcd.yaml 
	# BEFORE:
		volumes:
		  - hostPath:
			  path: /var/lib/etcd        ← OLD
			name: etcd-data

		# AFTER:
		volumes:
		  - hostPath:
			  path: /var/lib/etcd-backup ← NEW (matches your --data-dir)
			name: etcd-data

    DO NOT change:
		--data-dir flag (stays /var/lib/etcd — internal container path)
		volumeMounts.mountPath (stays /var/lib/etcd — internal container path)

e. Bring kube-apiserver back

	mv  /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/
	ls /etc/kubernetes/manifests/

5. Final verification — cluster healthy with restored data
 Wait and watch apiserver come back (~30-60s)
	crictl ps -a 
	crictl ps | grep kube-apiserver

	k get pods
	k get all 