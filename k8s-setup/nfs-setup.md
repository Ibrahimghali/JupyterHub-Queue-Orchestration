
---

# NFS Server and Client Setup Guide

This guide outlines the steps to configure an NFS server on a master node and NFS clients on worker nodes to share a directory over a network.

---

## Prerequisites
- Ubuntu-based systems (or compatible Linux distribution) on both master and worker nodes.
- Root or sudo privileges on all systems.
- Network connectivity between the master server and worker nodes.
- IP addresses of the master server (`master_IP`) and worker nodes (`worker1_IP`, `worker2_IP`, or subnet `192.168.1.0/24`).

---

## On the Master Server (NFS Server)

### 1. Install NFS Server
Update the package index and install the NFS kernel server package.

```bash
sudo apt update
sudo apt install nfs-kernel-server
```

### 2. Create and Configure the Shared Directory
Create a directory to share and set appropriate permissions.

```bash
sudo mkdir -p /mnt/nfs_share
sudo chown -R nobody:nogroup /mnt/nfs_share/
sudo chmod 777 /mnt/nfs_share/
```

### 3. Configure NFS Exports
Specify which directories to share and with which clients by editing the `/etc/exports` file.

```bash
sudo nano /etc/exports
```

Add one of the following configurations:

- **For specific worker IPs**:
  ```
  /mnt/nfs_share worker1_IP(rw,sync,no_subtree_check)
  /mnt/nfs_share worker2_IP(rw,sync,no_subtree_check)
  ```

- **For an entire subnet**:
  ```
  /mnt/nfs_share subnet/**(rw,sync,no_subtree_check)
  ```

**Options explained**:
- `rw`: Read and write permissions.
- `sync`: Synchronize file changes.
- `no_subtree_check`: Prevent subtree checking for improved reliability.

### 4. Export and Restart NFS Service
Apply the export configuration and restart the NFS service.

```bash
sudo exportfs -a
sudo systemctl restart nfs-kernel-server
```

### 5. Configure Firewall
Allow NFS traffic from the worker nodes through the firewall.

```bash
sudo ufw allow from worker1_IP to any port nfs
sudo ufw allow from worker2_IP to any port nfs
```

Or, for an entire subnet:
```bash
sudo ufw allow from 192.168.1.0/24 to any port nfs
```

Enable the firewall if not already active:
```bash
sudo ufw enable
```

---

## On Each Worker Node (NFS Client)

### 1. Install NFS Client
Install the NFS client package.

```bash
sudo apt update
sudo apt install nfs-common
```

### 2. Create Mount Point
Create a local directory to mount the NFS share.

```bash
sudo mkdir -p /mnt/nfs_clientshare
```

### 3. Mount the NFS Share
Mount the shared directory from the master server.

```bash
sudo mount master_IP:/mnt/nfs_share /mnt/nfs_clientshare
```

### 4. Configure Permanent Mount (Optional)
To automatically mount the NFS share on boot, edit the `/etc/fstab` file.

```bash
sudo nano /etc/fstab
```

Add the following line:
```
master_IP:/mnt/nfs_share  /mnt/nfs_clientshare  nfs  defaults  0  0
```

Save and exit. Test the fstab configuration:
```bash
sudo mount -a
```

---

## Verification

### On the Master Server
Create test files in the shared directory to verify access.

```bash
sudo touch /mnt/nfs_share/file{1..3}.txt
```

### On Each Worker Node
Check if the test files appear in the mounted directory.

```bash
ls -l /mnt/nfs_clientshare/
```

If the files (`file1.txt`, `file2.txt`, `file3.txt`) are listed, the NFS setup is working correctly.

---

## Troubleshooting Tips
- **Mount fails**: Ensure the NFS server is running (`sudo systemctl status nfs-kernel-server`) and the firewall allows NFS traffic.
- **Permission issues**: Verify directory permissions (`nobody:nogroup`) and export options (`rw`).
- **Network issues**: Confirm connectivity between master and worker nodes using `ping`.
- **Export errors**: Check `/etc/exports` syntax and re-run `sudo exportfs -a`.

---

