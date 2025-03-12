
---

# Kubernetes Cluster Installation Documentation  

## 1. Configure Hostname  
```bash
sudo hostnamectl set-hostname master
sudo reboot
echo "@ip_address master" | sudo tee -a /etc/hosts
echo "@ip_address worker2" | sudo tee -a /etc/hosts
echo "@ip_address worker3" | sudo tee -a /etc/hosts
```  

## 2. Disable Swap  
```bash
swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```  

## 3. Load Required Kernel Modules  
```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```  

## 4. Configure Sysctl Settings for Kubernetes Networking  
```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```  

### Verification Steps  
Check if required modules are loaded:  
```bash
lsmod | grep br_netfilter
lsmod | grep overlay
```  
Check sysctl settings:  
```bash
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```  

## 5. Install Containerd  
```bash
curl -LO https://github.com/containerd/containerd/releases/download/v1.7.14/containerd-1.7.14-linux-amd64.tar.gz
sudo tar Cxzvf /usr/local containerd-1.7.14-linux-amd64.tar.gz
```  

Configure and enable Containerd:  
```bash
curl -LO https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
sudo mkdir -p /usr/local/lib/systemd/system/
sudo mv containerd.service /usr/local/lib/systemd/system/
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
sudo systemctl daemon-reload
sudo systemctl enable --now containerd
```  

Check Containerd status:  
```bash
systemctl status containerd
```  

## 6. Install Runc  
```bash
curl -LO https://github.com/opencontainers/runc/releases/download/v1.1.12/runc.amd64
sudo install -m 755 runc.amd64 /usr/local/sbin/runc
```  

## 7. Install CNI Plugins  
```bash
curl -LO https://github.com/containernetworking/plugins/releases/download/v1.5.0/cni-plugins-linux-amd64-v1.5.0.tgz
sudo mkdir -p /opt/cni/bin
sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.5.0.tgz
```  

## 8. Install Kubernetes Components  
```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```  

Add Kubernetes repository:  
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```  

Install Kubernetes components:  
```bash
sudo apt-get update
sudo apt-get install -y kubelet=1.29.6-1.1 kubeadm=1.29.6-1.1 kubectl=1.29.6-1.1 --allow-downgrades --allow-change-held-packages
sudo apt-mark hold kubelet kubeadm kubectl
```  

Verify installation:  
```bash
kubeadm version
kubelet --version
kubectl version --client
```  

Configure CRI:  
```bash
sudo crictl config runtime-endpoint unix:///var/run/containerd/containerd.sock
```  

## 9. Initialize Kubernetes Cluster  
```bash
sudo kubeadm init
```  

Configure `kubectl` for user access:  
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```  

## 10. Install Network Plugin (Calico)  
```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/tigera-operator.yaml
curl https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/custom-resources.yaml -O
kubectl apply -f custom-resources.yaml
```  

## 11. Retrieve Join Command for Worker Nodes  
```bash
kubeadm token create --print-join-command
```  

## 12. Verify Cluster Installation  
```bash
kubectl get nodes
```  

---

