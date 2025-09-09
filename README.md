# DevOps Cheat Sheet

## 1. System Administration & Scripting

- Linux commands
- Shell scripting
- Python

### Linux Commands

#### 1. File Management

- `ls` - List directory contents.
- `cd` - Change directory.
- `pwd` - Print working directory.
- `cp` - Copy files or directories.
- `mv` - Move or rename files or directories.
- `rm` - Remove files or directories.
- `touch` - Create a new empty file.
- `mkdir` - Create a new directory.
- `rmdir` - Remove an empty directory.
- `cat` - Concatenate and display file contents.
- `head` - Display the first few lines of a file.
- `tail` - Display the last few lines of a file.
- `chmod` - Change file permissions.
- `chown` - Change file ownership.
- `find` - Search for files in a directory hierarchy.
- `locate` - Find files by name.
- `grep` - Search text using patterns.
- `diff` - Compare two files line by line.
- `tar` - Archive files.
- `zip/unzip` - Compress and extract files.
- `scp` - Securely copy files over SSH.

##### Examples

- `ls`: List files and directories.
  - `ls -l` # Long listing with details

- `cd`: Change directory.
  - `cd /home/swapna` # Move to /home/swapna directory

- `pwd`: Show current directory.
  - `pwd`

- `cp`: Copy files.
  - `cp file1.txt /tmp` # Copy file1.txt to /tmp directory

- `mv`: Move or rename files.
  - `mv oldname.txt newname.txt` # Rename file
  - `mv file1.txt /tmp` # Move to /tmp directory

- `rm`: Remove files or directories.
  - `rm file1.txt` # Remove a file
  - `rm -rf /tmp/old_directory` # Remove directory and contents

- `mkdir`: Create directories.
  - `mkdir new_folder` # Create a directory called new_folder

- `cat`: Display file contents.
  - `cat file.txt`

#### 2. System Information and Monitoring

- `top` - Display running processes and system usage.
- `htop` - Interactive process viewer.
- `ps` - Display current processes.
- `df` - Show disk space usage.
- `du` - Show directory space usage.
- `free` - Show memory usage.
- `uptime` - Show system uptime.
- `uname` - Show system information.
- `whoami` - Display the current logged-in user.
- `lsof` - List open files and associated processes.
- `vmstat` - Report virtual memory statistics.
- `iostat` - Report I/O statistics.
- `netstat` - Display network connections and routing tables.
- `ifconfig` - Display or configure a network interface.
- `ping` - Check network connectivity.
- `traceroute` - Track the route packets take to a destination.

##### Examples

- `top`: View running processes.
  - `top`

- `df`: Show disk usage.
  - `df -h` # Human-readable format

- `free`: Display memory usage.
  - `free -m` # Show memory in MB

- `uptime`: Check system uptime.
  - `uptime`

#### 3. Package Management (Ubuntu/Debian)

- `apt-get update` - Update package lists.
- `apt-get upgrade` - Upgrade all packages.
- `apt-get install` - Install packages.
- `apt-get remove` - Remove packages.
- `dpkg` - Install, remove, and manage individual Debian packages.
- `apt-get` - Install, remove, or update packages.

##### Examples

- `sudo apt-get update` # Update package lists
- `sudo apt-get install nginx` # Install NGINX

#### 4. User and Permission Management

- `useradd` - Add a new user.
- `userdel` - Delete a user.
- `usermod` - Modify a user.
- `passwd` - Change user password.
- `groupadd` - Create a new group.
- `groupdel` - Delete a group.
- `groups` - Show groups of a user.
- `su` - Switch user.
- `sudo` - Execute a command as another user, usually root.

##### Examples

- `useradd`: Add a new user.
  - `sudo useradd -m newuser` # Create a new user with a home directory

- `chmod`: Change file permissions.
  - `chmod 755 script.sh` # Set permissions for owner and others

- `chown`: Change file owner.
  - `sudo chown newuser file.txt` # Change ownership to newuser

#### 5. Networking

- `curl` - Transfer data from or to a server.
- `wget` - Download files from the internet.
- `ssh` - Secure shell to a remote server.
- `telnet` - Connect to a remote machine.
- `nslookup` - Query DNS records.
- `dig` - DNS lookup utility.
- `iptables` - Configure firewall rules.
- `firewalld` - Firewall management (CentOS/RHEL).
- `hostname` - Show or set the system hostname.
- `ping` - Check connectivity to a host.

##### Examples

- `ping google.com`
- `curl https://example.com`
- `ifconfig`

#### 6. Process Management

- `kill` - Send a signal to a process.
- `killall` - Kill processes by name.
- `pkill` - Kill processes by pattern matching.
- `bg` - Move a job to the background.
- `fg` - Bring a job to the foreground.
- `jobs` - List background jobs.
- `ps` - Show running processes.

##### Examples

- `ps aux | grep nginx` # List processes related to nginx
- `kill 1234` # Kill process with PID 1234
- `pkill nginx` # Kill all nginx processes

#### 7. Disk Management

- `fdisk` - Partition a disk.
- `mkfs` - Make a filesystem.
- `mount` - Mount a filesystem.
- `umount` - Unmount a filesystem.
- `lsblk` - List block devices.
- `blkid` - Print block device attributes.
- `fdisk` - Manage disk partitions.

##### Examples

- `sudo fdisk -l` # List disk partitions
- `sudo mount /dev/sdb1 /mnt` # Mount device sdb1 to /mnt

#### 8. Text Processing

- `awk` - Pattern scanning and processing.
- `sed` - Stream editor for modifying text.
- `sort` - Sort lines of text files.
- `uniq` - Report or omit repeated lines.
- `cut` - Remove sections from each line of files.
- `wc` - Word, line, character count.
- `tr` - Translate or delete characters.
- `nl` - Number lines of files.
- `grep` - Search text.

##### Examples

- `grep "error" /var/log/syslog` # Search for "error" in syslog
- `awk '{print $1}' file.txt` # Print the first column of each line
- `sed 's/old/new/g' file.txt` # Replace "old" with "new"

#### 9. Logging and Auditing

- `dmesg` - Print or control kernel ring buffer.
- `journalctl` - Query the systemd journal.
- `logger` - Add entries to the system log.
- `last` - Show listing of last logged-in users.
- `history` - Show command history.
- `tail -f` - Monitor logs in real time.

##### Examples

- `tail -f /var/log/nginx/access.log` # Follow NGINX access log
- `sudo journalctl -u nginx` # Logs for NGINX service

#### 10. Archiving and Backup

- `tar` - Archive files.
- `rsync` - Synchronize files and directories.

##### Examples

- `tar -cvf archive.tar /path/to/files` # Create an archive
- `tar -xvf archive.tar` # Extract an archive
- `rsync -avz /source /destination` # Sync with compression and archive mode

