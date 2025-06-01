
---

# 📘 Documentation d'Installation du Cluster Kubernetes (Dernières Versions)

---

## 🔧 1. Configuration du Nom d’Hôte & du Fichier Hosts

### Définir le nom d’hôte (sur le nœud maître) :

```bash
sudo hostnamectl set-hostname master
sudo reboot
```

### Mettre à jour `/etc/hosts` sur tous les nœuds :

```bash
echo "<master-ip> master" | sudo tee -a /etc/hosts
echo "<worker1-ip> worker1" | sudo tee -a /etc/hosts
echo "<worker2-ip> worker2" | sudo tee -a /etc/hosts
```

---

## 🚫 2. Désactiver le Swap

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

---

## 📦 3. Charger les Modules du Noyau Requis

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

---

## 🌐 4. Configurer les Paramètres Sysctl

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```

---

## 🐳 5. Installer Containerd (Dernière Version)

### Télécharger et installer Containerd :

```bash
curl -LO https://github.com/containerd/containerd/releases/download/v1.7.14/containerd-1.7.14-linux-amd64.tar.gz
sudo tar Cxzvf /usr/local containerd-1.7.14-linux-amd64.tar.gz
```

### Configurer et activer Containerd :

```bash
curl -LO https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
sudo mkdir -p /usr/local/lib/systemd/system/
sudo mv containerd.service /usr/local/lib/systemd/system/
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

sudo systemctl daemon-reload
sudo systemctl enable --now containerd
systemctl status containerd
```

---

## 🔐 6. Installer Runc (Dernière Version)

```bash
RUNC_VERSION=$(curl -s https://api.github.com/repos/opencontainers/runc/releases/latest | grep 'tag_name' | cut -d '"' -f 4)
echo "Dernière version de Runc : $RUNC_VERSION"

curl -LO https://github.com/opencontainers/runc/releases/download/${RUNC_VERSION}/runc.amd64
sudo install -m 755 runc.amd64 /usr/local/sbin/runc
```

---

## 🔌 7. Installer les Plugins CNI (Dernière Version)

```bash
CNI_VERSION=$(curl -s https://api.github.com/repos/containernetworking/plugins/releases/latest | grep 'tag_name' | cut -d '"' -f 4)
echo "Dernière version de CNI : $CNI_VERSION"

curl -LO https://github.com/containernetworking/plugins/releases/download/${CNI_VERSION}/cni-plugins-linux-amd64-${CNI_VERSION}.tgz
sudo mkdir -p /opt/cni/bin
sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-${CNI_VERSION}.tgz
```

---

## ☸️ 8. Installer Kubernetes (kubeadm, kubelet, kubectl)

### Ajouter le dépôt officiel :

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

### Installer Kubernetes :

```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

---

## 🚀 9. Initialiser le Cluster Kubernetes

### Pour un cluster **classique (non-HA)** :

```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

### Configurer `kubectl` pour l’utilisateur actuel :

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---

## 🌉 10. Installer Calico (Plugin Réseau - Dernière Version)

### Récupérer et appliquer Calico dynamiquement :

```bash
CALICO_VERSION=$(curl -s https://api.github.com/repos/projectcalico/calico/releases/latest | grep 'tag_name' | cut -d '"' -f 4)
echo "Dernière version de Calico : $CALICO_VERSION"

kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/${CALICO_VERSION}/manifests/tigera-operator.yaml
curl -LO https://raw.githubusercontent.com/projectcalico/calico/${CALICO_VERSION}/manifests/custom-resources.yaml
kubectl apply -f custom-resources.yaml
```

### Alternative (version documentée) :

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

---

## 🤝 11. Ajouter des Nœuds Travailleurs

### Générer la commande de jonction :

```bash
kubeadm token create --print-join-command
```

### Puis exécuter la commande sur chaque **nœud travailleur**.

---

## ✅ 12. Vérification du Cluster

```bash
kubectl get nodes -o wide
kubectl get pods -A
```

---

## 🛠️ Annexe : Initialisation du Cluster en Mode High Availability (HA)

### Exemple avec une IP de load balancer :

```bash
sudo kubeadm init \
  --control-plane-endpoint "LOAD_BALANCER_IP:6443" \
  --upload-certs \
  --pod-network-cidr=192.168.0.0/16
```

---

## 🔎 Remarques Finales

1. Toujours vérifier la [documentation officielle Kubernetes](https://kubernetes.io/fr/docs/home/).
2. Bien adapter les versions en fonction des exigences spécifiques.
3. Pour les clusters de production, sécuriser toutes les communications (SSL, pare-feu, etc.).

---