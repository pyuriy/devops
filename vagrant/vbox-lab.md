# Comprehensive Vagrant Lab with Oracle VirtualBox

This lab provides hands-on exercises to master Vagrant using Oracle VirtualBox as the provider. We'll build progressively from a single VM to a multi-VM web server setup with provisioning and networking. The lab assumes a Unix-like host (Linux/macOS; Windows users adapt paths/commands accordingly).

**Duration:** 2-3 hours  
**Objectives:**  
- Initialize and manage Vagrant VMs with VirtualBox.  
- Configure networking, synced folders, and provisioning.  
- Build a multi-VM environment simulating a web app stack.  

## Prerequisites
1. **Install Software:**
   - **VirtualBox:** Download from [virtualbox.org](https://www.virtualbox.org/wiki/Downloads) (version 7.0+ recommended as of 2025).
   - **Vagrant:** Download from [developer.hashicorp.com/vagrant/install](https://developer.hashicorp.com/vagrant/install) (version 2.4+).
   - Verify: `vagrant --version` and `VBoxManage --version`.

2. **Project Directory:** Create a new folder: `mkdir vagrant-lab && cd vagrant-lab`.

3. **Optional Plugin:** Install VBGuest for auto-guest additions: `vagrant plugin install vagrant-vbguest`.

4. **Test Setup:** Run `vagrant init --minimal` to generate a basic Vagrantfile, then `vagrant up --provider virtualbox` to ensure VirtualBox works. Destroy with `vagrant destroy`.

**Troubleshooting:** If VirtualBox fails, check BIOS/UEFI for virtualization (VT-x/AMD-V). On macOS, ensure Rosetta 2 for ARM.

## Lab 1: Basic Single-VM Setup
**Goal:** Launch an Ubuntu VM and access it via SSH.

1. **Initialize Project:**
   ```
   vagrant init ubuntu/jammy64
   ```
   This creates a `Vagrantfile` using Ubuntu 22.04 LTS box.

2. **Customize Vagrantfile (Basic Config):**
   Edit `Vagrantfile` (Ruby syntax; use `nano` or your editor):
   ```ruby
   Vagrant.configure("2") do |config|
     config.vm.box = "ubuntu/jammy64"
     config.vm.box_version = ">= 20241021.0.0"  # Pin recent version

     # Provider: VirtualBox
     config.vm.provider "virtualbox" do |vb|
       vb.memory = 2048  # RAM in MB
       vb.cpus = 2       # CPU cores
       vb.name = "lab-vm-1"  # VM name in VirtualBox
       vb.gui = false    # Headless (set true for GUI)
     end

     # Basic networking (default NAT)
     config.vm.network "forwarded_port", guest: 22, host: 2222
   end
   ```

3. **Launch VM:**
   ```
   vagrant up --provider virtualbox
   ```
   - Expected Output: Downloads box (~500MB), creates VM, boots (5-10 min first time).
   - Monitor in VirtualBox GUI: "lab-vm-1" should show "Running".

4. **Access VM:**
   ```
   vagrant ssh
   ```
   - Inside VM: Run `whoami` (should output `vagrant`).
   - Install a tool: `sudo apt update && sudo apt install -y htop`.
   - Exit: `exit`.

5. **Manage VM:**
   | Command | Purpose | Expected Result |
   |---------|---------|-----------------|
   | `vagrant status` | Check state | "running (virtualbox)" |
   | `vagrant halt` | Stop VM | Powers off gracefully |
   | `vagrant up` | Restart | Resumes from halted |
   | `vagrant reload --provision` | Restart + reprovision | Applies changes |
   | `vagrant destroy -f` | Delete VM | Removes VM/files (reversible via box) |

**Verification:** SSH in and run `htop`. If issues: Check `vagrant up -v` for verbose logs.

**Cleanup:** `vagrant destroy -f` (repeat per lab unless specified).

## Lab 2: Networking Configuration
**Goal:** Expose services and set up private networking.

1. **Update Vagrantfile:**
   Add to the `config.vm` block:
   ```ruby
   # Private network (host-only adapter)
   config.vm.network "private_network", ip: "192.168.56.10", type: "dhcp"  # Or static: ip: "192.168.56.10"

   # Port forwarding for a web server (e.g., Apache on port 80)
   config.vm.network "forwarded_port", guest: 80, host: 8080
   config.vm.network "forwarded_port", guest: 443, host: 8443
   ```

2. **Provision a Web Server (Inline Shell):**
   Add provisioning:
   ```ruby
   config.vm.provision "shell", inline: <<-SHELL
     sudo apt-get update
     sudo apt-get install -y apache2
     sudo systemctl start apache2
     sudo systemctl enable apache2
     echo "<h1>Hello from Vagrant Lab!</h1>" | sudo tee /var/www/html/index.html
   SHELL
   ```

3. **Launch and Test:**
   ```
   vagrant up --provider virtualbox
   vagrant ssh  # Inside: curl localhost (should show HTML)
   ```
   - From host: Open browser to `http://localhost:8080` (Apache welcome page modified).
   - Private IP: From host, ping `192.168.56.10` (if VirtualBox host-only network enabled).

4. **Advanced Networking Tips:**
   - **Bridged (Public):** Replace private with `config.vm.network "public_network", bridge: "en0"` (uses host's WiFi/Ethernet; exposes to LAN).
   - **Multiple Adapters:** VirtualBox auto-creates NAT; add host-only via `VBoxManage hostonlyif create`.
   - **Troubleshoot:** `vagrant reload` after changes. Check VirtualBox Network settings.

**Verification:** Browser shows "Hello from Vagrant Lab!" at :8080. Ping private IP succeeds.

## Lab 3: Synced Folders and File Sharing
**Goal:** Share host files with guest for development workflow.

1. **Create Host Files:**
   ```
   mkdir -p src/app
   echo "puts 'Hello from Ruby!'" > src/app/hello.rb
   ```

2. **Update Vagrantfile:**
   Add synced folder (default "." to "/vagrant" exists; customize):
   ```ruby
   config.vm.synced_folder "./src", "/home/vagrant/app", create: true, owner: "vagrant", group: "vagrant"
   # For better perf on macOS/Linux: type: "nfs" (requires NFS setup)
   ```

3. **Provision to Use Synced Files:**
   Update shell provision:
   ```ruby
   config.vm.provision "shell", inline: <<-SHELL
     sudo apt-get update
     sudo apt-get install -y ruby-full apache2
     # Use synced file
     cd /home/vagrant/app
     ruby hello.rb  # Outputs "Hello from Ruby!"
     echo "<?php echo 'Synced PHP works!'; ?>" > /home/vagrant/app/hello.php
     sudo ln -sf /home/vagrant/app/hello.php /var/www/html/
   SHELL
   ```

4. **Launch and Interact:**
   ```
   vagrant up --provider virtualbox
   vagrant ssh
   cd /home/vagrant/app
   ls  # Shows hello.rb
   ruby hello.rb  # Runs host-synced file
   ```
   - Edit `src/app/hello.rb` on host, then in VM: `ruby hello.rb` (changes reflect instantly).

5. **Options Table:**
   | Option | Purpose | Example |
   |--------|---------|---------|
   | `create: true` | Auto-create guest dir | As above |
   | `type: "nfs"` | Faster for large dirs (Linux/macOS) | `type: "nfs", nfs_version: 4` |
   | `mount_options` | Permissions | `mount_options: ["dmode=755", "fmode=644"]` |
   | `disabled: true` | Skip for production | N/A |

**Verification:** Edit host file, SSH in, run it—output updates. Browser at :8080 shows PHP page.

**Tip:** For Windows, use `type: "rsync"` or SMB.

## Lab 4: Advanced Provisioning
**Goal:** Automate setup with scripts and multi-provisioners.

1. **Create Provision Script:**
   ```
   cat > bootstrap.sh << EOF
   #!/bin/bash
   apt-get update
   apt-get install -y nginx git nodejs npm
   echo "Provision complete: $(date)" > /var/log/vagrant-provision.log
   npm install -g http-server
   EOF
   chmod +x bootstrap.sh
   ```

2. **Update Vagrantfile:**
   ```ruby
   # Shell from file
   config.vm.provision "shell", path: "bootstrap.sh", privileged: false

   # File upload (e.g., config file)
   config.vm.provision "file", source: "./nginx.conf", destination: "/home/vagrant/nginx.conf"

   # Ansible (install plugin if needed: vagrant plugin install vagrant-ansible)
   # config.vm.provision "ansible_local" do |ansible|
   #   ansible.playbook = "playbook.yml"
   # end
   ```

3. **Launch:**
   ```
   vagrant up --provider virtualbox --provision
   vagrant ssh  # cat /var/log/vagrant-provision.log
   ```
   - Expected: Nginx installed; run `nginx -t` to test config if uploaded.

4. **Provision Modes:**
   | Mode | When to Run | Example |
   |------|-------------|---------|
   | Default (`"once"`) | First `up` only | Basic setup |
   | `"always"` | Every `up/reload` | Dev changes |
   | `"never"` | Skip | `provision: { run: "never" }` |
   | Manual | `vagrant provision` | Updates |

**Verification:** SSH and run `nginx -v`. Check log for timestamp.

## Lab 5: Multi-VM Environment (Web + DB Stack)
**Goal:** Simulate a LAMP stack with web and DB VMs communicating.

1. **Backup Current Vagrantfile:** `cp Vagrantfile Vagrantfile.backup`.

2. **Multi-VM Vagrantfile:**
   Replace content:
   ```ruby
   Vagrant.configure("2") do |config|
     # Shared config
     config.vm.box = "ubuntu/jammy64"
     config.vm.provider "virtualbox" do |vb|
       vb.memory = 1024
       vb.cpus = 1
     end

     # Web VM
     config.vm.define "web" do |web|
       web.vm.hostname = "web.lab"
       web.vm.network "private_network", ip: "192.168.56.10"
       web.vm.network "forwarded_port", guest: 80, host: 8080
       web.vm.synced_folder "./src", "/var/www/html", create: true

       web.vm.provision "shell", inline: <<-SHELL
         apt-get update
         apt-get install -y apache2 php libapache2-mod-php php-mysql
         a2enmod rewrite
         systemctl restart apache2
         echo "<?php phpinfo(); ?>" > /var/www/html/info.php
       SHELL
     end

     # DB VM
     config.vm.define "db" do |db|
       db.vm.hostname = "db.lab"
       db.vm.network "private_network", ip: "192.168.56.11"

       db.vm.provision "shell", inline: <<-SHELL
         apt-get update
         apt-get install -y mariadb-server
         sudo mysql_secure_installation <<EOF

         y
         y
         rootpass
         rootpass
         y
         y
         y
         y
         EOF
         sudo sed -i 's/bind-address.*/bind-address = 0.0.0.0/' /etc/mysql/mariadb.conf.d/50-server.cnf
         sudo systemctl restart mariadb
       SHELL
     end
   end
   ```

3. **Launch Stack:**
   ```
   vagrant up  # Starts both (use 'vagrant up web' for single)
   ```
   - Access: `vagrant ssh web` or `vagrant ssh db`.
   - Test Connectivity: From web SSH: `ping 192.168.56.11`.
   - Browser: `http://localhost:8080/info.php` (PHP info).

4. **Orchestration:**
   - Status: `vagrant status` (shows both).
   - SSH Specific: `vagrant ssh web`.
   - Destroy: `vagrant destroy` (both) or `vagrant destroy db`.

**Verification:** From web VM: `mysql -h 192.168.56.11 -u root -p` (password: rootpass). Create DB if connected.

**Extensions:** Add Ansible for complex provisioning or Docker provider for containers.

## Cleanup and Best Practices
- **Full Cleanup:** `vagrant global-status` to list all, then `vagrant destroy -f <id>`. Prune boxes: `vagrant box prune`.
- **Performance:** Allocate 4GB+ host RAM for multi-VM. Use SSD for boxes.
- **Security:** Change default SSH key: `config.ssh.insert_key = false`. Use firewalls in provisioners.
- **Debugging:** `VAGRANT_LOG=debug vagrant up > log.txt 2>&1`.
- **Next Steps:** Explore cloud providers (vagrant-aws plugin) or Kubernetes integration.

This lab covers core Vagrant-VirtualBox workflows. Experiment by forking the Vagrantfile. For issues, check [Vagrant docs](https://developer.hashicorp.com/vagrant/docs/providers/virtualbox) or VirtualBox forums. Happy virtualizing!