#### 11. Shell Scripting

- `echo` - Display message or text.
- `read` - Read input from the user.
- `export` - Set environment variables.
- `alias` - Create shortcuts for commands.
- `sh` - Execute shell scripts.

##### Examples

- `echo "Hello, DevOps!"` # Print message
- `export PATH=$PATH:/new/path` # Add to PATH variable

#### 12. System Configuration and Management

- `crontab` - Schedule periodic tasks.
- `systemctl` - Control the systemd system and service manager.
- `service` - Start, stop, or restart services.
- `timedatectl` - Query and change the system clock.
- `reboot` - Restart the system.
- `shutdown` - Power off the system.

##### Examples

- `crontab -e` # Edit the crontab file
  - Example entry: `0 2 * * * /path/to/backup.sh`
- `sudo systemctl restart nginx` # Restart NGINX

#### 13. Containerization & Virtualization

- `docker` - Manage Docker containers.
- `kubectl` - Manage Kubernetes clusters.

##### Examples

- `docker ps` # List running containers
- `docker run -d -p 8080:80 nginx` # Run NGINX container
- `kubectl get pods` # List all pods
- `kubectl apply -f deployment.yaml` # Deploy configuration

#### 14. Git Version Control

- `git status` - Show the status of changes.
- `git add` - Add files to staging.
- `git commit` - Commit changes.
- `git push` - Push changes to a remote repository.
- `git pull` - Pull changes from a remote repository.
- `git clone` - Clone a repository.

##### Examples

- `git commit -m "<message>"` - Commit changes with a descriptive message.
- `git push <remote> <branch>` - Push changes to a remote repository.
- `git pull <remote> <branch>` - Pull changes from a remote repository.
- `git clone <repository>` - Clone a repository.
- `git remote -v` - Show URLs of remote repositories.
- `git remote add <name> <url>` - Add a new remote repository.
- `git remote remove <name>` - Remove a remote repository by name.
- `git remote rename <old-name> <new-name>` - Rename a remote repository.

#### 15. Others

- `env` - Display environment variables.
- `date` - Show or set the system date and time.
- `alias` - Create command shortcuts.
- `source` - Execute commands from a file in the current shell.
- `sleep` - Pause for a specified amount of time.

#### 16. Network Troubleshooting and Analysis

- `traceroute`: Track packet route.
  - `traceroute google.com` # Show hops to google.com

- `netstat`: View network connections, routing tables, and more.
  - `netstat -tuln` # Show active listening ports with protocol info

- `ss`: Display socket statistics (modern alternative to netstat).
  - `ss -tuln` # Show active listening ports

- `iptables`: Manage firewall rules.
  - `sudo iptables -L` # List current iptables rules

#### 17. Advanced File Management

- `find`: Search files by various criteria.
  - `find /var -name "*.log"` # Find all .log files under /var
  - `find /home -type d -name "test"` # Find directories named "test"

- `locate`: Quickly find files by name.
  - `locate apache2.conf` # Locate apache2 configuration file

#### 18. File Content and Manipulation

- `split`: Split files into parts.
  - `split -l 500 largefile.txt smallfile` # Split file into 500-line chunks

- `sort`: Sort lines in files.
  - `sort file.txt` # Sort lines alphabetically
  - `sort -n numbers.txt` # Sort numerically

- `uniq`: Remove duplicates from sorted files.
  - `sort file.txt | uniq` # Remove duplicate lines

#### 19. Advanced Shell Operations

- `xargs`: Build and execute commands from standard input.
  - `find . -name "*.txt" | xargs rm` # Delete all .txt files

- `tee`: Read from standard input and write to standard output and files.
  - `echo "new data" | tee file.txt` # Write output to file and terminal

#### 20. Performance Analysis

- `iostat`: Display CPU and I/O statistics.
  - `iostat -d 2` # Show disk I/O stats every 2 seconds

- `vmstat`: Report virtual memory stats.
  - `vmstat 1 5` # Display 5 samples at 1-second intervals

- `sar`: Collect and report system activity information.
  - `sar -u 5 5` # Report CPU usage every 5 seconds

#### 21. Disk and File System Analysis

- `lsblk`: List block devices.
  - `lsblk -f` # Show filesystems and partitions

- `blkid`: Display block device attributes.
  - `sudo blkid` # Show UUIDs for devices

- `ncdu`: Disk usage analyzer with a TUI.
  - `ncdu /` # Analyze root directory space usage

#### 22. File Compression and Decompression

- `gzip`: Compress files.
  - `gzip largefile.txt` # Compress file with .gz extension

- `gunzip`: Decompress .gz files.
  - `gunzip largefile.txt.gz` # Decompress file

- `bzip2`: Compress files with higher compression than gzip.
  - `bzip2 largefile.txt` # Compress file with .bz2 extension

#### 23. Environment Variables and Shell Management

- `env`: Display all environment variables.
  - `env`

- `set`: Set or display shell options and variables.
  - `set | grep PATH` # Show the PATH variable

- `unset`: Remove an environment variable.
  - `unset VAR_NAME` # Remove a specific environment variable

#### 24. Networking Utilities

- `arp`: Show or modify the IP-to-MAC address mappings.
  - `arp -a` # Display all IP-MAC mappings

- `nc (netcat)`: Network tool for debugging and investigation.
  - `nc -zv example.com 80` # Test if a specific port is open

- `nmap`: Network scanner to discover hosts and services.
  - `nmap -sP 192.168.1.0/24` # Scan all hosts on a subnet

#### 25. System Security and Permissions

- `umask`: Set default permissions for new files.
  - `umask 022` # Set default permissions to 755 for new files

- `chmod`: Change file or directory permissions.
  - `chmod 700 file.txt` # Owner only read, write, execute

- `chattr`: Change file attributes.
  - `sudo chattr +i file.txt` # Make file immutable

- `lsattr`: List file attributes.
  - `lsattr file.txt` # Show attributes for a file

#### 26. Container and Kubernetes Management

- `docker-compose`: Manage multi-container Docker applications.
  - `docker-compose up -d` # Start containers in detached mode

- `minikube`: Run a local Kubernetes cluster.
  - `minikube start` # Start minikube cluster

- `helm`: Kubernetes package manager.
  - `helm install myapp ./myapp-chart` # Install Helm chart for an app

#### 27. Advanced Git Operations

- `git stash`: Temporarily save changes.
  - `git stash` # Stash current changes

- `git rebase`: Reapply commits on top of another base commit.
  - `git rebase main` # Rebase current branch onto main

- `git log`: View commit history.
  - `git log --oneline --graph` # Compact log with graph view

#### 28. Troubleshooting and Debugging

- `strace`: Trace system calls and signals.
  - `strace -p 1234` # Trace process with PID 1234

