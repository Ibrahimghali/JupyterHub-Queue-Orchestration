
---

# 🚀 JupyterHub on Kubernetes with Task Queue and Resource Optimization (CPU/GPU)

> 📚 Final Year Project – National Engineering Degree in Computer Science, Data Engineering Option – Faculty of Sciences of Sfax

## 🎯 Project Objective

This project aims to deploy **JupyterHub** on a **high-availability Kubernetes cluster**, integrating an **Apache YuniKorn-based task queue** for efficient CPU/GPU resource management and ensuring **user data persistence** through NFS storage.
The platform enables multiple users to run Jupyter notebooks in parallel while maintaining performance and fair resource distribution.

---

## 🧱 Deployment Architecture

### 🔁 Figure 1: Global System Architecture

![Deployment Architecture](/assets/system_archtecture.png)

* **HAProxy** acts as a load balancer for the three Kubernetes master nodes.
* **JupyterHub** is deployed on the **worker nodes**.
* **NFS** provides persistent storage accessible across all pods.

### 👤 Figure 2: JupyterHub on Kubernetes

![JupyterHub Kubernetes](/assets/jupyterhub_on_k8s.png)

* Each user (e.g., Alice, Bob) is provisioned a dedicated JupyterLab environment.
* Kubernetes dynamically manages these environments through JupyterHub.

---

## 🔧 Technologies Used

| Technology          | Purpose                                                          |
| ------------------- | ---------------------------------------------------------------- |
| **Kubernetes**      | Container orchestration (multi-node, HA, fault-tolerant)         |
| **JupyterHub**      | Multi-user server for managing isolated JupyterLab environments  |
| **Helm**            | Kubernetes package manager (for deployment automation)           |
| **Apache YuniKorn** | Fine-grained resource scheduling (CPU/GPU) with queue management |
| **NFS**             | Shared persistent storage for notebooks and user data            |
| **HAProxy**         | Load balancer for Kubernetes control plane (masters)             |

---

## ⚙️ Key Features

* 🎓 Multi-user access to isolated Jupyter environments
* 📊 Dynamic and fair resource scheduling with YuniKorn
* 💾 Persistent notebook storage using NFS
* 🔁 High availability with 3 master nodes + HAProxy
* 🧠 Supports intensive workloads (AI/ML, scientific computing)

---

## 📈 Future Improvements

* 📉 **Monitoring with Prometheus & Grafana**:

  * Real-time visualization of CPU/GPU/memory usage
  * Custom dashboards for cluster administration

---

## 🎓 Academic Presentation

![Project Presentation Cover](/assets/presentation_cover.png)

*Project defense presentation at the Faculty of Sciences of Sfax, May 30, 2025*

---

## 📚 References

* [Kubernetes Documentation](https://kubernetes.io/docs/)
* [Zero to JupyterHub](https://zero-to-jupyterhub.readthedocs.io/)
* [JupyterHub Docs](https://jupyterhub.readthedocs.io/)
* [Apache YuniKorn](https://yunikorn.apache.org/docs/)
* [Prometheus Documentation](https://prometheus.io/docs/introduction/overview/)
* [Grafana Documentation](https://grafana.com/docs/)

---

## 🙌 Authors

* **[Ibrahim GHALI](mailto:ibrahim.elghai@outlook.com)**
* **[Hamza ZARAI](mailto:hamzazarai11@gmail.com)**

**Supervisor:** Mrs. Imen KETATA
**University:** Faculty of Sciences of Sfax
**Defense Date:** May 30, 2025

---