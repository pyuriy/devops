# Comprehensive SaltStack Lab Guide

Welcome to this hands-on lab for SaltStack (now part of the Salt Project), a powerful open-source automation tool for configuration management, orchestration, and remote execution. This guide is designed for beginners to intermediate users, assuming basic Linux command-line knowledge. We'll set up a lab environment quickly using Docker, then dive into core concepts with practical exercises.

The lab focuses on a **master-minion architecture**: The master controls minions (target systems) via secure, encrypted communication. By the end, you'll configure systems, apply states, and target minions dynamically.

**Estimated time:** 1-2 hours  
**Prerequisites:** Docker and Docker Compose installed on your host machine. (If not, install via [Docker docs](https://docs.docker.com/install/).)

## Lab 1: Environment Setup with Docker

To avoid complex VM provisioning, we'll use a pre-built Docker Compose setup for a Salt master and scalable minions. This spins up everything in minutes.

### Steps:
1. Clone the repo:
   ```
   git clone https://github.com/cyface/docker-saltstack.git
   cd docker-saltstack
   ```

2. Start the stack (1 master + 1 minion):
   ```
   docker-compose up
   ```
   - Run in detached mode for background: `docker-compose up -d`.
   - Scale to more minions: `docker-compose up --scale salt-minion=3` (great for testing targeting).

3. Verify the setup:
   - Shell into the master: `docker-compose exec salt-master bash`
   - Ping all minions: `salt '*' test.ping`
     - Expected output: Minions respond with `True`.
   - Check OS on minions: `salt '*' grains.get os`

4. Access a minion shell: `docker-compose exec salt-minion bash` (use `--index=1` for specific instances if scaled).

5. Stop/clean up: `docker-compose down`.

**Troubleshooting:** Watch logs in the terminal for DEBUG output. Ensure ports 4505-4506 are open if networking issues arise.

### docker-compose.yml (Reference)
```yaml
version: '3.7'
services:  
  salt-master:    
    build:      
      context: .      
      dockerfile: Dockerfile.master
  salt-minion:    
    build:      
      context: .      
      dockerfile: Dockerfile.minion    
    depends_on:      
      - salt-master
```
This builds from Dockerfiles in the repo, pre-configuring Salt.

**Exercise 1.1:** Scale to 2 minions (`docker-compose up --scale salt-minion=2`), ping them individually (e.g., `salt 'salt-minion-1' test.ping`), and restart one (`docker-compose restart salt-minion`).

## Lab 2: Key Management and Basic Connectivity

Salt uses public-key cryptography for secure auth. Minions auto-generate keys; the master must accept them.

### Steps (from master shell):
1. List pending keys: `salt-key -L`
   - You'll see unaccepted minion keys (e.g., `salt-minion`).

2. **Verify fingerprints** (security best practice):
   - Master fingerprint: `salt-key -F master` (copy `master.pub` value).
   - On minion: Shell in and run `salt-call key.finger --local`.
   - Match them; if good, accept: `salt-key -a salt-minion` (or `-A` for all).

3. Test connectivity: `salt '*' test.ping` (should return `True` for accepted minions).

**Exercise 2.1:** Reject a key (`salt-key -r salt-minion`), restart the minion (`docker-compose restart salt-minion`), and re-accept it after verifying fingerprints.

## Lab 3: Remote Execution Basics

Execute commands across minions without SSH. Use `salt '<target>' <module>.<function> [args]`.

### Key Commands:
| Command | Description | Example Output |
|---------|-------------|----------------|
| `salt '*' test.version` | Check Salt version on all minions | `salt-minion: 3007` |
| `salt '*' grains.items` | View system facts (grains: OS, IP, etc.) | YAML dict with hardware/OS info |
| `salt '*' cmd.run 'ls -l /etc'` | Run shell command | Directory listing |
| `salt '*' pkg.install vim` | Install package (auto-detects manager) | Installs Vim on all |
| `salt '*' disk.usage` | Check disk stats | Human-readable usage |

- **Targeting:** `*` = all; `salt-minion` = specific; `salt-minion-*` = glob.
- **Output formats:** Add `--out=pprint` for pretty-print YAML.
- Use `salt-call` on minions for local testing: `salt-call test.ping`.

**Exercise 3.1:** 
- Install `htop` on all: `salt '*' pkg.install htop`.
- List network interfaces: `salt '*' network.interfaces`.
- Run a custom command: `salt '*' cmd.run 'uptime'`.

## Lab 4: Configuration Management with States

States (SLS files in YAML) define "desired state" (e.g., install packages, manage files). Apply with `state.apply`.

### Steps:
1. Create `/srv/salt` dir on master (if not exists): `mkdir -p /srv/salt`.

2. Simple state: Install Nginx.
   - File: `/srv/salt/nginx.sls`
     ```yaml
     nginx:
       pkg.installed: []
       service.running:
         - require:
           - pkg: nginx  # Dependency: Install before starting service
     ```
   - Apply: `salt '*' state.apply nginx`
     - Minions report `Succeeded` if applied.

3. File management: Add a config.
   - Create `/srv/salt/nginx.conf` with basic content (e.g., `server { listen 80; }`).
   - Update `/srv/salt/nginx.sls`:
     ```yaml
     /etc/nginx/sites-available/default:
       file.managed:
         - source: salt://nginx.conf
         - mode: 644
     ```
   - Re-apply: `salt '*' state.apply nginx`.

4. Top file for orchestration: `/srv/salt/top.sls` maps states to targets.
   ```yaml
   base:
     '*':  # All minions
       - nginx
   ```
   - Highstate (full apply): `salt '*' state.highstate` or `salt '*' state.apply`.

**Exercise 4.1:** 
- Create a `users.sls` state to add a user (`user.present: - name: labuser - shell: /bin/bash`).
- Use Jinja for OS-specific packages: In a state, `{% if grains['os'] == 'Ubuntu' %}apt{% else %}yum{% endif %}`.
- Apply to one minion only: `salt 'salt-minion' state.apply users`.

## Lab 5: Advanced Targeting and Pillars

### Targeting:
- **Grains:** `salt -G 'os:Ubuntu' test.ping` (OS-based).
- **Compound:** `salt -C 'G@os:Ubuntu and not web*' pkg.upgrade`.
- **Nodegroups:** Define in master config (`/etc/salt/master`): `nodegroups: webservers: 'web* or L@roles:web'`.

### Pillars (Secure Data):
1. Create `/srv/pillar/users.sls`:
   ```yaml
   users:
     labuser:
       shell: /bin/bash
   ```
2. Top pillar: `/srv/pillar/top.sls`
   ```yaml
   base:
     '*':
       - users
   ```
3. Use in state (`/srv/salt/user_setup.sls`):
   ```yaml
   {% for name, info in pillar['users'].items() %}
   {{ name }}:
     user.present:
       - shell: {{ info.shell }}
   {% endfor %}
   ```
4. Apply: `salt '*' state.apply user_setup`.

**Exercise 5.1:** Target Ubuntu minions (if varied in your setup) to install Apache (`httpd` vs. `apache2`). Use pillars for a secret (e.g., API key) in a file state.

## Next Steps and Resources
- Scale your lab: Add real VMs or cloud instances.
- Explore reactors/orchestration for event-driven automation.
- Official Tutorials: [Salt in 10 Minutes](https://docs.saltproject.io/en/3007/topics/tutorials/walkthrough.html) and [User Guide](https://docs.saltproject.io).
- Video: [Zero to Hero on YouTube](https://www.youtube.com/watch?v=EiQty-dVfqY).
- Awesome List: GitHub's [awesome-saltproject](https://github.com/saltstack/awesome-saltproject) for more formulas and tools.

This lab gets you productive—experiment and break things!

# SaltStack Lab Resources

The follwoing resources provide installation guides, tutorials, videos, and community tools to support your SaltStack learning journey.

## Installation and Setup
- **[Docker Installation Docs](https://docs.docker.com/install/)**: Official guide for installing Docker and Docker Compose on various platforms, essential for the lab's containerized environment.
- **[Docker SaltStack GitHub Repo](https://github.com/cyface/docker-saltstack.git)**: Pre-built Docker Compose setup for Salt master and minions, used to quickly spin up the lab environment.

## Tutorials and Documentation
- **[Salt in 10 Minutes Tutorial](https://docs.saltproject.io/en/3007/topics/tutorials/walkthrough.html)**: Quick-start walkthrough from the Salt Project docs, covering basic installation and first commands.
- **[Salt Project User Guide](https://docs.saltproject.io)**: Comprehensive official documentation for SaltStack, including modules, states, and advanced features.

## Videos and Community Resources
- **[Zero to Hero SaltStack Video](https://www.youtube.com/watch?v=EiQty-dVfqY)**: YouTube tutorial series taking you from beginner to advanced SaltStack concepts with practical demos.
- **[Awesome SaltProject GitHub List](https://github.com/saltstack/awesome-saltproject)**: Curated collection of Salt formulas, tools, and integrations for extending your automation workflows.
