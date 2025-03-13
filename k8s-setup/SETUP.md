
---

# Kubernetes Cluster Installation Documentation (Latest Versions)

## 1. Configure Hostname

Set the hostname for the master node and update `/etc/hosts`:

```bash
sudo hostnamectl set-hostname master
sudo reboot
```

Add entries for all nodes in `/etc/hosts`:

```bash
# Replace <IP_ADDRESS> with actual node IPs
echo "<master-ip> master" | sudo tee -a /etc/hosts
echo "<worker1-ip> worker1" | sudo tee -a /etc/hosts
echo "<worker2-ip> worker2" | sudo tee -a /etc/hosts
```

---

## 2. Disable Swap

Kubernetes requires swap to be disabled:

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

---

## 3. Load Required Kernel Modules

Load the necessary kernel modules for Kubernetes networking:

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

---

## 4. Configure Sysctl Settings

Enable IP forwarding and ensure proper networking:

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```

---

## 5. Install Containerd (Latest Version)

Install the latest version of Containerd as the container runtime:

### Fetch the latest version dynamically



### Download and install Containerd

```bash
curl -LO https://github.com/containerd/containerd/releases/download/v1.7.14/containerd-1.7.14-linux-amd64.tar.gz
sudo tar Cxzvf /usr/local containerd-1.7.14-linux-amd64.tar.gz
```

### Configure and enable Containerd

```bash
curl -LO https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
sudo mkdir -p /usr/local/lib/systemd/system/
sudo mv containerd.service /usr/local/lib/systemd/system/
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
sudo systemctl daemon-reload
sudo systemctl enable --now containerd

systemctl status containerd
```

---

## 6. Install Runc (Latest Version)

Install the latest version of `runc`:

### Fetch the latest version dynamically

```bash
RUNC_VERSION=$(curl -s https://api.github.com/repos/opencontainers/runc/releases/latest | grep 'tag_name' | cut -d '"' -f 4)
echo "Latest Runc version: $RUNC_VERSION"
```

### Download and install Runc

```bash
curl -LO https://github.com/opencontainers/runc/releases/download/${RUNC_VERSION}/runc.amd64
sudo install -m 755 runc.amd64 /usr/local/sbin/runc
```

---

## 7. Install CNI Plugins (Latest Version)

Install the latest version of CNI plugins:

### Fetch the latest version dynamically

```bash
CNI_VERSION=$(curl -s https://api.github.com/repos/containernetworking/plugins/releases/latest | grep 'tag_name' | cut -d '"' -f 4)
echo "Latest CNI version: $CNI_VERSION"
```

### Download and install CNI plugins

```bash
curl -LO https://github.com/containernetworking/plugins/releases/download/${CNI_VERSION}/cni-plugins-linux-amd64-${CNI_VERSION}.tgz
sudo mkdir -p /opt/cni/bin
sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-${CNI_VERSION}.tgz
```

---

## 8. Install Kubernetes Components (Latest Version)

Install the latest version of Kubernetes components (`kubeadm`, `kubelet`, and `kubectl`):

### Add Kubernetes repository

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

### Install the latest Kubernetes components

```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

---

## 9. Initialize Kubernetes Cluster

Initialize the Kubernetes cluster:

```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

Configure `kubectl` for the current user:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---

## 10. Install Calico Network Plugin (Latest Version)

Install the latest version of Calico for networking:

### Fetch the latest version dynamically

```bash
CALICO_VERSION=$(curl -s https://api.github.com/repos/projectcalico/calico/releases/latest | grep 'tag_name' | cut -d '"' -f 4)
echo "Latest Calico version: $CALICO_VERSION"
```

### Install Calico

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/${CALICO_VERSION}/manifests/tigera-operator.yaml
curl -LO https://raw.githubusercontent.com/projectcalico/calico/${CALICO_VERSION}/manifests/custom-resources.yaml
kubectl apply -f custom-resources.yaml
```

---

## 11. Join Worker Nodes

Retrieve the join command for worker nodes:

```bash
kubeadm token create --print-join-command
```

Run the join command on worker nodes.

---

## 12. Verify Cluster Installation

Check the status of the cluster:

```bash
kubectl get nodes -o wide
kubectl get pods -A
```

---

### Notes:

1. Always check the official Kubernetes 