# Vagrant Cheat Sheet

Vagrant is a tool for building, managing, and distributing development environments using virtual machines or containers. This cheat sheet covers essential CLI commands, Vagrantfile configuration, and common features (as of late 2025). Assumes basic Ruby knowledge for Vagrantfile (a Ruby DSL).

## Quick Start
1. Install Vagrant: Download from [developer.hashicorp.com/vagrant/install](https://developer.hashicorp.com/vagrant/install).
2. Initialize: `vagrant init <box-name>` (e.g., `vagrant init ubuntu/jammy64`).
3. Start: `vagrant up`.
4. SSH in: `vagrant ssh`.
5. Destroy: `vagrant destroy`.

## CLI Commands
Use `vagrant --help` for full options. Commands operate on the current directory's Vagrantfile unless `--directory` specified.

| Command | Description | Common Options | Example |
|---------|-------------|----------------|---------|
| `vagrant version` | Display Vagrant version. | None | `vagrant version` → "Vagrant 2.4.1" |
| `vagrant init [box]` | Initialize a new Vagrantfile. | `--box-version`, `--output` | `vagrant init generic/ubuntu2204 --box-version 4.2.0` |
| `vagrant up` | Start and provision the VM. | `--provider`, `--provision`, `--no-provision` | `vagrant up --provider virtualbox` |
| `vagrant halt` | Stop the VM gracefully. | `--force` | `vagrant halt` |
| `vagrant reload` | Restart VM, reapplying config. | `--provision` | `vagrant reload --provision` |
| `vagrant destroy` | Stop and delete the VM/resources. | `--force` | `vagrant destroy -f` |
| `vagrant status` | Show VM status. | None | `vagrant status` |
| `vagrant ssh [command]` | SSH into VM (runs command if provided). | `-- -L 8080:localhost:8080` (SSH opts) | `vagrant ssh` or `vagrant ssh sudo apt update` |
| `vagrant provision` | Run provisioners. | `--provision-with <name>` | `vagrant provision` |
| `vagrant suspend` | Save VM state to disk. | None | `vagrant suspend` |
| `vagrant resume` | Resume from suspended state. | None | `vagrant resume` |
| `vagrant box list` | List installed boxes. | `--boxed` | `vagrant box list` |
| `vagrant box add <name>` | Add a box from URL/HashiCorp Atlas. | `--provider`, `--box-version` | `vagrant box add generic/ubuntu2204` |
| `vagrant box remove <name>` | Remove a box. | `--provider`, `--box-version` | `vagrant box remove generic/ubuntu2204 --provider virtualbox` |
| `vagrant plugin list` | List installed plugins. | None | `vagrant plugin list` |
| `vagrant plugin install <name>` | Install a plugin. | None | `vagrant plugin install vagrant-vbguest` |
| `vagrant global-status` | Show status of all Vagrant environments. | `--prune` | `vagrant global-status` |

**Global Options** (apply to most commands):
- `-h, --help`: Show help.
- `-v, --verbose`: Verbose output.
- `-q, --quiet`: Quiet mode.
- `--debug`: Debug mode.

## Box Management
Boxes are pre-built VM images. Source from [app.vagrantup.com/boxes](https://app.vagrantup.com/boxes).
- Add: `vagrant box add <url-or-name>`.
- Update: `vagrant box update`.
- Prune unused: `vagrant box prune`.

## Vagrantfile Configuration
Vagrantfile is a Ruby file in your project root. Basic structure:
```ruby
Vagrant.configure("2") do |config|
  # Global config
end
```
Use `vagrant init` to generate a template.

### VM Configuration (`config.vm`)
| Option | Description | Example |
|--------|-------------|---------|
| `box` | Base box name. | `config.vm.box = "ubuntu/jammy64"` |
| `box_version` | Pin box version. | `config.vm.box_version = "1.0.0"` |
| `box_download_insecure` | Skip SSL verification for box download. | `config.vm.box_download_insecure = true` |
| `hostname` | Set VM hostname. | `config.vm.hostname = "myvm"` |
| `boot_timeout` | Boot timeout in seconds. | `config.vm.boot_timeout = 600` |

### Provider-Specific Config
Overrides for providers (e.g., VirtualBox).
```ruby
config.vm.provider "virtualbox" do |vb|
  vb.memory = "2048"
  vb.cpus = 2
  vb.gui = true  # Show GUI
end
```
Common providers:
- **VirtualBox** (default): `vb.name = "mybox"`, `vb.customize ["modifyvm", :id, "--usb", "on"]`.
- **VMware**: Requires license; use `vmware_fusion` or `vmware_workstation` provider.
- **Hyper-V**: Windows-native; `hyperv.use_vagrantmodules = true`.
- **libvirt**: For KVM/QEMU; plugin needed.
- **Docker**: Container-based; `config.vm.provider "docker" do |d| d.image = "ubuntu" end`.

### Networking (`config.vm.network`)
| Type | Description | Example |
|------|-------------|---------|
| Private Network | Internal IP (NAT'd). | `config.vm.network "private_network", ip: "192.168.50.4"` |
| Public Network | Bridged to host network. | `config.vm.network "public_network", bridge: "en1"` |
| Forwarded Port | Map host port to guest. | `config.vm.network "forwarded_port", guest: 80, host: 8080` |
| Host-Only | Custom host-only network. | `config.vm.network "private_network", type: "dhcp"` |

### Synced Folders (`config.vm.synced_folder`)
Share host folders with guest.
```ruby
config.vm.synced_folder ".", "/vagrant"  # Default: project dir to /vagrant
config.vm.synced_folder "./src", "/app/src", type: "nfs"  # NFS for performance
```
Options: `create: true`, `owner: "vagrant"`, `mount_options: ["dmode=755"]`.

### Provisioning (`config.vm.provision`)
Run scripts/config on `vagrant up`.
| Type | Description | Example |
|------|-------------|---------|
| Shell | Inline or file script. | `config.vm.provision "shell", inline: "echo 'Hello'"` or `path: "bootstrap.sh"` |
| File | Upload file to guest. | `config.vm.provision "file", source: "keys/id_rsa", destination: "~/.ssh/id_rsa"` |
| Ansible | Ansible playbook (plugin optional). | `config.vm.provision "ansible" do |ansible| ansible.playbook = "playbook.yml" end` |
| Chef (Solo/Client) | Chef recipes. | `config.vm.provision "chef_solo", run_list: ["recipe[apache]"]` |
| Puppet | Puppet manifests. | `config.vm.provision "puppet" do |puppet| puppet.manifests_path = "manifests" end` |
| Salt | Salt states. | `config.vm.provision "salt" do |salt| salt.masterless = true; salt.run_highstate = true end` |

Options for provisioners: `run: "always"` (or "once", "never"), `privileged: false`.

## Multi-VM Environments
Define multiple machines:
```ruby
Vagrant.configure("2") do |config|
  config.vm.define "web" do |web|
    web.vm.box = "ubuntu/jammy64"
    # web config...
  end
  config.vm.define "db" do |db|
    db.vm.box = "centos/8"
    # db config...
  end
end
```
Start specific: `vagrant up web`.

## Plugins and Advanced
- Install plugins: `vagrant plugin install <name>` (e.g., `vagrant-vbguest` for guest additions).
- Environment vars: `VAGRANT_VAGRANTFILE=otherfile vagrant up`.
- Sharing: `vagrant package` to create .box file.
- Debugging: `VAGRANT_LOG=debug vagrant up`.

## Tips
- Boxes: Search at [app.vagrantup.com](https://app.vagrantup.com).
- Logs: Check `vagrant up --debug` or `~/.vagrant.d/logs/`.
- Multi-host: Use `--machine-readable` for scripting.
- Security: Avoid `config.ssh.insert_key = true` in production.

For full docs, visit [developer.hashicorp.com/vagrant](https://developer.hashicorp.com/vagrant). This sheet focuses on core usage—extend with plugins for cloud providers (AWS, Azure).
