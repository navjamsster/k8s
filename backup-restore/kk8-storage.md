https://killercoda.com/cka-mock-practice/scenario/PersistentVolumeClaim-Mount-Storage
https://killercoda.com/cka-mock-practice/scenario/OpenEBS-Local-StorageClass

PersistentVolume (PV): A piece of storage in the cluster provisioned by an administrator or dynamically provisioned

PersistentVolumeClaim (PVC): A request for storage by a user. It's like a "purchase order" for storage resources

StorageClass: Provides a way to describe the "classes" of storage available. Different classes might map to quality-of-service levels, backup policies, or other policies

Dynamic Provisioning: Allows storage volumes to be created on-demand, eliminating the need for cluster administrators to pre-provision storage

PV (700Mi) ← binds to ← PVC (350Mi) ← used by ← Pod

The PVC binds to a PV when:

The storage class matches (or both are empty)
The access modes are compatible
The PV has sufficient capacity
Optional: volumeName explicitly specifies the PV


Storage Classes
storageClassName: Defines the "class" of storage (e.g., fast SSD, slow HDD, cloud storage)
Both PV and PVC must have matching storage classes to bind
local-path is commonly used for local storage provisioners


Local Volumes & Node Affinity
Local volumes are tied to a specific node:

The PV has nodeAffinity that restricts it to node01
Any pod using this PVC must schedule on node01
This is why all your pods ended up on the same node!


Storage Architecture:
=====================

Node01 (Physical Storage)
    │
    └─► /mnt/disks/ssd1 (700Mi)
            │
            └─► PersistentVolume: nginx-pv
                    │
                    ├─ Capacity: 700Mi
                    ├─ StorageClass: local-path
                    └─ NodeAffinity: node01
                        │
                        └─► PersistentVolumeClaim: nginx-pv-claim
                                │
                                ├─ Request: 350Mi
                                ├─ Namespace: nginx-cyperpunk
                                └─ Bound to: nginx-pv
                                    │
                                    └─► Deployment: nginx-scifi-portal
                                            │
                                            ├─ Volume: nginx-pv (references PVC)
                                            └─ VolumeMount: /usr/share/nginx/html
                                                │
                                                └─► 3 Pods on node01

root@node01:

findmnt | grep pods
df -h
findmnt
mount | grep nginx-pv

root@controlplane:~$ k exec nginx-scifi-portal-7874cc5cc9-hm7sb -n nginx-cyperpunk -- findmnt | grep 'usr/share/nginx/html'

|-/usr/share/nginx/html                     /dev/vda1[/mnt/disks/ssd1]                                                                                                                      ext4    rw,relatime,discard,errors=remount-ro,commit=30

Access Modes:

ReadWriteOnce (RWO): Single node can mount as read-write
ReadOnlyMany (ROX): Multiple nodes can mount as read-only
ReadWriteMany (RWX): Multiple nodes can mount as read-write


Use volumeName in PVC to explicitly bind to a specific PV


Always check PVC status with kubectl get pvc -n <namespace> (should be Bound)
Verify volume mounts with kubectl describe pod <pod-name> or kubectl exec into the pod
For local volumes, remember pods must be on the same node as the PV
Storage requests in PVC must be ≤ PV capacity
Match storageClassName between PV and PVC (or both should be empty)
Use volumeName in PVC to explicitly bind to a specific PV


How It Works Together
StorageClass (local-path)
        ↓
PersistentVolumeClaim (processor-cache)
        ↓ (triggers dynamic provisioning)
PersistentVolume (auto-created)
        ↓ (bound to)
PersistentVolumeClaim (Bound status)
        ↓ (referenced by)
Pod Volume (cache-storage)
        ↓ (mounted at)
Container Path (/cache)


Storage Provisioning Flow:
--------------------------
1. User creates PVC → Specifies size, StorageClass, access mode
2. StorageClass Provisioner → Detects new PVC
3. Provisioner creates PV → Allocates storage on the node
4. PV binds to PVC → Status changes to "Bound"
5. Pod references PVC → Via volume definition
6. Container mounts volume → Data persists across pod restarts

Volume Lifecycle:
----------------
PVC Created (Pending)
    ↓
Dynamic Provisioner Acts
    ↓
PV Created Automatically
    ↓
PVC Status → Bound
    ↓
Pod Scheduled with Volume
    ↓
Container Uses /cache Directory
    ↓
Pod Deleted → Data Persists
    ↓
New Pod Uses Same PVC → Data Available