- `lsof`: List open files by processes.
  - `lsof -i :8080` # List processes using port 8080

- `dmesg`: Print kernel ring buffer messages.
  - `dmesg | tail -10` # View last 10 kernel messages

#### 29. Data Manipulation and Processing

- `paste`: Merge lines of files.
  - `paste file1.txt file2.txt` # Combine lines from two files

- `join`: Join lines of two files on a common field.
  - `join file1.txt file2.txt` # Join files on matching lines

- `column`: Format text output into columns.
  - `cat data.txt | column -t` # Display data in columns

#### 30. File Transfer

- `rsync`: Sync files between local and remote systems.
  - `rsync -avz /local/dir user@remote:/remote/dir`

- `scp`: Securely copy files between hosts.
  - `scp file.txt user@remote:/path/to/destination` # Copy to remote

- `ftp`: Transfer files using FTP protocol.
  - `ftp example.com` # Connect to FTP server example.com

#### 31. Job Management and Scheduling

- `bg`: Send a job to the background.
  - `./script.sh &` # Run a script in the background

- `fg`: Bring a background job to the foreground.
  - `fg %1` # Bring job 1 to the foreground

- `at`: Schedule a command to run once at a specified time.
  - `echo "echo Hello, DevOps" | at now + 2 minutes` # Run in 2 minutes

### Shell Scripting

#### 1. Automating Server Provisioning (AWS EC2 Launch)

```bash
#!/bin/bash

# Variables
INSTANCE_TYPE="t2.micro"
AMI_ID="ami-0abcdef1234567890" # Replace with the correct AMI ID
KEY_NAME="my-key-pair" # Replace with your key pair name
SECURITY_GROUP="sg-0abc1234def567890" # Replace with your security group ID
SUBNET_ID="subnet-0abc1234def567890" # Replace with your subnet ID
REGION="us-west-2" # Replace with your AWS region

# Launch EC2 instance
aws ec2 run-instances --image-id $AMI_ID --count 1 --instance-type $INSTANCE_TYPE \
--key-name $KEY_NAME --security-group-ids $SECURITY_GROUP --subnet-id $SUBNET_ID --region $REGION
echo "EC2 instance launched successfully!"
```

#### 2. System Monitoring (CPU Usage Alert)

```bash
#!/bin/bash

# Threshold for CPU usage
CPU_THRESHOLD=80

# Get the current CPU usage
CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | sed "s/.*, *\\([0-9.]*\\)%* id.*/\\1/" | awk '{print 100 - $1}')

# Check if CPU usage exceeds threshold
if (( $(echo "$CPU_USAGE > $CPU_THRESHOLD" | bc -l) )); then
echo "Alert: CPU usage is above $CPU_THRESHOLD%. Current usage is $CPU_USAGE%" | mail -s "CPU Usage Alert" user@example.com
fi
```

#### 3. Backup Automation (MySQL Backup)

```bash
#!/bin/bash

# Variables
DB_USER="root"
DB_PASSWORD="password"
DB_NAME="my_database"
BACKUP_DIR="/backup"
DATE=$(date +%F)

# Create backup directory if it doesn't exist
mkdir -p $BACKUP_DIR

# Backup command
mysqldump -u $DB_USER -p$DB_PASSWORD $DB_NAME > $BACKUP_DIR/backup_$DATE.sql

# Optional: Compress the backup
gzip $BACKUP_DIR/backup_$DATE.sql

echo "Backup completed successfully!"
```

#### 4. Log Rotation and Cleanup

```bash
#!/bin/bash

# Variables
LOG_DIR="/var/log/myapp"
ARCHIVE_DIR="/var/log/myapp/archive"
DAYS_TO_KEEP=30

# Create archive directory if it doesn't exist
mkdir -p $ARCHIVE_DIR

# Find and compress logs older than 7 days
find $LOG_DIR -type f -name "*.log" -mtime +7 -exec gzip {} \; -exec mv {} $ARCHIVE_DIR \;

# Delete logs older than 30 days
find $ARCHIVE_DIR -type f -name "*.log.gz" -mtime +$DAYS_TO_KEEP -exec rm {} \;

echo "Log rotation and cleanup completed!"
```

#### 5. CI/CD Pipeline Automation (Trigger Jenkins Job)

```bash
#!/bin/bash

# Jenkins details
JENKINS_URL="http://jenkins.example.com"
JOB_NAME="my-pipeline-job"
USER="your-username"
API_TOKEN="your-api-token"

# Trigger Jenkins job
curl -X POST "$JENKINS_URL/job/$JOB_NAME/build" --user "$USER:$API_TOKEN"

echo "Jenkins job triggered successfully!"
```

#### 6. Deployment Automation (Kubernetes Deployment)

```bash
#!/bin/bash

# Variables
NAMESPACE="default"
DEPLOYMENT_NAME="my-app"
IMAGE="my-app:v1.0"

# Deploy to Kubernetes
kubectl set image deployment/$DEPLOYMENT_NAME $DEPLOYMENT_NAME=$IMAGE --namespace=$NAMESPACE

echo "Deployment updated to version $IMAGE!"
```

#### 7. Infrastructure as Code (Terraform Apply)

```bash
#!/bin/bash

# Variables
TF_DIR="/path/to/terraform/config"

# Navigate to Terraform directory
cd $TF_DIR

# Run terraform apply
terraform apply -auto-approve
echo "Terraform apply completed successfully!"
```

#### 8. Database Management (PostgreSQL Schema Migration)

```bash
#!/bin/bash

# Variables
DB_USER="postgres"
DB_PASSWORD="password"
DB_NAME="my_database"
MIGRATION_FILE="/path/to/migration.sql"

# Run schema migration
PGPASSWORD=$DB_PASSWORD psql -U $DB_USER -d $DB_NAME -f $MIGRATION_FILE

echo "Database schema migration completed!"
```

#### 9. User Management (Add User to Group)

```bash
#!/bin/bash

# Variables
USER_NAME="newuser"
GROUP_NAME="devops"

# Add user to group
usermod -aG $GROUP_NAME $USER_NAME

echo "User $USER_NAME added to group $GROUP_NAME!"
```

#### 10. Security Audits (Check for Open Ports)

```bash
#!/bin/bash

# Check for open ports
OPEN_PORTS=$(netstat -tuln)

# Check if any ports are open (excluding localhost)
if [[ $OPEN_PORTS =~ "0.0.0.0" || $OPEN_PORTS =~ "127.0.0.1" ]]; then
  echo "Security Alert: Open ports detected!"
  echo "$OPEN_PORTS" | mail -s "Open Ports Security Alert" user@example.com
else
  echo "No open ports detected."
fi
```

#### 11. Performance Tuning

This script clears memory caches and restarts services to free up system resources.

