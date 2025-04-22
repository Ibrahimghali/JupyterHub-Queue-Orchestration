
---

# Documentation d'Installation du Cluster Kubernetes (Dernières Versions)

## 1. Configuration du Nom d'Hôte

Définir le nom d'hôte pour le nœud maître et mettre à jour `/etc/hosts` :

```bash
sudo hostnamectl set-hostname master
sudo reboot
```

Ajouter les entrées pour tous les nœuds dans `/etc/hosts` :

```bash
# Remplacez <IP_ADDRESS> par les adresses IP réelles des nœuds
echo "<master-ip> master" | sudo tee -a /etc/hosts
echo "<worker1-ip> worker1" | sudo tee -a /etc/hosts
echo "<worker2-ip> worker2" | sudo tee -a /etc/hosts
```

---

## 2. Désactiver le Swap

Kubernetes nécessite la désactivation du swap :

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

---

## 3. Charger les Modules du Noyau Requis

Charger les modules nécessaires pour le réseau Kubernetes :

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

---

## 4. Configurer les Paramètres Sysctl

Activer le routage IP et assurer une bonne gestion du réseau :

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```

---

## 5. Installer Containerd (Dernière Version)

Installation de la dernière version de Containerd comme runtime de conteneur :

### Télécharger et installer Containerd

```bash
curl -LO https://github.com/containerd/containerd/releases/download/v1.7.14/containerd-1.7.14-linux-amd64.tar.gz
sudo tar Cxzvf /usr/local containerd-1.7.14-linux-amd64.tar.gz
```

### Configurer et activer Containerd

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

## 6. Installer Runc (Dernière Version)

Installation de la dernière version de `runc` :

### Récupérer la dernière version dynamiquement

```bash
RUNC_VERSION=$(curl -s https://api.github.com/repos/opencontainers/runc/releases/latest | grep 'tag_name' | cut -d '"' -f 4)
echo "Dernière version de Runc : $RUNC_VERSION"
```

### Télécharger et installer Runc

```bash
curl -LO https://github.com/opencontainers/runc/releases/download/${RUNC_VERSION}/runc.amd64
sudo install -m 755 runc.amd64 /usr/local/sbin/runc
```

---

## 7. Installer les Plugins CNI (Dernière Version)

Installation de la dernière version des plugins CNI :

### Récupérer la dernière version dynamiquement

```bash
CNI_VERSION=$(curl -s https://api.github.com/repos/containernetworking/plugins/releases/latest | grep 'tag_name' | cut -d '"' -f 4)
echo "Dernière version de CNI : $CNI_VERSION"
```

### Télécharger et installer les plugins CNI

```bash
curl -LO https://github.com/containernetworking/plugins/releases/download/${CNI_VERSION}/cni-plugins-linux-amd64-${CNI_VERSION}.tgz
sudo mkdir -p /opt/cni/bin
sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-${CNI_VERSION}.tgz
```

---

## 8. Installer les Composants Kubernetes (Dernière Version)

Installation de `kubeadm`, `kubelet` et `kubectl` :

### Ajouter le dépôt Kubernetes

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

### Installer les composants Kubernetes

```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

---

## 9. Initialiser le Cluster Kubernetes

Initialisation du cluster Kubernetes :

```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

Configurer `kubectl` pour l'utilisateur actuel :

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---

## 10. Installer le Plugin Réseau Calico (Dernière Version)

Installation de la dernière version de Calico pour le réseau :

### Récupérer la dernière version dynamiquement

```bash
CALICO_VERSION=$(curl -s https://api.github.com/repos/projectcalico/calico/releases/latest | grep 'tag_name' | cut -d '"' -f 4)
echo "Dernière version de Calico : $CALICO_VERSION"
```

### Installer Calico

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/${CALICO_VERSION}/manifests/tigera-operator.yaml
curl -LO https://raw.githubusercontent.com/projectcalico/calico/${CALICO_VERSION}/manifests/custom-resources.yaml
kubectl apply -f custom-resources.yaml
```

---

## 11. Rejoindre les Nœuds Travailleurs

Récupérer la commande pour rejoindre les nœuds :

```bash
kubeadm token create --print-join-command
```

Exécuter cette commande sur les nœuds travailleurs.

---

## 12. Vérifier l'Installation du Cluster

Vérifier l'état du cluster :

```bash
kubectl get nodes -o wide
kubectl get pods -A
```

---

### Remarque :

1. Toujours consulter la documentation officielle de Kubernetes.
2. Appliquer Calico en exécutant la commande :

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```
