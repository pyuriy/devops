# Comprehensive Ansible Role Lab

This hands-on lab guides you through creating, structuring, testing, and deploying Ansible roles. It's designed for intermediate users familiar with basic Ansible playbooks. We'll build a role to deploy a simple web server (NGINX) on Linux hosts, adding features like variables, templates, handlers, and dependencies. Estimated time: 1-2 hours.

**Prerequisites:**
- Ansible installed (version 2.14+ recommended; check with `ansible --version`).
- Vagrant and VirtualBox (or similar) for a test VM, or access to remote Linux hosts (e.g., Ubuntu/CentOS).
- Basic YAML and Linux knowledge.
- Inventory file with at least one test host (e.g., `inventory.ini` with `[webservers]` group).

We'll use a local Vagrant VM as the target host for simplicity.

## Lab Setup (10 minutes)

1. **Create Project Directory:**
   ```bash
   mkdir ansible-role-lab && cd ansible-role-lab
   ansible-galaxy init roles/webserver  # Initializes role skeleton
   ```

2. **Set Up Inventory and Test Host:**
   Create `inventory.ini`:
   ```
   [webservers]
   test-vm ansible_host=127.0.0.1 ansible_port=2222 ansible_user=vagrant ansible_ssh_private_key_file=.vagrant/machines/default/virtualbox/private_key
   ```
   
   Start a Vagrant VM (if using Vagrant):
   ```bash
   vagrant init ubuntu/jammy64  # Or centos/stream9 for RHEL
   vagrant up --provider=virtualbox
   vagrant provision -f <(echo '# Empty provisioner to bootstrap')
   ```
   Adjust `ansible_port` if needed (default Vagrant SSH is 2222).

3. **Configure Ansible:**
   Create `ansible.cfg`:
   ```
   [defaults]
   inventory = inventory.ini
   roles_path = roles
   host_key_checking = False
   ```

4. **Verify Connectivity:**
   ```bash
   ansible webservers -m ping
   ```
   Expected output: `test-vm | SUCCESS => {"changed": false, "ping": "pong"}`

## Step 1: Basic Role Structure (15 minutes)

Explore the generated `roles/webserver/` directory:
```
roles/webserver/
├── defaults/     # Default vars (low precedence)
├── files/        # Static files
├── handlers/     # Handler tasks
├── meta/         # Metadata & dependencies
├── tasks/        # Main tasks
├── templates/    # Jinja2 templates
├── tests/        # Test files (ignore for now)
└── vars/         # Role vars (high precedence)
```

**Task 1.1: Add Default Variables**
In `roles/webserver/defaults/main.yml`:
```yaml
---
# Default NGINX config
nginx_port: 8080
nginx_user: www-data
server_name: localhost
docroot: /var/www/html
```

**Task 1.2: Create Basic Tasks**
In `roles/webserver/tasks/main.yml`:
```yaml
---
- name: Install NGINX
  ansible.builtin.package:
    name: nginx
    state: present
  become: true

- name: Start and enable NGINX
  ansible.builtin.systemd:
    name: nginx
    state: started
    enabled: true
  become: true
```

**Task 1.3: Run the Role**
Create `site.yml` playbook:
```yaml
---
- hosts: webservers
  become: true
  roles:
    - webserver
```

Execute:
```bash
ansible-playbook site.yml
```
Expected: NGINX installed and running. Verify on host: `curl http://localhost:80` (should show NGINX welcome page).

## Step 2: Templates and Handlers (20 minutes)

**Task 2.1: Add a Custom Config Template**
Create `roles/webserver/templates/nginx.conf.j2`:
```
server {
    listen {{ nginx_port }};
    server_name {{ server_name }};
    root {{ docroot }};
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Update `roles/webserver/tasks/main.yml` (add after install task):
```yaml
- name: Deploy NGINX config
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/sites-available/default
    backup: true
  become: true
  notify: Restart NGINX

- name: Ensure docroot exists
  ansible.builtin.file:
    path: "{{ docroot }}"
    state: directory
    mode: '0755'
  become: true
```

**Task 2.2: Add Handlers**
In `roles/webserver/handlers/main.yml`:
```yaml
---
- name: Restart NGINX
  ansible.builtin.systemd:
    name: nginx
    state: restarted
  become: true
```

**Task 2.3: Test Changes**
Override vars in `site.yml`:
```yaml
---
- hosts: webservers
  become: true
  vars:
    nginx_port: 8081
    docroot: /var/www/mysite
  roles:
    - webserver
```
Run: `ansible-playbook site.yml`
- Check: `sudo nginx -t` (config valid), `curl http://localhost:8081` (should 404, but server up).
- Handler triggers only on config change (idempotent).