```bash
#!/bin/bash

# Clear memory caches to free up resources
sync; echo 3 > /proc/sys/vm/drop_caches

# Restart services to free up resources
systemctl restart nginx
systemctl restart apache2
```

#### 12. Automated Testing

This script runs automated tests using a testing framework like pytest for Python or JUnit for Java.

```bash
#!/bin/bash

# Run unit tests using pytest (Python example)
pytest tests/

# Or, run JUnit tests (Java example)
mvn test
```

#### 13. Scaling Infrastructure

This script automatically scales EC2 instances in an Auto Scaling group based on CPU usage.

```bash
#!/bin/bash

# Check CPU usage and scale EC2 instances
CPU_USAGE=$(aws cloudwatch get-metric-statistics --namespace AWS/EC2 --metric-name CPUUtilization --dimensions Name=InstanceId,Value=i-1234567890abcdef0 --statistics Average --period 300 --start-time $(date -d '5 minutes ago' --utc +%FT%TZ) --end-time $(date --utc +%FT%TZ) --query 'Datapoints[0].Average' --output text)

if (( $(echo "$CPU_USAGE > 80" | bc -l) )); then
  aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-auto-scaling-group --desired-capacity 3
fi
```

#### 14. Environment Setup

This script sets environment variables for different environments (development, staging, production).

```bash
#!/bin/bash

# Set environment variables for different stages
if [ "$1" == "production" ]; then
  export DB_HOST="prod-db.example.com"
  export API_KEY="prod-api-key"
elif [ "$1" == "staging" ]; then
  export DB_HOST="staging-db.example.com"
  export API_KEY="staging-api-key"
else
  export DB_HOST="dev-db.example.com"
  export API_KEY="dev-api-key"
fi
```

#### 15. Error Handling and Alerts

This script checks logs for errors and sends a Slack notification if an error is found.

```bash
#!/bin/bash

# Check logs for error messages and send Slack notification
if grep -i "error" /var/log/myapp.log; then
  curl -X POST -H 'Content-type: application/json' --data '{"text":"Error found in logs!"}' https://hooks.slack.com/services/your/webhook/url
fi
```

#### 16. Automated Software Installation and Updates

This script installs Docker if it's not already installed on the system.

```bash
#!/bin/bash

# Install Docker
if ! command -v docker &> /dev/null; then
  curl -fsSL https://get.docker.com -o get-docker.sh
  sudo sh get-docker.sh
fi
```

#### 17. Configuration Management

This script updates configuration files (like nginx.conf) across multiple servers.

```bash
#!/bin/bash

# Update nginx configuration across all servers
scp nginx.conf user@server:/etc/nginx/nginx.conf
ssh user@server "systemctl restart nginx"
```

#### 18. Health Check Automation

This script checks the health of multiple web servers by making HTTP requests.

```bash
#!/bin/bash

# Check if web servers are running
for server in "server1" "server2" "server3"; do
  curl -s --head http://$server | head -n 1 | grep "HTTP/1.1 200 OK" > /dev/null
  if [ $? -ne 0 ]; then
    echo "$server is down"
  else
    echo "$server is up"
  fi
done
```

#### 19. Automated Cleanup of Temporary Files

This script removes files older than 30 days from the /tmp directory to free up disk space.

```bash
#!/bin/bash

# Remove files older than 30 days in /tmp
find /tmp -type f -mtime +30 -exec rm -f {} \;
```

#### 20. Environment Variable Management

This script sets environment variables from a .env file.

```bash
#!/bin/bash

# Set environment variables from a .env file
export $(grep -v '^#' .env | xargs)
```

#### 21. Server Reboot Automation

This script automatically reboots the server during off-hours (between 2 AM and 4 AM).

```bash
#!/bin/bash

# Reboot server during off-hours
if [ $(date +%H) -ge 2 ] && [ $(date +%H) -lt 4 ]; then
  sudo reboot
fi
```

#### 22. SSL Certificate Renewal

This script renews SSL certificates using certbot and reloads the web server.

```bash
#!/bin/bash

# Renew SSL certificates using certbot
certbot renew
systemctl reload nginx
```

#### 23. Automatic Scaling of Containers

This script checks the CPU usage of a Docker container and scales it based on usage.

```bash
#!/bin/bash

# Check CPU usage of a Docker container and scale if necessary
CPU_USAGE=$(docker stats --no-stream --format "{{.CPUPerc}}" my-container | sed 's/%//')
if (( $(echo "$CPU_USAGE > 80" | bc -l) )); then
  docker-compose scale my-container=3
fi
```

#### 24. Backup Verification

This script verifies the integrity of backup files and reports any corrupted ones.

```bash
#!/bin/bash

# Verify backup files integrity
for backup in /backups/*.tar.gz; do
  if ! tar -tzf $backup > /dev/null 2>&1; then
    echo "Backup $backup is corrupted"
  else
    echo "Backup $backup is valid"
  fi
done
```

#### 25. Automated Server Cleanup

This script removes unused Docker images, containers, and volumes to save disk space.

```bash
#!/bin/bash

# Remove unused Docker images, containers, and volumes
docker system prune -af
```

#### 26. Version Control Operations

This script pulls the latest changes from a Git repository and creates a release tag.

```bash
#!/bin/bash

# Pull latest changes from Git repository and create a release tag
git pull origin main
git tag -a v$(date +%Y%m%d%H%M%S) -m "Release $(date)"
git push origin --tags
```

#### 27. Application Deployment Rollback

This script reverts to the previous Docker container image if a deployment fails.

```bash
#!/bin/bash

# Rollback to the previous Docker container image if deployment fails
if [ $? -ne 0 ]; then
  docker-compose down
  docker-compose pull my-app:previous
  docker-compose up -d
fi
```

#### 28. Automated Log Collection

This script collects logs from multiple servers and uploads them to an S3 bucket.

```bash
#!/bin/bash

# Collect logs and upload them to an S3 bucket
tar -czf /tmp/logs.tar.gz /var/log/*
aws s3 cp /tmp/logs.tar.gz s3://my-log-bucket/logs/$(date +%Y%m%d%H%M%S).tar.gz
```

#### 29. Security Patch Management

This script checks for available security patches and applies them automatically.

```bash
#!/bin/bash

# Check and apply security patches
sudo apt-get update
sudo apt-get upgrade -y --only-upgrade
```

#### 30. Custom Monitoring Scripts

This script checks if a database service is running and restarts it if necessary.

```bash
#!/bin/bash

# Check if a database service is running and restart it if necessary
if ! systemctl is-active --quiet mysql; then
  systemctl restart mysql
  echo "MySQL service was down and has been restarted"
else
  echo "MySQL service is running"
fi
```

#### 31. DNS Configuration Automation (Route 53)

