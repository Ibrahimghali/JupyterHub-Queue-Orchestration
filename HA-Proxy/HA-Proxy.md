# HAProxy TCP Setup Guide for Kubernetes High Availability

This guide provides step-by-step instructions to configure HAProxy as a TCP load balancer for a highly available Kubernetes (K8s) cluster. HAProxy will distribute Kubernetes API server traffic (port 6443) across multiple control plane nodes in TCP mode, ensuring high availability. The setup integrates with the previously documented NFS shared storage and Helm installations for a cohesive K8s environment.

---

## Prerequisites

- **System**: Ubuntu/Debian-based system for the HAProxy server (separate from K8s nodes).
- **Kubernetes Cluster**:
  - Multiple control plane nodes (e.g., `192.168.1.10`, `192.168.1.11`, `192.168.1.12`) running the K8s API server on port 6443.
  - Worker nodes configured to access the control plane via HAProxy.
- **Network**:
  - A virtual IP or DNS name for HAProxy (e.g., `192.168.1.5` or `k8s-api.example.com`).
  - Connectivity between HAProxy, control plane nodes, and worker nodes.
- **NFS Setup**: Configured as per the prior guide (`/mnt/nfs_share` on the master, mounted as `/mnt/nfs_clientshare` on nodes).
- **Helm**: Installed as per the prior guide for managing K8s applications.
- **Privileges**: Root or sudo access on the HAProxy server.
- **Firewall**: Port 6443 (K8s API) open.

---

## Installation Steps

### 1. Install HAProxy

Install HAProxy on the dedicated load balancer server.

```bash
sudo apt update
sudo apt install haproxy -y
```

### 2. Enable HAProxy Service

Ensure HAProxy starts on boot.

```bash
sudo systemctl enable haproxy
```

Verify the service:

```bash
sudo systemctl status haproxy
```

### 3. Configure HAProxy for TCP Load Balancing

Edit the HAProxy configuration file to load balance TCP traffic for the K8s API server.

```bash
sudo nano /etc/haproxy/haproxy.cfg
```

Replace the default content with the following TCP-only configuration:

```
global
    log /dev/log local0
    maxconn 4096
    user haproxy
    group haproxy
    daemon

defaults
    log global
    mode tcp
    option tcplog
    timeout connect 5000
    timeout client 50000
    timeout server 50000
    retries 3

frontend k8s_api_frontend
    bind *:6443
    mode tcp
    default_backend k8s_api_backend

backend k8s_api_backend
    mode tcp
    balance roundrobin
    option tcp-check
    server controlplane1 192.168.1.10:6443 check
    server controlplane2 192.168.1.11:6443 check
    server controlplane3 192.168.1.12:6443 check
```

**Configuration explanation**:

- **global**: Sets process-wide parameters, including logging, max connections, and daemon mode.
- **defaults**: Configures TCP mode, TCP logging, timeouts, and retry behavior for all sections.
- **frontend k8s_api_frontend**: Listens on port 6443 for TCP traffic and forwards it to the backend.
- **backend k8s_api_backend**: Defines control plane nodes as backend servers, using `roundrobin` for load balancing. The `option tcp-check` performs TCP health checks to ensure node availability.
- Replace `192.168.1.10`, `192.168.1.11`, and `192.168.1.12` with your control plane node IPs. Adjust the number of servers as needed.

Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X` in nano).

### 4. Validate Configuration

Check for syntax errors in the configuration.

```bash
sudo haproxy -c -f /etc/haproxy/haproxy.cfg
```

If valid, proceed. Fix any reported errors.

### 5. Restart HAProxy

Apply the configuration by restarting HAProxy.

```bash
sudo systemctl restart haproxy
```

Confirm the service is running:

```bash
sudo systemctl status haproxy
```

### 6. Configure Firewall

Allow TCP traffic to the K8s API port (6443).

```bash
sudo ufw allow 6443
sudo ufw enable
```

For restricted access, allow only the subnet of worker nodes:

```bash
sudo ufw allow from 192.168.1.0/24 to any port 6443
```

---

## Integration with Kubernetes

### 1. Configure Kubeadm for HAProxy Endpoint

When initializing the K8s cluster, use the HAProxy IP or DNS as the control plane endpoint.

On the first control plane node:

```bash
sudo kubeadm init --control-plane-endpoint 192.168.1.5:6443 --upload-certs
```

Replace `192.168.1.5` with your HAProxy IP or DNS (e.g., `k8s-api.example.com`). Ensure port 6443 is specified.

For additional control plane nodes:

```bash
sudo kubeadm join 192.168.1.5:6443 --token <token> --discovery-token-ca-cert-hash <hash> --control-plane --certificate-key <key>
```

### 2. Configure Worker Nodes

Worker nodes should join the cluster using the HAProxy endpoint:

```bash
sudo kubeadm join 192.168.1.5:6443 --token <token> --discovery-token-ca-cert-hash <hash>
```

### 3. NFS Integration

Use the NFS setup (`/mnt/nfs_share` on the master, mounted as `/mnt/nfs_clientshare` on nodes) for persistent storage. Create a PersistentVolume (PV) for K8s:

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
    server: <nfs_server_ip>  # e.g., master_IP from NFS guide
    path: /mnt/nfs_share
```

Apply using `kubectl` or Helm:

```bash
kubectl apply -f nfs-pv.yaml
```

### 4. Helm Integration

Use Helm (installed as per the prior guide) to deploy K8s applications, such as Ingress controllers or monitoring tools, leveraging the NFS-backed storage for persistent data.

---

## Verification

### 1. Test TCP Load Balancing

Verify connectivity to the K8s API via HAProxy:

```bash
curl -k https://192.168.1.5:6443/version
```

This should return the K8s API version (e.g., `{"major":"1","minor":"28",...}`). The `-k` flag skips SSL verification for testing.

### 2. Verify High Availability

Temporarily stop the API server on one control plane node:

```bash
sudo systemctl stop kubelet
```

Retry the `curl` command. HAProxy should route traffic to another control plane node.

### 3. Monitor Backend Health

Check HAProxy logs to verify backend server status:

```bash
sudo journalctl -u haproxy
```

Look for lines indicating successful connections or health check failures. Alternatively, use `netstat` to confirm HAProxy is listening:

```bash
sudo netstat -tuln | grep 6443
```

---

## Troubleshooting Tips

- **API unreachable**: Ensure HAProxy is running (`sudo systemctl status haproxy`) and port 6443 is open (`sudo netstat -tuln | grep 6443`). Verify firewall rules and control plane IPs.
- **Health check failures**: Confirm `kube-apiserver` is running on each control plane node (`sudo netstat -tuln | grep 6443`). Check HAProxy logs for errors.
- **Kubeadm join failures**: Verify the `--control-plane-endpoint` matches the HAProxy IP and port 6443 is accessible (`telnet 192.168.1.5 6443`).
- **NFS issues**: Check mounts on nodes (`df -h /mnt/nfs_clientshare`) and NFS server permissions (`nobody:nogroup`).
- **Connection timeouts**: Increase `timeout client` or `timeout server` in `haproxy.cfg` if needed.

---

## Notes

- **HAProxy High Availability**: For production, deploy multiple HAProxy instances with Keepalived to manage a virtual IP, preventing HAProxy from being a single point of failure.
- **Scaling**: Add or remove control plane nodes in the `backend` section of `haproxy.cfg` and restart HAProxy.
- **Security**: Restrict port 6443 access to trusted subnets. Consider VPN or private networking for production.
- **Monitoring**: Export HAProxy logs to a SIEM or use a TCP-compatible monitoring tool (e.g., Prometheus with Blackbox Exporter) to track control plane availability.
- **Helm and NFS**: Use Helm to deploy stateful applications with NFS-backed storage for persistence.

For further details, consult the HAProxy documentation or Kubernetes HA guide.