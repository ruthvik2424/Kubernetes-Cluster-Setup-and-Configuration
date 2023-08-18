# Kubernetes-Cluster
## Setup Your Own Kubernetes Cluster Using Kubeadm On Ubuntu 20.04 

In this project, the goal is to set up a Kubernetes cluster on a virtualized environment using VMware Workstation, with Ubuntu 20.04 as the operating system. Kubernetes is a powerful container orchestration platform that enables the management and deployment of containerized applications. VMware Workstation provides the virtualization infrastructure necessary to create and manage virtual machines for this purpose.

## The process involves several key steps:

**Environment Setup**: Ensure that you have VMware Workstation installed on your host machine and a copy of Ubuntu 20.04 available for installation.

**Virtual Machine Creation**: Open VMware Workstation and create virtual machines that will serve as nodes in the Kubernetes cluster. These nodes will include a master node responsible for managing the cluster and one or more worker nodes for running applications.

**Ubuntu 20.04 Installation**: Install Ubuntu 20.04 on each of the virtual machines created in the previous step. Configure network settings and update the operating system to the latest packages.

**Kubernetes Installation**: Use the terminal within each virtual machine to install Kubernetes components. This includes installing the kubeadm, kubelet, and kubectl tools. Initiate the Kubernetes cluster on the master node using kubeadm.

**Cluster Configuration**: Configure kubectl on the master node to communicate with the cluster. Set up networking, such as choosing a network plugin for pod communication.

**Worker Node Joining**: Use the token provided by the kubeadm init command to join worker nodes to the cluster. This enables them to participate in workload execution.

**Testing and Deployment**: Verify the health of the Kubernetes cluster using kubectl commands. Deploy sample applications to ensure that the cluster is functioning as expected.

**Cluster Management**: Explore Kubernetes concepts such as pods, services, deployments, and namespaces to gain a deeper understanding of how the cluster operates.

**By the end of this project, you'll have successfully created a Kubernetes cluster on Ubuntu 20.04 using VMware Workstation. This will provide you with a platform to deploy, manage, and scale containerized applications with ease.**

# Steps to Install and Set Up Your Own Kubernetes Cluster

## Disable Firewall and Swap
We stop the firewall service and prevent it from starting at boot time to ensure it doesn't interfere with our Kubernetes setup. 
```bash
sudo systemctl stop ufw
sudo systemctl disable ufw
```
We also disable the swap partition to create a smoother experience for Kubernetes.
```bash
sudo swapoff -a
sudo nano /etc/fstab
# Remove the Swap Partition line  
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
## Add Docker’s official GPG key
```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```
## Use the following command to set up the repository
```bash
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
## Update The Repository And Install Containerd
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
# apt-transport-https may be a dummy package; if so, you can skip that package
sudo apt-get install -y apt-transport-https ca-certificates curl
```
## Download the public signing key for the Kubernetes package repositories
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
##  Add the appropriate Kubernetes apt repository:
```bash
# This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
## Update the apt package index, install kubelet, kubeadm and kubectl
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
# Don't Clear the Output It is Required Further
# Change the machine_internal_ip with the actual ip address of the Machine
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
Finally, we use the command provided by the kubeadm init output to join worker nodes to the cluster.
```bash
kubectl join ....
```
## Use the provided command from the Kubeadm init command output to join worker nodes to the cluster.

## Check That The Worker Node Has Join SuccessFully And Are In Ready State Or Not And All Pods Are In Running State
```bash
kubectl get nodes
kubectl get pods -A
```
## If Not In Ready State Wait For Some Time And Let It Get Setup

## If It Is Ready State Go For Your First Deployment Using The File Provide In The Repository 
```bash
git clone https://github.com/ruthvik2424/kubernetes-cluster.git
kubectl apply -f first-deployment.yaml
```
## Or
```bash
kubectl create deployment nginx-deploy --image=nginx
```
## Now You Have SuccessFully Deployed Your Own Kubernates Cluster.