```bash
#!/bin/bash

# Variables
ZONE_ID="your-hosted-zone-id"
DOMAIN_NAME="your-domain.com"
NEW_IP="your-new-ip-address"

# Update Route 53 DNS record
aws route53 change-resource-record-sets --hosted-zone-id $ZONE_ID --change-batch '{
  "Changes": [
    {
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "'$DOMAIN_NAME'",
        "Type": "A",
        "TTL": 60,
        "ResourceRecords": [
          {
            "Value": "'$NEW_IP'"
          }
        ]
      }
    }
  ]
}'
```

#### 32. Automated Code Linting and Formatting (ESLint and Prettier)

```bash
#!/bin/bash

# Run ESLint
npx eslint . --fix

# Run Prettier
npx prettier --write "**/*.js"
```

#### 33. Automated API Testing (Using curl)

```bash
#!/bin/bash

# API URL
API_URL="https://your-api-endpoint.com/endpoint"

# Make GET request and check for 200 OK response
RESPONSE=$(curl --write-out "%{http_code}" --silent --output /dev/null $API_URL)

if [ $RESPONSE -eq 200 ]; then
  echo "API is up and running"
else
  echo "API is down. Response code: $RESPONSE"
fi
```

#### 34. Container Image Scanning (Using Trivy)

```bash
#!/bin/bash

# Image to scan
IMAGE_NAME="your-docker-image:latest"

# Run Trivy scan
trivy image --exit-code 1 --severity HIGH,CRITICAL $IMAGE_NAME

if [ $? -eq 1 ]; then
  echo "Vulnerabilities found in image: $IMAGE_NAME"
  exit 1
else
  echo "No vulnerabilities found in image: $IMAGE_NAME"
fi
```

#### 35. Disk Usage Monitoring and Alerts (Email Notification)

```bash
#!/bin/bash

# Disk usage threshold
THRESHOLD=80

# Get current disk usage percentage
DISK_USAGE=$(df / | grep / | awk '{ print $5 }' | sed 's/%//g')

# Check if disk usage exceeds threshold
if [ $DISK_USAGE -gt $THRESHOLD ]; then
  echo "Disk usage is above threshold: $DISK_USAGE%" | mail -s "Disk Usage Alert" your-email@example.com
fi
```

#### 36. Automated Load Testing (Using Apache Benchmark)

```bash
#!/bin/bash

# Target URL
URL="https://your-application-url.com"

# Run Apache Benchmark with 1000 requests and 10 concurrent requests
ab -n 1000 -c 10 $URL
```

#### 37. Automated Email Reports (Server Health Report)

```bash
#!/bin/bash

# Server Health Report
REPORT=$(top -n 1 | head -n 10)

# Send report via email
echo "$REPORT" | mail -s "Server Health Report" your-email@example.com
```

#### 38. DNS Configuration Automation (Route 53)

Introduction: This script automates the process of updating DNS records in AWS Route 53 when the IP address of a server changes. It ensures that DNS records are updated dynamically when new servers are provisioned.

```bash
#!/bin/bash

# Variables
ZONE_ID="your-hosted-zone-id"
DOMAIN_NAME="your-domain.com"
NEW_IP="your-new-ip-address"

# Update Route 53 DNS record
aws route53 change-resource-record-sets --hosted-zone-id $ZONE_ID --change-batch '{
  "Changes": [
    {
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "'$DOMAIN_NAME'",
        "Type": "A",
        "TTL": 60,
        "ResourceRecords": [
          {
            "Value": "'$NEW_IP'"
          }
        ]
      }
    }
  ]
}'
```

#### 39. Automated Code Linting and Formatting (ESLint and Prettier)

Introduction: This script runs ESLint and Prettier to check and automatically format JavaScript code before deployment. It ensures code quality and consistency.

```bash
#!/bin/bash

# Run ESLint
npx eslint . --fix

# Run Prettier
npx prettier --write "**/*.js"
```

#### 40. Automated API Testing (Using curl)

Introduction: This script automates the process of testing an API by sending HTTP requests and verifying the response status. It helps ensure that the API is functioning correctly.

```bash
#!/bin/bash

# API URL
API_URL="https://your-api-endpoint.com/endpoint"

# Make GET request and check for 200 OK response
RESPONSE=$(curl --write-out "%{http_code}" --silent --output /dev/null $API_URL)

if [ $RESPONSE -eq 200 ]; then
  echo "API is up and running"
else
  echo "API is down. Response code: $RESPONSE"
fi
```

#### 41. Container Image Scanning (Using Trivy)

Introduction: This script scans Docker images for known vulnerabilities using Trivy. It ensures that only secure images are deployed in production.

```bash
#!/bin/bash

# Image to scan
IMAGE_NAME="your-docker-image:latest"

# Run Trivy scan
trivy image --exit-code 1 --severity HIGH,CRITICAL $IMAGE_NAME

if [ $? -eq 1 ]; then
  echo "Vulnerabilities found in image: $IMAGE_NAME"
  exit 1
else
  echo "No vulnerabilities found in image: $IMAGE_NAME"
fi
```

#### 42. Disk Usage Monitoring and Alerts (Email Notification)

Introduction: This script monitors disk usage and sends an alert via email if the disk usage exceeds a specified threshold. It helps in proactive monitoring of disk space.

```bash
#!/bin/bash

# Disk usage threshold
THRESHOLD=80

# Get current disk usage percentage
DISK_USAGE=$(df / | grep / | awk '{ print $5 }' | sed 's/%//g')

# Check if disk usage exceeds threshold
if [ $DISK_USAGE -gt $THRESHOLD ]; then
  echo "Disk usage is above threshold: $DISK_USAGE%" | mail -s "Disk Usage Alert" your-email@example.com
fi
```

#### 43. Automated Load Testing (Using Apache Benchmark)

Introduction: This script runs load tests using Apache Benchmark (ab) to simulate traffic on an application. It helps measure the performance and scalability of the application.

```bash
#!/bin/bash

# Target URL
URL="https://your-application-url.com"

# Run Apache Benchmark with 1000 requests and 10 concurrent requests
ab -n 1000 -c 10 $URL
```

#### 44. Automated Email Reports (Server Health Report)

Introduction: This script generates a server health report using system commands like top and sends it via email. It helps keep track of server performance and health.

```bash
#!/bin/bash

# Server Health Report
REPORT=$(top -n 1 | head -n 10)

# Send report via email
echo "$REPORT" | mail -s "Server Health Report" your-email@example.com
```

#### 45. Automating Documentation Generation (Using pdoc for Python)

Introduction: This script generates HTML documentation from Python code using pdoc. It helps automate the process of creating up-to-date documentation from the source code.

```bash
#!/bin/bash

# Generate documentation using pdoc
pdoc --html your-python-module --output-dir docs/

# Optionally, you can zip the generated docs
zip -r docs.zip docs/
```

#### Cron Job Syntax

