# Rancher Cheatsheet

## Overview
Rancher is an open-source platform for managing Kubernetes clusters across hybrid and multi-cloud environments. This cheatsheet covers common commands, configurations, and tips for using Rancher effectively.

## Installation
### Prerequisites
- Docker installed on the host
- Minimum 4GB RAM, 2 CPUs
- Supported OS: Ubuntu, CentOS, or RHEL

### Install Rancher (Single Node)
```bash
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  --privileged \
  rancher/rancher:latest
```

### Install Rancher with External TLS
```bash
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  -v /etc/ssl/certs:/etc/rancher/ssl \
  --privileged \
  rancher/rancher:latest \
  --no-cacerts
```

## CLI Commands (rancher CLI)
### Installation
```bash
# Linux
curl -sL https://releases.rancher.com/cli2/latest/rancher-linux-amd64.tar.gz | tar -xz -C /usr/local/bin/

# macOS
curl -sL https://releases.rancher.com/cli2/latest/rancher-darwin-amd64.tar.gz | tar -xz -C /usr/local/bin/
```

### Login to Rancher Server
```bash
rancher login https://<rancher-server-url> --token <api-token>
```

### Cluster Management
```bash
# List clusters
rancher clusters ls

# Create a cluster
rancher cluster create --name my-cluster

# Import an existing Kubernetes cluster
rancher cluster import <cluster-id>

# Delete a cluster
rancher cluster rm <cluster-id>
```

### Kubectl Context
```bash
# Get kubeconfig for a cluster
rancher cluster kubeconfig <cluster-id> > kubeconfig.yaml

# Use kubectl with Rancher
kubectl --kubeconfig kubeconfig.yaml get pods
```

## RKE (Rancher Kubernetes Engine)
### Install RKE CLI
```bash
# Linux
curl -sL https://releases.rancher.com/rke/latest/rke_linux-amd64 -o /usr/local/bin/rke
chmod +x /usr/local/bin/rke

# macOS
curl -sL https://releases.rancher.com/rke/latest/rke_darwin-amd64 -o /usr/local/bin/rke
chmod +x /usr/local/bin/rke
```

### Create RKE Cluster
1. Create `cluster.yml`:
```yaml
nodes:
  - address: <node-ip>
    user: ubuntu
    role: [controlplane,worker,etcd]
    ssh_key_path: ~/.ssh/id_rsa
```

2. Run RKE:
```bash
rke up --config cluster.yml
```

### Upgrade RKE Cluster
```bash
rke up --config cluster.yml --update-only
```

## Common Tasks
### Add a Node to Cluster
1. Edit `cluster.yml` to add a new node:
```yaml
nodes:
  - address: <new-node-ip>
    user: ubuntu
    role: [worker]
    ssh_key_path: ~/.ssh/id_rsa
```

2. Run:
```bash
rke up --config cluster.yml
```

### Backup and Restore
#### Backup Etcd
```bash
rancher cluster etcd-backup create <cluster-id>
```

#### List Backups
```bash
rancher cluster etcd-backup ls <cluster-id>
```

#### Restore Etcd
```bash
rancher cluster etcd-backup restore <cluster-id> <backup-id>
```

### Deploy an Application
```bash
# Deploy via Rancher CLI
rancher apps install <catalog-name> <app-name> --namespace <namespace>

# Example: Deploy nginx
rancher apps install rancher-library/nginx nginx --namespace default
```

## Configuration Tips
### Access Rancher UI
- Default URL: `https://<server-ip>`
- Default credentials: `admin` / (set during first login)

### Enable Monitoring
1. Navigate to Cluster > Tools > Monitoring
2. Enable Prometheus and Grafana
3. Configure alerts via Alerting rules

### Set Up Authentication
- **Local**: Default admin user
- **External**:
  ```bash
  # Configure LDAP/AD
  rancher auth config ldap
  ```

### Helm Charts
```bash
# Add Helm repo
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable

# Install Rancher via Helm
helm install rancher rancher-stable/rancher --namespace cattle-system
```

## Troubleshooting
### Check Rancher Logs
```bash
docker logs <rancher-container-id>
```

### Common Issues
- **Node not ready**: Verify SSH access and Docker status on nodes
- **API errors**: Check API token and network connectivity
- **Etcd issues**: Ensure sufficient disk space and network stability

### Reset Admin Password
```bash
docker exec <rancher-container-id> reset-password
```

## Useful Resources
- Official Docs: https://rancher.com/docs/
- Community: https://forums.rancher.com/
- GitHub: https://github.com/rancher/rancher
