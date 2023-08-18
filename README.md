# kubernetes-cluster
create a kubernetes cluster on ubuntu 20.04 on Vmvare Workstation
# Steps to Install and Set Up Your Own Kubernetes Cluster

## Disable Firewall and Swap

We stop the firewall service and prevent it from starting at boot time to ensure it doesn't interfere with our Kubernetes setup. We also disable the swap partition to create a smoother experience for Kubernetes.

```bash
sudo systemctl stop ufw
sudo systemctl disable ufw

sudo swapoff -a
sudo nano /etc/fstab

Configure Kernel Modules
We add necessary kernel modules to handle networking and container overlay. These modules help in communication between containers and manage networking efficiently.

# Load required kernel modules
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

# Enable the loaded modules
sudo modprobe overlay
sudo modprobe br_netfilter

# Configure kernel parameters
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply the configured settings
sudo sysctl --system

Remove Previous Docker Versions
We remove any existing Docker-related packages to ensure a clean installation of Kubernetes components.

for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do
    sudo apt-get remove $pkg;
done

Install Containerd
We install Containerd, a lightweight container runtime, which Kubernetes will use to manage containers.

# Set up repositories and install required tools
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg

# Configure Docker repository and install Containerd
# (Commands to add repository key and configure repository)
sudo apt-get update
sudo apt-get install containerd.io

# Configure Containerd to work with Kubernetes
# (Commands to replace the configuration file content)
sudo rm /etc/containerd/config.toml
sudo nano /etc/containerd/config.toml

# Restart the Containerd service
sudo systemctl restart containerd

Install Kubernetes Components
We install essential Kubernetes components - kubelet, kubeadm, and kubectl - which are necessary to run and manage our cluster.

# Set up repositories and install required tools
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl

# Configure Kubernetes repository and install components
# (Commands to add repository key and configure repository)
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl

Initialize the Kubernetes Master Node
We initialize the Kubernetes master node, which sets up the foundation for our cluster. We specify a network range for pods to communicate with each other.

# Initialize the master node with provided settings
sudo kubeadm init --pod-network-cidr=10.32.0.0/12 --apiserver-advertise-address=machine_internal_ip

# Set up the configuration for kubectl
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown \$(id -u):\$(id -g) $HOME/.kube/config

Set Up Pod Networking
We deploy a network plugin (Weave) to enable communication between pods in our cluster.

# Apply the Weave network plugin
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml

# Check the status of the pods
kubectl get pods -A

Edit Weave Networking Config
We adjust the configuration of the Weave network plugin to match our network range.

# Edit the Weave network daemonset
kubectl edit ds weave-net -n kube-system

# Add this line below other environment variables
# IPALLOC_RANGE=10.32.0.0/12  (--pod-network-cidr)

Join Worker Node
Finally, we use the command provided by the kubeadm init output to join worker nodes to the cluster.
kubectl join ....

Use the provided command from the Kubeadm init command output to join worker nodes to the cluster.