- List all cron jobs: `crontab -l`
- Edit cron jobs: `crontab -e`
- Remove all cron jobs: `crontab -r`
- Use a specific editor (e.g., nano): `EDITOR=nano crontab -e`

##### Cron Job Syntax

```
* * * * * command_to_execute
┬ ┬ ┬ ┬ ┬
│ │ │ │ │
│ │ │ │ └─── Day of the week (0-6, Sunday=0)
│ │ │ └───── Month (1-12 or JAN-DEC)
│ │ └─────── Day of the month (1-31)
│ └───────── Hour (0-23)
└─────────── Minute (0-59)
```

##### Examples

- Run a script every minute: `* * * * * /path/to/script.sh`
- Run a script every 5 minutes: `*/5 * * * * /path/to/script.sh`
- Run a script every 10 minutes: `*/10 * * * * /path/to/script.sh`
- Run a script at midnight: `0 0 * * * /path/to/script.sh`
- Run a script every hour: `0 * * * * /path/to/script.sh`
- Run a script every 2 hours: `0 */2 * * * /path/to/script.sh`
- Run a script every Sunday at 3 AM: `0 3 * * 0 /path/to/script.sh`
- Run a script at 9 AM on the 1st of every month: `0 9 1 * * /path/to/script.sh`
- Run a script every Monday to Friday at 6 PM: `0 18 * * 1-5 /path/to/script.sh`
- Run a script on the first Monday of every month: `0 9 * * 1 [ "$(date +\%d)" -le 7 ] && /path/to/script.sh`
- Run a script on specific dates (e.g., 1st and 15th of the month): `0 12 1,15 * * /path/to/script.sh`
- Run a script between 9 AM and 5 PM, every hour: `0 9-17 * * * /path/to/script.sh`
- Run a script every reboot: `@reboot /path/to/script.sh`
- Run a script daily at midnight: `@daily /path/to/script.sh`
- Run a script weekly at midnight on Sunday: `@weekly /path/to/script.sh`
- Run a script monthly at midnight on the 1st: `@monthly /path/to/script.sh`
- Run a script yearly at midnight on January 1st: `@yearly /path/to/script.sh`
- Redirect cron job output to a log file: `0 0 * * * /path/to/script.sh >> /var/log/script.log 2>&1`
- Run a job only if the previous instance is not running: `0 * * * * flock -n /tmp/job.lock /path/to/script.sh`
- Run a script with a random delay (0-59 minutes): `RANDOM_DELAY=$((RANDOM % 60)) && sleep $RANDOM_DELAY && /path/to/script.sh`
- Run a script with environment variables:
  ```
  SHELL=/bin/bash
  PATH=/usr/local/bin:/usr/bin:/bin
  0 5 * * * /path/to/script.sh
  ```
- Check cron logs (Ubuntu/Debian): `grep CRON /var/log/syslog`
- Check cron logs (Red Hat/CentOS): `grep CRON /var/log/cron`
- Restart cron service (Linux): `sudo systemctl restart cron`
- Check if cron service is running: `sudo systemctl status cron`

### Python

#### Python Basics

- Run a script: `python script.py`
- Start interactive mode: `python`
- Check Python version: `python --version`
- Install a package: `pip install package_name`

Create a virtual environment:
- `python -m venv venv`
- `source venv/bin/activate` # On Linux/macOS
- `venv\Scripts\activate` # On Windows

- Deactivate virtual environment: `deactivate`

#### 1. File Operations

- Read a file:
  ```python
  with open('file.txt', 'r') as file:
    content = file.read()
  print(content)
  ```

- Write to a file:
  ```python
  with open('output.txt', 'w') as file:
    file.write('Hello, DevOps!')
  ```

#### 2. Environment Variables

- Get an environment variable:
  ```python
  import os
  db_user = os.getenv('DB_USER')
  print(db_user)
  ```

- Set an environment variable:
  ```python
  import os
  os.environ['NEW_VAR'] = 'value'
  ```

#### 3. Subprocess Management

- Run shell commands:
  ```python
  import subprocess
  result = subprocess.run(['ls', '-l'], capture_output=True, text=True)
  print(result.stdout)
  ```

#### 4. API Requests

- Make a GET request:
  ```python
  import requests
  response = requests.get('https://api.example.com/data')
  print(response.json())
  ```

#### 5. JSON Handling

- Read JSON from a file:
  ```python
  import json
  with open('data.json', 'r') as file:
    data = json.load(file)
  print(data)
  ```

- Write JSON to a file:
  ```python
  import json
  data = {'name': 'DevOps', 'type': 'Workflow'}
  with open('output.json', 'w') as file:
    json.dump(data, file, indent=4)
  ```

#### 6. Logging

- Basic logging setup:
  ```python
  import logging
  logging.basicConfig(level=logging.INFO)
  logging.info('This is an informational message')
  ```

#### 7. Working with Databases

- Connect to a SQLite database:
  ```python
  import sqlite3
  conn = sqlite3.connect('example.db')
  cursor = conn.cursor()
  cursor.execute('CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, name TEXT)')
  conn.commit()
  conn.close()
  ```

#### 8. Automation with Libraries

- Using Paramiko for SSH connections:
  ```python
  import paramiko
  ssh = paramiko.SSHClient()
  ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
  ssh.connect('hostname', username='user', password='password')
  stdin, stdout, stderr = ssh.exec_command('ls')
  print(stdout.read().decode())
  ssh.close()
  ```

#### 9. Error Handling

- Try-except block:
  ```python
  try:
    # code that may raise an exception
    risky_code()
  except Exception as e:
    print(f'Error occurred: {e}')
  ```

#### 10. Docker Integration

- Using the docker package to interact with Docker:
  ```python
  import docker
  client = docker.from_env()
  containers = client.containers.list()
  for container in containers:
    print(container.name)
  ```

#### 11. Working with YAML Files

- Read a YAML file:
  ```python
  import yaml
  with open('config.yaml', 'r') as file:
    config = yaml.safe_load(file)
  print(config)
  ```

- Write to a YAML file:
  ```python
  import yaml
  data = {'name': 'DevOps', 'version': '1.0'}
  with open('output.yaml', 'w') as file:
    yaml.dump(data, file)
  ```

#### 12. Parsing Command-Line Arguments

- Using argparse:
  ```python
  import argparse
  parser = argparse.ArgumentParser(description='Process some integers.')
  parser.add_argument('--num', type=int, help='an integer for the accumulator')
  args = parser.parse_args()
  print(args.num)
  ```

#### 13. Monitoring System Resources

- Using psutil to monitor system resources:
  ```python
  import psutil
  print(f"CPU Usage: {psutil.cpu_percent()}%")
  print(f"Memory Usage: {psutil.virtual_memory().percent}%")
  ```

