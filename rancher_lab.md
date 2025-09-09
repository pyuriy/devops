# Rancher Lab: Deploy and Manage a Kubernetes Cluster

This lab provides step-by-step instructions to practice using Rancher to deploy and manage a Kubernetes cluster. You'll install Rancher, create a cluster, deploy an application, and perform basic management tasks.

## Lab Objectives
- Install Rancher using Docker
- Create a Kubernetes cluster using Rancher
- Deploy a sample application (nginx)
- Monitor and manage the cluster
- Perform a simple backup of the cluster

## Prerequisites
- A Linux machine (Ubuntu 20.04+ recommended) with:
  - 4GB RAM, 2 CPUs
  - Docker installed (`sudo apt install docker.io`)
  - Internet access
- Basic knowledge of Docker and Kubernetes
- A text editor (e.g., `nano` or `vim`)

## Lab Steps

### Step 1: Install Rancher
1. **Run Rancher in Docker**:
   ```bash
   sudo docker run -d --restart=unless-stopped \
     -p 80:80 -p 443:443 \
     --privileged \
     rancher/rancher:latest
   ```
2. **Verify Rancher is running**:
   ```bash
   docker ps
   ```
   Look for the `rancher/rancher` container with ports `80` and `443`.
3. **Access Rancher UI**:
   - Open a browser and navigate to `https://<server-ip>`.
   - Ignore self-signed certificate warnings (if any).
   - Set a new admin password when prompted and log in.

### Step 2: Create a Kubernetes Cluster
1. **Log in to Rancher UI**:
   - Use the admin credentials set in Step 1.
2. **Create a new cluster**:
   - Click **Clusters** > **Create**.
   - Select **Custom** (for a simple lab setup).
   - Name the cluster `lab-cluster`.
   - Leave default settings for Kubernetes version and networking (e.g., Canal).
   - Click **Next**.
3. **Add a node**:
   - Select all roles: **etcd**, **Control Plane**, **Worker**.
   - Copy the generated Docker command for the node.
4. **Run the node command**:
   - On your Linux machine, paste and run the copied Docker command (starts the Rancher agent).
   - Example:
     ```bash
     sudo docker run -d --privileged --restart=unless-stopped \
       -p 80:80 -p 443:443 \
       rancher/rancher-agent:v2.8.2
     ```
5. **Wait for cluster activation**:
   - In the Rancher UI, monitor the cluster status until it shows **Active** (takes ~2-5 minutes).

### Step 3: Deploy a Sample Application (nginx)
1. **Access the cluster**:
   - In Rancher UI, click **Clusters** > **lab-cluster**.
   - Navigate to **Workloads** > **Deploy**.
2. **Deploy nginx**:
   - Name: `nginx-app`
   - Image: `nginx:latest`
   - Namespace: `default`
   - Port Mapping: Map container port `80` to host port `8080` (type: ClusterIP).
   - Click **Launch**.
3. **Verify deployment**:
   - Check **Workloads** to ensure `nginx-app` is **Active**.
   - Run on your Linux machine:
     ```bash
     curl http://localhost:8080
     ```
     You should see the nginx welcome page HTML.

### Step 4: Enable Monitoring
1. **Enable cluster monitoring**:
   - In Rancher UI, go to **Clusters** > **lab-cluster** > **Tools** > **Monitoring**.
   - Enable Prometheus and Grafana with default settings.
   - Click **Save**.
2. **View dashboards**:
   - Once active, click **Grafana** link in the Monitoring section.
   - Explore default dashboards (e.g., node CPU/memory usage).
3. **Set up a simple alert**:
   - Go to **Tools** > **Alerting**.
   - Create an alert rule:
     - Name: `HighCPU`
     - Condition: Node CPU usage > 80%
     - Notification: Log to Rancher (for simplicity).
   - Save and test by stressing the node (e.g., run `stress` if installed).

### Step 5: Backup the Cluster
1. **Create an etcd backup**:
   - In Rancher UI, go to **Clusters** > **lab-cluster** > **Tools** > **Backups**.
   - Click **Create Backup**.
   - Name: `lab-backup`.
   - Select **One-time backup** and save.
2. **Verify backup**:
   - Check the **Backups** section to confirm `lab-backup` is listed.
3. **Simulate a restore** (optional, read-only):
   - View the backup details and note the snapshot ID for future restores.

### Step 6: Clean Up
1. **Delete the application**:
   - Go to **Workloads** > **nginx-app** > **Delete**.
2. **Remove the cluster**:
   - Go to **Clusters** > **lab-cluster** > **Delete**.
   - Confirm deletion.
3. **Stop Rancher**:
   ```bash
   docker stop <rancher-container-id>
   docker rm <rancher-container-id>
   ```

## Troubleshooting
- **Rancher UI inaccessible**: Ensure Docker container is running (`docker ps`) and ports `80`/`443` are open (`sudo netstat -tuln`).
- **Cluster not active**: Verify the Rancher agent container is running and has network access to the Rancher server.
- **Curl fails**: Check if the nginx pod is running (`kubectl get pods`) and port mapping is correct.

## Additional Notes
- **Scaling**: Add more nodes by running additional agent commands from the cluster creation page.
- **Kubectl access**: Download the kubeconfig from **Clusters** > **lab-cluster** > **Kubeconfig File** and use with `kubectl`.
- **Explore**: Try deploying other apps from the Rancher Catalog (e.g., WordPress) or enable logging.

## Resources
- Rancher Docs: https://rancher.com/docs/
- Rancher CLI: https://rancher.com/docs/rancher/v2.x/en/cli/
- Kubernetes Basics: https://kubernetes.io/docs/tutorials/