## Step 3: Variables and Conditionals (15 minutes)

**Task 3.1: OS-Specific Tasks**
Create `roles/webserver/tasks/redhat.yml` (for RHEL/CentOS):
```yaml
---
- name: Install NGINX on RedHat
  ansible.builtin.dnf:
    name: nginx
    state: present
  become: true
```

Update `roles/webserver/tasks/main.yml`:
```yaml
---
- name: Include OS-specific install
  include_tasks: "{{ ansible_os_family | lower }}.yml"
  when: ansible_os_family in ['RedHat', 'Debian']

# ... (rest of tasks)
```

**Task 3.2: Role Variables with Precedence**
In `roles/webserver/vars/main.yml` (high precedence, e.g., for Debian):
```yaml
---
nginx_user: nginx  # Overrides default for Debian
```

Test on mixed inventory (add a CentOS VM if possible). Run playbook; vars should apply correctly.

## Step 4: Dependencies and Meta (15 minutes)

**Task 4.1: Add a Dependency Role**
Create a simple `roles/common/` for firewall setup:
```bash
ansible-galaxy init roles/common
```
In `roles/common/tasks/main.yml`:
```yaml
---
- name: Allow HTTP on firewall
  ansible.posix.firewalld:
    service: http
    permanent: true
    state: enabled
    immediate: true
  become: true
  when: ansible_os_family == 'RedHat'
```

In `roles/webserver/meta/main.yml`:
```yaml
---
dependencies:
  - role: common
    vars:
      http_port: "{{ nginx_port }}"
galaxy_info:
  author: Your Name
  description: NGINX Web Server Role
  min_ansible_version: 2.14
  platforms:
    - name: Ubuntu
      versions:
        - jammy
    - name: EL
      versions:
        - "9"
```

**Task 4.2: Run with Dependency**
Execute `ansible-playbook site.yml`. Firewall rule applies before webserver tasks.

## Step 5: Testing the Role (15 minutes)

**Task 5.1: Molecule Testing (Optional Setup)**
Install Molecule: `pip install molecule molecule-plugins[containers] docker` (requires Docker).
In `roles/webserver/molecule/default/molecule.yml` (generated by init; update):
```yaml
---
dependency:
  name: galaxy
driver:
  name: docker
platforms:
  - name: instance
    image: ubuntu:22.04
    pre_build_image: true
provisioner:
  name: ansible
verifier:
  name: ansible
```

Run tests:
```bash
cd roles/webserver
molecule test
```
- Creates container, runs role, verifies (add custom tests in `tests/test_default.yml`, e.g., assert NGINX running).

**Task 5.2: Manual Verification**
- Idempotency: Run playbook twice; no changes on second run.
- Tags: Add `tags: install` to install task; run `ansible-playbook site.yml --tags install`.
- Dry Run: `ansible-playbook site.yml --check`.

## Exercises (Advanced Practice, 20+ minutes)

1. **Extend the Role:** Add a task to deploy a static `index.html` from `files/` dir. Use `copy` module and notify handler.

2. **Argument Specs:** In `roles/webserver/meta/argument_specs.yml`:
   ```yaml
   ---
   argument_specs:
     main:
       options:
         nginx_port:
           type: int
           required: true
           default: 80
   ```
   Test invalid input: `ansible-playbook site.yml -e "nginx_port=abc"` (should error).

3. **Dynamic Include:** In playbook, use `include_role` with loop over hosts for multi-site deploy:
   ```yaml
   - name: Deploy multi-site
     include_role:
       name: webserver
     vars:
       nginx_port: "{{ item.port }}"
       server_name: "{{ item.name }}"
     loop:
       - { port: 8080, name: site1 }
       - { port: 8081, name: site2 }
   ```

4. **Publish to Galaxy:** Update `meta/main.yml` with full `galaxy_info`. Run `ansible-galaxy role build roles/webserver` and upload to Galaxy.

## Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| SSH Connection Fails | Check `ansible_host`, keys, and `vagrant ssh-config > ansible_ssh_common_args="-o StrictHostKeyChecking=no"`. |
| Package Not Found | Use `ansible.builtin.package` for auto-detection; test OS facts with `ansible webservers -m setup -a "filter=ansible_os_family"`. |
| Handler Not Triggering | Ensure `notify` matches handler name exactly; use `flush_handlers` if needed mid-play. |
| Var Precedence Issues | Debug with `-v` flag; remember: defaults < playbook vars < role vars. |

## Cleanup
```bash
vagrant destroy  # If using Vagrant
rm -rf roles/ ansible-role-lab/
```

This lab demonstrates role reusability. For more, explore Ansible Galaxy roles like `geerlingguy.nginx`. Experiment and iterate!
