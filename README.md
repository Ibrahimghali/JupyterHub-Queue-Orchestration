
---

# 🚀 JupyterHub sur Kubernetes avec File d’Attente et Optimisation des Ressources (CPU/GPU)

> 📚 Projet de fin d’études – Diplôme National d’Ingénieur en Informatique, Option Ingénierie des Données – Faculté des Sciences de Sfax

## 🎯 Objectif du Projet

Ce projet vise à déployer **JupyterHub** sur un cluster **Kubernetes hautement disponible**, en intégrant une **file d’attente Apache YuniKorn** pour la gestion des ressources CPU/GPU et en assurant la **persistance des données utilisateurs** via un stockage NFS.
L’environnement permet à plusieurs utilisateurs d’exécuter des notebooks en parallèle tout en maintenant la performance et l’équité d’accès aux ressources.

---

## 🧱 Architecture Déployée

### 🔁 Figure 1 : Architecture Globale

![Deployment Architecture](/assets/system_archtecture.png)

* **HAProxy** assure la répartition de charge vers les 3 nœuds maîtres Kubernetes.
* **JupyterHub** est déployé sur les nœuds **workers**.
* **NFS** fournit un stockage persistant accessible à tous les pods.

### 👤 Figure 2 : JupyterHub sur Kubernetes

![JupyterHub Kubernetes](/assets/jupyterhub_on_k8s.png)

* Chaque utilisateur (ex. Alice, Bob) dispose d’un environnement JupyterLab dédié.
* Le tout est orchestré dynamiquement par Kubernetes, via JupyterHub.

---

## 🔧 Technologies Utilisées

| Technologie         | Rôle                                                               |
| ------------------- | ------------------------------------------------------------------ |
| **Kubernetes**      | Orchestration de conteneurs (multi-node, HA, tolérance aux pannes) |
| **JupyterHub**      | Serveur multi-utilisateurs pour environnements JupyterLab          |
| **Helm**            | Gestionnaire de packages Kubernetes (chart de déploiement)         |
| **Apache YuniKorn** | Planification fine des ressources (CPU/GPU) avec file d’attente    |
| **NFS**             | Stockage partagé pour la persistance des notebooks et données      |
| **HAProxy**         | Load balancer entre les nœuds maîtres                              |

---

## ⚙️ Fonctionnalités Clés

* 🎓 Accès multi-utilisateurs à des environnements Jupyter isolés
* 📊 Gestion dynamique et équitable des ressources (YuniKorn)
* 💾 Sauvegarde des notebooks grâce à NFS
* 🔁 Tolérance aux pannes avec 3 nœuds maîtres + HAProxy
* 🧠 Compatible avec des workloads intensifs (IA/ML, calcul scientifique)

---

## 📈 Améliorations Futures

* 📉 **Monitoring avec Prometheus & Grafana** :

  * Visualisation temps réel de l’utilisation CPU/GPU, mémoire
  * Tableaux de bord personnalisés pour l’administration

---

## 📚 Références

* [Kubernetes Documentation](https://kubernetes.io/docs/)
* [Zero to JupyterHub](https://zero-to-jupyterhub.readthedocs.io/)
* [JupyterHub Docs](https://jupyterhub.readthedocs.io/)
* [Apache YuniKorn](https://yunikorn.apache.org/docs/)
* [Prometheus Docs](https://prometheus.io/docs/introduction/overview/)
* [Grafana Documentation](https://grafana.com/docs/)

---

## 🙌 Auteurs

* **[Ibrahim GHALI](mailto:ibrahim.elghai@outlook.com)**
* **[Hamza ZARAI](mailto:hamzazarai11@gmail.com)**


**Encadrante :** Mme Imen KETATA
**Université :** Faculté des Sciences de Sfax
**Date de soutenance :** 30/05/2025

---