#### 14. Handling HTTP Requests with Flask

- Basic Flask API:
  ```python
  from flask import Flask, jsonify
  app = Flask(__name__)

  @app.route('/health', methods=['GET'])
  def health_check():
    return jsonify({'status': 'healthy'})

  if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
  ```

#### 15. Creating Docker Containers

- Using the Docker SDK to create a container:
  ```python
  import docker
  client = docker.from_env()
  container = client.containers.run('ubuntu', 'echo Hello World', detach=True)
  print(container.logs())
  ```

#### 16. Scheduling Tasks

- Using schedule for task scheduling:
  ```python
  import schedule
  import time

  def job():
    print("Running scheduled job...")

  schedule.every(1).minutes.do(job)

  while True:
    schedule.run_pending()
    time.sleep(1)
  ```

#### 17. Version Control with Git

- Using GitPython to interact with Git repositories:
  ```python
  import git
  repo = git.Repo('/path/to/repo')
  repo.git.add('file.txt')
  repo.index.commit('Added file.txt')
  ```

#### 18. Email Notifications

- Sending emails using smtplib:
  ```python
  import smtplib
  from email.mime.text import MIMEText

  msg = MIMEText('This is the body of the email')
  msg['Subject'] = 'Email Subject'
  msg['From'] = 'you@example.com'
  msg['To'] = 'recipient@example.com'

  with smtplib.SMTP('smtp.example.com', 587) as server:
    server.starttls()
    server.login('your_username', 'your_password')
    server.send_message(msg)
  ```

#### 19. Creating Virtual Environments

- Creating and activating a virtual environment:
  ```python
  import os
  import subprocess

  # Create virtual environment
  subprocess.run(['python3', '-m', 'venv', 'myenv'])

  # Activate virtual environment (Windows)
  os.system('myenv\\Scripts\\activate')

  # Activate virtual environment (Linux/Mac)
  os.system('source myenv/bin/activate')
  ```

#### 20. Integrating with CI/CD Tools

- Using the requests library to trigger a Jenkins job:
  ```python
  import requests

  url = 'http://your-jenkins-url/job/your-job-name/build'
  response = requests.post(url, auth=('user', 'token'))
  print(response.status_code)
  ```

#### 21. Database Migration

- Using Alembic for database migrations:
  ```bash
  alembic revision -m "initial migration"
  alembic upgrade head
  ```

#### 22. Testing Code

- Using unittest for unit testing:
  ```python
  import unittest

  def add(a, b):
    return a + b

  class TestMathFunctions(unittest.TestCase):
    def test_add(self):
      self.assertEqual(add(2, 3), 5)

  if __name__ == '__main__':
    unittest.main()
  ```

#### 23. Data Transformation with Pandas

- Using pandas for data manipulation:
  ```python
  import pandas as pd

  df = pd.read_csv('data.csv')
  df['new_column'] = df['existing_column'] * 2
  df.to_csv('output.csv', index=False)
  ```

#### 24. Using Python for Infrastructure as Code

- Using boto3 for AWS operations:
  ```python
  import boto3

  ec2 = boto3.resource('ec2')
  instances = ec2.instances.filter(Filters=[{'Name': 'instance-state-name', 'Values': ['running']}])
  for instance in instances:
    print(instance.id, instance.state)
  ```

#### 25. Web Scraping

- Using BeautifulSoup to scrape web pages:
  ```python
  import requests
  from bs4 import BeautifulSoup

  response = requests.get('http://example.com')
  soup = BeautifulSoup(response.content, 'html.parser')
  print(soup.title.string)
  ```

#### 26. Using Fabric for Remote Execution

- Running commands on a remote server:
  ```python
  from fabric import Connection

  conn = Connection(host='user@hostname', connect_kwargs={'password': 'your_password'})
  conn.run('uname -s')
  ```

#### 27. Automating AWS S3 Operations

- Upload and download files using boto3:
  ```python
  import boto3

  s3 = boto3.client('s3')

  # Upload a file
  s3.upload_file('local_file.txt', 'bucket_name', 's3_file.txt')

  # Download a file
  s3.download_file('bucket_name', 's3_file.txt', 'local_file.txt')
  ```

#### 28. Monitoring Application Logs

- Tail logs using tail -f equivalent in Python:
  ```python
  import time

  def tail_f(file):
    file.seek(0, 2) # Move to the end of the file
    while True:
      line = file.readline()
      if not line:
        time.sleep(0.1) # Sleep briefly
        continue
      print(line)

  with open('app.log', 'r') as log_file:
    tail_f(log_file)
  ```

#### 29. Container Health Checks

- Check the health of a running Docker container:
  ```python
  import docker

  client = docker.from_env()
  container = client.containers.get('container_id')
  print(container.attrs['State']['Health']['Status'])
  ```

#### 30. Using requests for Rate-Limited APIs

- Handle rate limiting in API requests:
  ```python
  import requests
  import time

  url = 'https://api.example.com/data'
  while True:
    response = requests.get(url)
    if response.status_code == 200:
      print(response.json())
      break
    elif response.status_code == 429: # Too Many Requests
      time.sleep(60) # Wait a minute before retrying
    else:
      print('Error:', response.status_code)
      break
  ```

#### 31. Docker Compose Integration

- Using docker-compose in Python:
  ```python
  import os
  import subprocess

  # Start services defined in docker-compose.yml
  subprocess.run(['docker-compose', 'up', '-d'])

  # Stop services
  subprocess.run(['docker-compose', 'down'])
  ```

#### 32. Terraform Execution

- Executing Terraform commands with subprocess:
  ```python
  import subprocess

  # Initialize Terraform
  subprocess.run(['terraform', 'init'])

  # Apply configuration
  subprocess.run(['terraform', 'apply', '-auto-approve'])
  ```

#### 33. Working with Prometheus Metrics

- Scraping and parsing Prometheus metrics:
  ```python
  import requests

  response = requests.get('http://localhost:9090/metrics')
  metrics = response.text.splitlines()

  for metric in metrics:
    print(metric)
  ```

#### 34. Using pytest for Testing

- Simple test case with pytest:
  ```python
  def add(a, b):
    return a + b

  def test_add():
    assert add(2, 3) == 5
  ```

#### 35. Creating Webhooks

- Using Flask to create a simple webhook:
  ```python
  from flask import Flask, request

  app = Flask(__name__)

  @app.route('/webhook', methods=['POST'])
  def webhook():
    data = request.json
    print('Received data:', data)
    return 'OK', 200

  if __name__ == '__main__':
    app.run(port=5000)
  ```

#### 36. Using Jinja2 for Configuration Templates

- Render configuration files with Jinja2:
  ```python
  from jinja2 import Template

  template = Template('Hello, {{ name }}!')
  rendered = template.render(name='DevOps')
  print(rendered)
  ```

