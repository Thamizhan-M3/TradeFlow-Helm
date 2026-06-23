# NFS Server Setup Guide for TradeFlow (EC2)

This guide walks you through setting up an NFS server on a dedicated EC2 instance to serve as persistent storage for the TradeFlow microservices, specifically MongoDB.

## Prerequisites
- An EC2 instance running Ubuntu 22.04 LTS or Amazon Linux 2023.
- Security Group allowing inbound NFS traffic (TCP port 2049) from the Kubernetes worker nodes' IP range.

## Installation

### 1. Update and Install NFS Kernel Server (Ubuntu)
```bash
sudo apt update
sudo apt install nfs-kernel-server -y
```

### 2. Create the Export Directory
We will create a directory to store the TradeFlow data.
```bash
sudo mkdir -p /var/nfs/tradeflow
sudo chown nobody:nogroup /var/nfs/tradeflow
sudo chmod 777 /var/nfs/tradeflow
```

### 3. Configure Exports
Edit the `/etc/exports` file to allow the Kubernetes cluster to mount the directory.
```bash
echo "/var/nfs/tradeflow *(rw,sync,no_subtree_check,no_root_squash)" | sudo tee -a /etc/exports
```
*(Note: For production security, replace `*` with the specific CIDR block of your Kubernetes nodes, e.g., `10.0.0.0/16`)*

### 4. Apply Configuration and Start Service
```bash
sudo exportfs -a
sudo systemctl restart nfs-kernel-server
sudo systemctl enable nfs-kernel-server
```

## Kubernetes Integration
Update the `server` IP in `k8s/storageclass-nfs.yaml` to point to the private IP of this EC2 instance. Ensure your Kubernetes cluster has the NFS CSI driver or provisioner installed to dynamically provision PVs using the `nfs-storage` StorageClass.
