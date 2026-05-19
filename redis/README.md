##  Redis Cluster on Single VM

**✅ Best DevOps-Automated Way: Redis Cluster on Single VM**

Here is the **most professional, clean, and maintainable** approach using Vagrant + shell provisioning + custom systemd services.

### Final Recommended `Vagrantfile`

Replace your existing `Vagrantfile` with this:

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"
  config.vm.box_download_insecure = true
  config.vm.hostname = "redis-cluster"

  config.vm.provider "virtualbox" do |vb|
    vb.name = "redis-cluster-vm"
    vb.memory = "4096"      # 4GB strongly recommended
    vb.cpus = 4
    vb.gui = false
  end

  # Forward ports for easy access from host (optional)
  (0..5).each do |i|
    port = 7000 + i
    config.vm.network "forwarded_port", guest: port, host: port, host_ip: "127.0.0.1"
  end

  # Fully Automated Provisioning
  config.vm.provision "shell", inline: <<-SHELL
    echo "========================================"
    echo "🚀 Redis Cluster Automated Setup (6 nodes)"
    echo "========================================"

    apt-get update -qq
    apt-get install -y redis-server redis-tools

    # Prepare user and directories
    id -u redis &>/dev/null || useradd -r -s /bin/false redis
    mkdir -p /etc/redis/cluster /var/lib/redis /var/log/redis
    chown -R redis:redis /var/lib/redis /var/log/redis /etc/redis
    chmod 755 /var/lib/redis /var/log/redis

    # ====================== Create 6 Redis Nodes ======================
    for i in {0..5}; do
      port=$((7000 + i))
      node_dir="/etc/redis/cluster/${port}"

      mkdir -p "${node_dir}" "/var/lib/redis/${port}"

      cat > "${node_dir}/redis_${port}.conf" <<EOF
port ${port}
bind 0.0.0.0
protected-mode no
dir /var/lib/redis/${port}
pidfile /var/run/redis/redis_${port}.pid
logfile /var/log/redis/redis_${port}.log
loglevel notice
daemonize no
appendonly no

# Cluster Configuration
cluster-enabled yes
cluster-node-timeout 5000
cluster-config-file nodes-${port}.conf
cluster-require-full-coverage no
EOF

      chown -R redis:redis "${node_dir}" "/var/lib/redis/${port}"
      chmod 644 "${node_dir}/redis_${port}.conf"
    done

    # ====================== Systemd Services ======================
    for i in {0..5}; do
      port=$((7000 + i))

      cat > /etc/systemd/system/redis-${port}.service <<EOL
[Unit]
Description=Redis Cluster Node ${port}
After=network.target

[Service]
Type=notify
User=redis
Group=redis
ExecStart=/usr/bin/redis-server /etc/redis/cluster/${port}/redis_${port}.conf --supervised systemd
ExecStop=/usr/bin/redis-cli -h 127.0.0.1 -p ${port} shutdown
Restart=always
RestartSec=5
LimitNOFILE=65535
RuntimeDirectory=redis

[Install]
WantedBy=multi-user.target
EOL
    done

    # Reload systemd
    systemctl daemon-reload

    # Enable and start all 6 nodes
    for i in {0..5}; do
      port=$((7000 + i))
      systemctl enable --now redis-${port}
    done

    echo "✅ All Redis nodes started."

    # ====================== Initialize Cluster ======================
    echo "⏳ Initializing Redis Cluster (this may take a few seconds)..."

    redis-cli --cluster create \
      127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 \
      127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 \
      --cluster-replicas 1 --cluster-yes

    echo "🎉 Redis Cluster Setup Completed!"
    echo ""
    echo "Cluster Nodes:"
    redis-cli -p 7000 cluster nodes | head -n 10
    echo ""
    echo "Access from host:"
    echo "redis-cli -p 7000 ping"
  SHELL
