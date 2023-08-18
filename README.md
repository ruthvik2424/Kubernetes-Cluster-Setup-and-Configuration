# Kubernetes-Cluster
Create Your Own Kubernetes Cluster Using Ubuntu 20.04 On Vmvare Workstation
# Steps to Install and Set Up Your Own Kubernetes Cluster

## Disable Firewall and Swap
We stop the firewall service and prevent it from starting at boot time to ensure it doesn't interfere with our Kubernetes setup. 
```bash
sudo systemctl stop ufw
sudo systemctl disable ufw
```
## We also disable the swap partition to create a smoother experience for Kubernetes.
```bash
sudo swapoff -a
sudo nano /etc/fstab
```
## Configure Kernel Modules
We add necessary kernel modules to handle networking and container overlay. These modules help in communication between containers and manage networking efficiently.

## Load Required Kernel Modules
```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```
## Enable The Loaded Modules
```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```
## Configure Kernel Parameters
```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
```
## Apply The Configured Settings
```bash
sudo sysctl --system
```
## Remove Previous Docker Versions
We remove any existing Docker-related packages to ensure a clean installation of Kubernetes components.
```bash

for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do
    sudo apt-get remove $pkg;
done
```
## Install Containerd
We install Containerd, a lightweight container runtime, which Kubernetes will use to manage containers.
## Set Up Repositories And Install Required Tools
```bash
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
```
## Configure Docker Repository And Install Containerd
```bash
sudo apt-get update
sudo apt-get install containerd.io
```
## Configure Containerd To Work With Kubernetes
```bash
sudo rm /etc/containerd/config.toml
sudo nano /etc/containerd/config.toml
```
## Add This Lines In The File
```bash
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
```
## Restart The Containerd Service
```bash
sudo systemctl restart containerd
```
## Install Kubernetes Components
We install essential Kubernetes components - kubelet, kubeadm, and kubectl - which are necessary to run and manage our cluster.
## Set Up Repositories And Install Required Tools
```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
```
## Configure Kubernetes Repository And Install Components
```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
```
#  Upto Here Similar Steps You Have To Perform On The Worker Node 

## Initialize The Kubernetes Master Node
We initialize the Kubernetes master node, which sets up the foundation for our cluster. We specify a network range for pods to communicate with each other
## Initialize The Master Node With Provided Settings
```bash
sudo kubeadm init --pod-network-cidr=10.32.0.0/12 --apiserver-advertise-address=machine_internal_ip
```
## Set Up The Configuration For Kubectl
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown \$(id -u):\$(id -g) $HOME/.kube/config
```
## Set Up Pod Networking
We deploy a network plugin (Weave) to enable communication between pods in our cluster.

## Apply The Weave Network Plugin
```bash
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```
## Check The Status Of The Pods
```bash
kubectl get pods -A
```
## Edit Weave Networking Config
We adjust the configuration of the Weave network plugin to match our network range.
## Edit the Weave Network Daemonset
```bash
kubectl edit ds weave-net -n kube-system
```
## Add This Line Below Other Environment Variables
```bash
- name: IPALLOC_RANGE
- value: 10.32.0.0/12  #(--pod-network-cidr) value
```
## Join Worker Node
## Finally, we use the command provided by the kubeadm init output to join worker nodes to the cluster.
```bash
kubectl join ....
```
## Use the provided command from the Kubeadm init command output to join worker nodes to the cluster.
