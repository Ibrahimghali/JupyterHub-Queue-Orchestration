
---
## 🎯 Objectif

Installer **YuniKorn**, un scheduler pour Kubernetes, à l'aide de **Helm**.

---

## 🧰 Étapes d'installation

### 1. Ajouter le dépôt Helm de YuniKorn

```bash
helm repo add yunikorn https://apache.github.io/yunikorn-release
```

### 2. Mettre à jour les dépôts Helm

```bash
helm repo update
```

### 3. Créer un namespace dédié

```bash
kubectl create namespace yunikorn
```

### 4. Installer YuniKorn dans le namespace

```bash
helm install yunikorn yunikorn/yunikorn --namespace yunikorn
```

---

## 🌐 Exposer l’interface Web de YuniKorn

### 5. Modifier le type de service en LoadBalancer (pour accès externe)

```bash
kubectl patch svc yunikorn-service -n yunikorn -p '{"spec": {"type": "LoadBalancer"}}'
```

> ⚠️ Si vous utilisez **MetalLB**, celui-ci attribuera automatiquement une adresse IP pour rendre le service accessible depuis l’extérieur du cluster.

---