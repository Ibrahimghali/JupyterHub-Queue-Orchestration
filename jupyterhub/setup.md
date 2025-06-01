
---

# 🚀 Déploiement de JupyterHub sur Ubuntu avec Kubeadm, MetalLB et NFS

## 📋 Prérequis

* 3 machines Ubuntu (1 master, 2 workers)
* Cluster Kubernetes installé via `kubeadm`
* Helm installé (`v3` recommandé)
* Serveur **NFS** configuré et exporté
* **MetalLB** installé pour la gestion des IPs externes

---

## 🛠️ Étapes

### 1. Installer MetalLB

Ajoutez le dépôt Helm et installez MetalLB :

```bash
helm repo add metallb https://metallb.github.io/metallb
helm repo update
helm install metallb metallb/metallb -n metallb-system --create-namespace
```

Créez un fichier `metallb-config.yaml` :

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: jupyterhub-pool
  namespace: metallb-system
spec:
  addresses:
    - 10.110.190.1-10.110.190.250  # <-- Adaptez à votre réseau local
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: jupyterhub-adv
  namespace: metallb-system
```

Appliquez la configuration :

```bash
kubectl apply -f metallb-config.yaml
```

---

### 2. Configurer le stockage NFS

#### Sur le **serveur NFS** :

```bash
sudo apt install nfs-kernel-server
sudo mkdir -p /srv/nfs/jupyterhub
sudo chown nobody:nogroup /srv/nfs/jupyterhub
sudo chmod 777 /srv/nfs/jupyterhub
```

Ajoutez cette ligne dans `/etc/exports` :

```exports
/srv/nfs/jupyterhub *(rw,sync,no_subtree_check,no_root_squash)
```

Redémarrez le service :

```bash
sudo exportfs -ra
sudo systemctl restart nfs-kernel-server
```

#### Sur les **nœuds Kubernetes** :

```bash
sudo apt install nfs-common
```

Créez un fichier `nfs-pv.yaml` :

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  nfs:
    path: /srv/nfs/jupyterhub
    server: <NFS_SERVER_IP>
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
```

Appliquez le volume :

```bash
kubectl apply -f nfs-pv.yaml
```

---

### 3. Ajouter le dépôt Helm de JupyterHub

```bash
helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/
helm repo update
```

---

### 4. Créer un secret Docker pour une image privée (optionnel)

```bash
kubectl create secret docker-registry jupyterhub \
  --docker-server=docker.io \
  --docker-username=your-username \
  --docker-password=your-password \
  --docker-email=your-email \
  --namespace=jupyterhub
```

> ⚠️ **Important :** Ne publiez jamais vos identifiants DockerHub dans un dépôt public !

---

### 5. Déployer JupyterHub avec Helm

Créez un fichier `config.yaml` :

```yaml
proxy:
  secretToken: "<randomly-generated-token>"
  service:
    type: LoadBalancer

singleuser:
  storage:
    type: dynamic
    dynamic:
      storageClass: ""
    pvcNameTemplate: "claim-{username}"
    capacity: 1Gi
  image:
    name: ibrahimghali/jupyter-custom
    tag: latest
    imagePullPolicy: Always
    pullSecrets:
      - name: jupyterhub

hub:
  db:
    type: sqlite-pvc
  extraVolumes:
    - name: hub-db-dir
      persistentVolumeClaim:
        claimName: nfs-pvc
  extraVolumeMounts:
    - name: hub-db-dir
      mountPath: /srv/jupyterhub
```

Créez le namespace et déployez :

```bash
kubectl create namespace jupyterhub

helm upgrade --install jupyterhub jupyterhub/jupyterhub \
  --namespace jupyterhub \
  --version=1.2.0 \
  --values config.yaml
```

---

### 6. Vérifier le déploiement

```bash
kubectl get all -n jupyterhub
kubectl get svc -n jupyterhub
```

L’adresse IP externe assignée par **MetalLB** devrait apparaître ici.

---

## 🔗 Ressources utiles

* 📘 [Documentation Helm Chart JupyterHub](https://jupyterhub.github.io/helm-chart/)
* 📝 [Article Medium original](https://medium.com/@texasdave2/deploy-jupyterhub-on-ubuntu-with-kubeadm-metallb-and-nfs-67ff41a394e4)

---