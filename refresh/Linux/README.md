To help you master these systems at a level appropriate for a Senior Engineer, we will move away from default configurations. This guide focuses on **Hardening, Performance Tuning, and Production-Ready Automation**.

---

## **Phase 1: Hardening & Networking (The Foundation)**

The goal is a "Zero Trust" host where services are invisible to the public internet.

### **1.1. Host Hardening**
1.  **Update & Clean:** `sudo apt update && sudo apt upgrade -y`
2.  **SSH Security:**
    *   Edit `/etc/ssh/sshd_config`:
        *   `PasswordAuthentication no`
        *   `PermitRootLogin no`
        *   `PubkeyAuthentication yes`
    *   Reload: `sudo systemctl restart ssh`
3.  **Firewall (UFW):**
    *   `sudo ufw default deny incoming`
    *   `sudo ufw allow 51820/udp` (Wireguard)
    *   `sudo ufw allow [Your-Home-IP] to any port 22` (Emergency SSH)
    *   `sudo ufw enable`

### **1.2. Wireguard Setup**
1.  **Install:** `sudo apt install wireguard -y`
2.  **Generate Keys:**
    ```bash
    wg genkey | tee privatekey | wg pubkey > publickey
    ```
3.  **Configure Interface (`/etc/wireguard/wg0.conf`):**
    ```ini
    [Interface]
    PrivateKey = <Server-Private-Key>
    Address = 10.0.0.1/24
    ListenPort = 51820

    [Peer]
    PublicKey = <Your-Laptop-Public-Key>
    AllowedIPs = 10.0.0.2/32
    ```
4.  **Enable:** `sudo wg-quick up wg0` and `sudo systemctl enable wg-quick@wg0`

---

## **Phase 2: Data Persistence (PostgreSQL & MySQL)**

### **2.1. PostgreSQL (Systemd Installation)**
1.  **Tune for Performance:** Use `pgtune` logic. Edit `/etc/postgresql/16/main/postgresql.conf`:
    *   `max_connections = 100`
    *   `shared_buffers = [25% of RAM]`
    *   `effective_cache_size = [75% of RAM]`
2.  **Network Isolation:** In `pg_hba.conf`, restrict connections to the Wireguard IP range:
    ```text
    host    all             all             10.0.0.0/24            scram-sha-256
    ```

### **2.2. MySQL (Docker Installation)**
1.  **Deploy with Config Mount:**
    ```bash
    docker run -d \
      --name mysql-prod \
      -v /opt/mysql/data:/var/lib/mysql \
      -v /opt/mysql/my.cnf:/etc/mysql/my.cnf \
      -e MYSQL_ROOT_PASSWORD=strong_pass \
      mysql:8.0
    ```
2.  **Automation Script (Bash):** Create `db_backup.sh`:
    ```bash
    #!/bin/bash
    TIMESTAMP=$(date +%F_%H-%M-%S)
    BACKUP_DIR="/backups/mysql"
    docker exec mysql-prod /usr/bin/mysqldump -u root -p'pass' --all-databases | gzip > "$BACKUP_DIR/db_$TIMESTAMP.sql.gz"
    find $BACKUP_DIR -mtime +7 -delete # Rotation
    ```

---

## **Phase 3: Messaging & Caching (Redis & RabbitMQ)**

### **3.1. Redis (High Performance)**
1.  **Memory Management:** In `/etc/redis/redis.conf`:
    *   `maxmemory 512mb`
    *   `maxmemory-policy allkeys-lru` (Evict least recently used when full)
2.  **Security:** 
    *   `bind 10.0.0.1` (Only listen on Wireguard)
    *   `requirepass your_secure_password`
    *   `rename-command FLUSHALL ""` (Disable dangerous commands)

### **3.2. RabbitMQ (Reliability)**
1.  **Installation:** Use the official Erlang/RabbitMQ repos for the latest version.
2.  **Resource Limits:** Create `/etc/rabbitmq/rabbitmq.conf`:
    ```ini
    vm_memory_high_watermark.relative = 0.4
    disk_free_limit.absolute = 2GB
    ```
3.  **Monitoring Bash Script:** Use `rabbitmqctl` to check queue health:
    ```bash
    #!/bin/bash
    QUEUES=$(rabbitmqctl list_queues name messages_ready | grep -v "Timeout" | awk '{if($2>1000) print $1}')
    if [ ! -z "$QUEUES" ]; then
      echo "Alert: Queues $QUEUES are backing up!" | mail -s "RabbitMQ Alert" yuriy@example.com
    fi
    ```

---

## **Phase 4: Traffic Routing (Nginx & SSL)**

### **4.1. The Reverse Proxy**
1.  **Main Config (`/etc/nginx/nginx.conf`):**
    *   `worker_processes auto;`
    *   `worker_connections 2048;`
    *   `keepalive_timeout 65;`
2.  **Security Headers:** Add to your site config:
    ```nginx
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
    add_header Strict-Transport-Security "max-age=31536000";
    ```
3.  **SSL:** Use `certbot --nginx -d yourdomain.com`. Verify the cron job in `/etc/cron.d/certbot` for auto-renewal.

---

## **Phase 5: Containerization & Orchestration (Docker)**



### **5.1. Multi-Stage Build (Go Example)**
Create a `Dockerfile` that minimizes attack surface by using `scratch` or `alpine`:
```dockerfile
# Stage 1: Build
FROM golang:1.22-alpine AS builder
WORKDIR /app
COPY . .
RUN go build -o main .

# Stage 2: Final
FROM alpine:latest
COPY --from=builder /app/main /main
CMD ["./main"]
```

### **5.2. Docker Compose (The Stack)**
Create `docker-compose.yml` to link services:
```yaml
services:
  app:
    build: .
    networks:
      - backend
    depends_on:
      - redis
      - db
  
  redis:
    image: redis:alpine
    networks:
      - backend

  db:
    image: postgres:16
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - backend

networks:
  backend:
    driver: bridge

volumes:
  pgdata:
```

---

## **The Automation Bridge: Systemd + Bash**

For any service running on the host (not in Docker), create a custom **Systemd Unit** to ensure it starts after the network is up.

**Example: `/etc/systemd/system/my-automation.service`**
```ini
[Unit]
Description=Custom Health Check Script
After=network.target

[Service]
ExecStart=/usr/local/bin/health-check.sh
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

**Next Step:** Run `systemctl daemon-reload` and `systemctl enable --now my-automation`.
```
