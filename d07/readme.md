This question needs to be solved on node node01. To access the node using SSH, use the credentials below:

username: bob
password: caleston123

As an administrator, you need to prepare node01 to install kubernetes. One of the steps is installing a container runtime. Install the cri-docker_0.3.16.3-0.debian.deb package located in /root and ensure that the cri-docker service is running and enabled to start on boot.


Use dpkg to install the package and systemctl to manage the service.

SSH to node01 as follows, using password caleston123:

ssh bob@node01

Switch to root using sudo -i or prefix the commands below using sudo.

The cri-docker_0.3.16.3-0.debian.deb package is located in the /root directory. Use dpkg to install the package:

dpkg -i /root/cri-docker_0.3.16.3-0.debian.deb

After installing the package, start the cri-docker service and enable it to start on boot:

systemctl start cri-docker
systemctl enable cri-docker

Verify that the cri-docker service is running:

systemctl is-active cri-docker

Check that it is enabled to start on boot:

systemctl is-enabled cri-docker

You should see active and enabled as the output for both commands


Storage class imperative way 

kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup nginx-resolver-service
kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup nginx-resolver-service > /root/CKA/nginx.svc

kubectl run nginx-resolver --image=nginx
kubectl expose pod nginx-resolver --name=nginx-resolver-service --port=80 --target-port=80 --type=ClusterIP



================

systemctl status <svc>       → is it running? last logs?
systemctl start <svc>        → start now
systemctl stop <svc>         → stop now
systemctl restart <svc>      → stop + start
systemctl enable <svc>       → start on boot
systemctl disable <svc>      → don't start on boot
systemctl enable --now <svc> → enable + start together ← most useful
systemctl daemon-reload      → re-read unit files (after editing .service)
systemctl is-active <svc>    → quick active/inactive check
systemctl is-enabled <svc>   → enabled/disabled check
systemctl cat <svc>          → show full unit file
systemctl list-units --state=failed → find all failed services

journalctl -u <svc> -n 50    → last 50 log lines
journalctl -xeu <svc>        → verbose recent logs ← troubleshooting ✅
journalctl -u <svc> -f       → live log stream


SCENARIO                              COMMAND(S)
──────────────────────────────────────────────────────────────────────────
Node is NotReady                      systemctl status kubelet
                                      systemctl restart kubelet
                                      journalctl -u kubelet -n 50

kubelet not starting after upgrade    systemctl daemon-reload
                                      systemctl restart kubelet
                                      journalctl -xeu kubelet

Bootstrap new worker node             systemctl enable --now containerd
                                      systemctl enable --now kubelet

After editing kubelet config          systemctl daemon-reload
                                      systemctl restart kubelet
                                      systemctl status kubelet

Verify service will survive reboot    systemctl is-enabled kubelet

Check why service crashed             journalctl -u kubelet -n 50

See all failed services on node       systemctl list-units --state=failed

View exact kubelet startup command    systemctl cat kubelet


The CKA Troubleshooting Golden Flow
The most tested scenario in the Troubleshooting domain (30%) is a broken node. This is the flow every time:

text
1. kubectl get nodes
   → node01 is NotReady

2. SSH to node01
   ssh node01

3. Check kubelet
   systemctl status kubelet
   → active? failed? inactive?

4. If failed → check logs
   journalctl -u kubelet -n 50
   → what's the actual error?

5. Fix the issue
   (wrong config, missing file, wrong cert, etc.)

6. Reload + restart
   sudo systemctl daemon-reload
   sudo systemctl restart kubelet

7. Verify
   systemctl status kubelet
   → active (running) ✅

8. Back on control plane
   kubectl get nodes
   → node01 Ready ✅


apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: example-gateway
spec:
  gatewayClassName: example-class
  listeners:
	- protocol: HTTPS
	  name: https
	  port: 443
	  hostname: foo.example.com
	  tls:
		mode: Terminate
		certificateRefs:
		  - kind: Secret
			group: ""  #  Can be omitted if it is from the main API group (empty by default)
			name: tls-secret


apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: example-httproute
spec:
  parentRefs:
  - name: example-gateway
  hostnames:
  - "foo.example.com"
  rules:
  - matches:
	- path:
		type: PathPrefix
		value: /
	backendRefs:
	- name: example-svc
	  port: 8080  



      In this task, we will use the helm commands and jq tool. Here are the steps: -



Run the helm ls command with -A option to list the releases deployed on all the namespaces using helm.
helm ls -A



We will use the jq tool to extract the image name from the deployments.
kubectl get deploy -n <NAMESPACE> <DEPLOYMENT-NAME> -o json | jq -r '.spec.template.spec.containers[].image'



Replace <NAMESPACE> with the namespace and <DEPLOYMENT-NAME> with the deployment name, which we get from the previous commands.

After finding the kodekloud/webapp-color:v1 image, use the helm uninstall to remove the deployed chart that are using this vulnerable image.


helm uninstall <RELEASE-NAME> -n <NAMESPACE> 