#### 37. Encrypting and Decrypting Data

- Using cryptography to encrypt and decrypt:
  ```python
  from cryptography.fernet import Fernet

  # Generate a key
  key = Fernet.generate_key()
  cipher_suite = Fernet(key)

  # Encrypt
  encrypted_text = cipher_suite.encrypt(b'Secret Data')

  # Decrypt
  decrypted_text = cipher_suite.decrypt(encrypted_text)
  print(decrypted_text.decode())
  ```

#### 38. Error Monitoring with Sentry

- Sending error reports to Sentry:
  ```python
  import sentry_sdk

  sentry_sdk.init('your_sentry_dsn')

  def divide(a, b):
    return a / b

  try:
    divide(1, 0)
  except ZeroDivisionError as e:
    sentry_sdk.capture_exception(e)
  ```

#### 39. Setting Up Continuous Integration with GitHub Actions

- Sample workflow file (.github/workflows/ci.yml):
  ```yaml
  name: CI
  on: [push]
  jobs:
    build:
      runs-on: ubuntu-latest
      steps:
      - uses: actions/checkout@v2
      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.8'
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
      - name: Run tests
        run: |
          pytest
  ```

#### 40. Creating a Simple API with FastAPI

- Using FastAPI for high-performance APIs:
  ```python
  from fastapi import FastAPI

  app = FastAPI()

  @app.get('/items/{item_id}')
  async def read_item(item_id: int):
    return {'item_id': item_id}

  if __name__ == '__main__':
    import uvicorn
    uvicorn.run(app, host='0.0.0.0', port=8000)
  ```

#### 41. Log Aggregation with ELK Stack

- Sending logs to Elasticsearch:
  ```python
  from elasticsearch import Elasticsearch

  es = Elasticsearch(['http://localhost:9200'])

  log = {'level': 'info', 'message': 'This is a log message'}
  es.index(index='logs', body=log)
  ```

#### 42. Using pandas for ETL Processes

- Performing ETL with pandas:
  ```python
  import pandas as pd

  # Extract
  data = pd.read_csv('source.csv')

  # Transform
  data['new_column'] = data['existing_column'].apply(lambda x: x * 2)

  # Load
  data.to_csv('destination.csv', index=False)
  ```

#### 43. Serverless Applications with AWS Lambda

- Deploying a simple AWS Lambda function:
  ```python
  import json

  def lambda_handler(event, context):
    return {
      'statusCode': 200,
      'body': json.dumps('Hello from Lambda!')
    }
  ```

#### 44. Working with Redis

- Basic operations with Redis using redis-py:
  ```python
  import redis

  r = redis.StrictRedis(host='localhost', port=6379, db=0)

  # Set a key
  r.set('foo', 'bar')

  # Get a key
  print(r.get('foo'))
  ```

#### 45. Using pyngrok for Tunneling

- Create a tunnel to expose a local server:
  ```python
  from pyngrok import ngrok

  # Start the tunnel
  public_url = ngrok.connect(5000)
  print('Public URL:', public_url)

  # Keep the tunnel open
  input('Press Enter to exit...')
  ```

#### 46. Creating a REST API with Flask-RESTful

- Building REST APIs with Flask-RESTful:
  ```python
  from flask import Flask
  from flask_restful import Resource, Api

  app = Flask(__name__)
  api = Api(app)

  class HelloWorld(Resource):
    def get(self):
      return {'hello': 'world'}

  api.add_resource(HelloWorld, '/')

  if __name__ == '__main__':
    app.run(debug=True)
  ```

#### 47. Using asyncio for Asynchronous Tasks

- Running asynchronous tasks in Python:
  ```python
  import asyncio

  async def main():
    print('Hello')
  await asyncio.sleep(1)
    print('World')

  asyncio.run(main())
  ```

#### 48. Network Monitoring with scapy

- Packet sniffing using scapy:
  ```python
  from scapy.all import sniff

  def packet_callback(packet):
    print(packet.summary())

  sniff(prn=packet_callback, count=10)
  ```

#### 49. Handling Configuration Files with configparser

- Reading and writing to INI configuration files:
  ```python
  import configparser

  config = configparser.ConfigParser()
  config.read('config.ini')

  print(config['DEFAULT']['SomeSetting'])

  config['DEFAULT']['NewSetting'] = 'Value'
  with open('config.ini', 'w') as configfile:
    config.write(configfile)
  ```

#### 50. WebSocket Client Example

- Creating a WebSocket client with websocket-client:
  ```python
  import websocket

  def on_message(ws, message):
    print("Received message:", message)

  ws = websocket.WebSocketApp("ws://echo.websocket.org", on_message=on_message)
  ws.run_forever()
  ```

### Version Control

#### Git

##### 1. Git Setup and Configuration

- `git --version` # Check Git version
- `git config --global user.name "Your Name"` # Set global username
- `git config --global user.email "your.email@example.com"` # Set global email
- `git config --global core.editor "vim"` # Set default editor
- `git config --global init.defaultBranch main` # Set default branch name
- `git config --list` # View Git configuration
- `git help <command>` # Get help for a Git command

##### 2. Creating and Cloning Repositories

- `git init` # Initialize a new Git repository
- `git clone <repo_url>` # Clone an existing repository
- `git remote add origin <repo_url>` # Link local repo to a remote repo
- `git remote -v` # List remote repositories

##### 3. Staging and Committing Changes

- `git status` # Check the status of changes
- `git add <file>` # Add a file to the staging area
- `git add .` # Add all files to the staging area
- `git commit -m "Commit message"` # Commit staged files
- `git commit -am "Commit message"` # Add & commit changes in one step
- `git commit --amend -m "New message"` # Modify the last commit message

##### 4. Viewing History and Logs

- `git log` # Show commit history
- `git log --oneline` # Show history in one-line format
- `git log --graph --decorate --all` # Display commit history as a graph
- `git show <commit_id>` # Show details of a specific commit
- `git diff` # Show unstaged changes
- `git diff --staged` # Show staged but uncommitted changes
- `git blame <file>` # Show who modified each line of a file

##### 5. Branching and Merging

- `git branch` # List all branches
- `git branch <branch_name>` # Create a new branch
- `git checkout <branch_name>` # Switch to another branch
- `git checkout -b <branch_name>` # Create and switch to a new branch
- `git merge <branch_name>` # Merge a branch into the current branch
- `git branch -d <branch_name>` # Delete a local branch
- `git branch -D <branch_name>` # Force delete a branch

##### 6. Working with Remote Repositories

- `git fetch` # Fetch changes from remote repo
- `git pull origin <branch_name>` # Pull latest changes
- `git push origin <branch_name>` # Push changes to remote repo
- `git push -u origin <branch_name>` # Push and set upstream tracking

Note: The remaining content will be added later.