end
```

---

### How to Deploy

```bash
vagrant up
```

Or re-provision if VM already exists:

```bash
vagrant reload --provision
```

---

### Why This Is the Best DevOps Approach

- Fully automated with one `vagrant up`
- Clean separation of concerns (configs, services)
- Each node runs as independent **systemd unit** (`redis-7000.service`, etc.)
- Easy to manage: `systemctl status redis-7000`, `journalctl -u redis-7000 -f`
- Scalable and version-controlled
- Good resource allocation (4GB RAM, 4 CPUs)
- Ports forwarded for easy testing from your laptop

---


## What does "**The nodes must be started and managed using custom systemd units (services)**" mean?

It means that **each Redis node** (whether in Cluster or Sentinel mode) must run as a **separate, independent systemd service** on the Linux system — not just started manually with `redis-server` command or all together under one service.

### In Simple Terms:

Instead of doing this (not desired):
```bash
redis-server /path/to/config1.conf     # manual, not managed
redis-server /path/to/config2.conf     # manual
```

You must do this (required):

- Create **individual systemd unit files** like:
  - `redis-7000.service`
  - `redis-7001.service`
  - `redis-7002.service`
  - etc.

- Then manage them using standard systemctl commands:
  ```bash
  sudo systemctl start redis-7000
  sudo systemctl stop redis-7001
  sudo systemctl restart redis-7000
  sudo systemctl status redis-7002
  sudo systemctl enable redis-7000     # start automatically after reboot
  ```

### Why This Requirement?

This is considered a **best practice** in DevOps / production environments because:

| Benefit                        | Description |
|--------------------------------|-----------|
| **Proper Service Management**  | Systemd handles starting, stopping, restarting, and monitoring |
| **Automatic Restart**          | If a node crashes, systemd restarts it automatically |
| **Logging**                    | Logs go to journalctl (`journalctl -u redis-7000 -f`) |
| **Resource Limits**            | Easy to set CPU, memory, file descriptors limits |
| **Dependency Management**      | Can define startup order |
| **Standard Linux Way**         | Matches how real services (nginx, mysql, etc.) are managed |
| **Observability**              | Easy to monitor individual nodes |

### What a Custom Systemd Unit Looks Like

```ini
[Unit]
Description=Redis Cluster Node on port 7000
After=network.target

[Service]
Type=notify
User=redis
Group=redis
ExecStart=/usr/bin/redis-server /etc/redis/cluster/7000/redis_7000.conf --supervised systemd
ExecStop=/usr/bin/redis-cli -h 127.0.0.1 -p 7000 shutdown
Restart=always
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```

This is what was implemented in the previous Vagrantfiles — each node gets its own `.service` file.

## Testing it works fine

**✅ How to Test Your Redis Cluster Setup (on the Vagrant VM)**

Here’s a complete testing checklist — from basic to advanced — to verify that your Redis Cluster is working correctly.

### 1. Connect to the VM and Run Basic Checks

```bash
vagrant ssh
```

### 2. Check Systemd Services (Most Important)

```bash
# Check all 6 services
sudo systemctl status redis-70* --no-pager

# Or check one by one
sudo systemctl status redis-7000 redis-7001 redis-7002
```

**Expected**: All services should be `active (running)`.

### 3. Basic Connectivity Test

```bash
# Test each node
for port in {7000..7005}; do
  echo -n "Port $port: "
  redis-cli -p $port ping
done
```

**Expected**: All should return `PONG`.

### 4. Cluster Health Verification (Key Commands)

Run these commands:

```bash
# 1. Cluster nodes (topology)
redis-cli -p 7000 cluster nodes

# 2. Cluster state summary
redis-cli -p 7000 cluster info

# 3. Best overall health check
redis-cli --cluster check 127.0.0.1:7000
```

**What to look for**:
- `cluster_state:ok`
- `cluster_slots_assigned:16384` and `cluster_slots_ok:16384`
- 3 masters + 3 replicas
- No `fail` or `pfail` flags on nodes

### 5. Data Write & Read Test (with Cluster Mode)

```bash
# Set keys that will go to different nodes
redis-cli -p 7000 -c SET name "Yuriy"
redis-cli -p 7000 -c SET city "Toronto"
redis-cli -p 7000 -c SET counter 100

# Read them back
redis-cli -p 7000 -c GET name
redis-cli -p 7000 -c GET city
redis-cli -p 7000 -c INCR counter
```

The `-c` flag enables **cluster mode** (follows redirections automatically).

### 6. Advanced Validation Commands

```bash
# Check slot distribution
redis-cli -p 7000 cluster slots | head -n 20

# Info from all nodes at once
redis-cli --cluster call 127.0.0.1:7000 info | grep -E "cluster|role|connected"

# Memory and performance stats
redis-cli -p 7000 info memory
redis-cli -p 7000 info stats
```

### 7. Test Failover (Optional but Recommended)

Simulate a master failure:

```bash
# Find a master node
redis-cli -p 7000 cluster nodes | grep master

# Stop one master (example: 7000)
sudo systemctl stop redis-7000

# Wait 10-15 seconds and check cluster again
redis-cli -p 7001 cluster nodes

# Restart it
sudo systemctl start redis-7000
```

A replica should be promoted to master automatically.

### 8. Quick One-Liner Health Check Script

You can create this helper script:

```bash
cat > ~/check-redis-cluster.sh <<'EOF'
#!/bin/bash
echo "=== Redis Cluster Health Check ==="
redis-cli -p 7000 cluster info | grep -E "cluster_state|cluster_slots|cluster_known_nodes"
echo "Nodes:"
redis-cli -p 7000 cluster nodes | wc -l
echo "Check:"
redis-cli --cluster check 127.0.0.1:7000 | tail -n 5
EOF

chmod +x ~/check-redis-cluster.sh
~/check-redis-cluster.sh
```

---

**All Good If You See**:
- All 6 services running
- `PONG` from all nodes
- `cluster_state:ok`
- 16384 slots assigned
- 3 masters + 3 replicas
- Able to read/write with `-c` flag
