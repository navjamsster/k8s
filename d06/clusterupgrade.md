ssh controlplan >
  cat /etc/*release* ->



sudo apt update

apt     = Advanced Package Tool
        = Linux package manager for Ubuntu/Debian

update  = "Refresh the package INDEX (catalogue)"
        = NOT installing anything
        = NOT upgrading anything
        = Just READING the latest list of available packages
          from all registered repositories


apt update = going to a bookstore catalogue
             to see what books are available
             and at which versions

apt upgrade = actually BUYING the books

apt install = buying one SPECIFIC book 


kubeadm updgrade steps 

# On CONTROL PLANE:

# Step 1 — Change repo to new minor version
vim /etc/apt/sources.list.d/kubernetes.list
# change v1.35 → v1.36

# Step 2 — Unhold, update, install, hold
sudo apt-mark unhold kubeadm
sudo apt-get update
sudo apt-get install -y kubeadm='1.36.0-1.1'
sudo apt-mark hold kubeadm

# Step 3 — Plan and apply the upgrade
sudo kubeadm upgrade plan
sudo kubeadm upgrade apply v1.36.0

# Step 4 — Upgrade kubelet and kubectl too
sudo apt-mark unhold kubelet kubectl
sudo apt-get install -y kubelet='1.36.0-1.1' kubectl='1.36.0-1.1'
sudo apt-mark hold kubelet kubectl

# Step 5 — Restart kubelet
sudo systemctl daemon-reload
sudo systemctl restart kubelet

# On WORKER NODES (one at a time):
kubectl drain node01 --ignore-daemonsets
# (then SSH into worker and repeat apt steps for kubeadm + kubelet)
kubectl uncordon node01]


apt-mark = a utility to change the STATUS/FLAGS of packages

apt-mark hold      → put "DO NOT UPGRADE" flag on package
apt-mark unhold    → remove "DO NOT UPGRADE" flag
apt-mark auto      → mark as automatically installed
apt-mark manual    → mark as manually installed
apt-mark showhold  → list all packages currently on hold