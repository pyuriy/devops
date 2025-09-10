# Ansible Cheatsheet

This cheatsheet provides a quick reference for common Ansible commands, playbook structure, and best practices.

## Table of Contents
- [Basic Concepts](#basic-concepts)
- [Common Commands](#common-commands)
- [Playbook Structure](#playbook-structure)
- [Common Modules](#common-modules)
- [Variables](#variables)
- [Loops](#loops)
- [Conditionals](#conditionals)
- [Roles](#roles)
- [Inventory](#inventory)
- [Best Practices](#best-practices)
- [Debugging](#debugging)

## Basic Concepts
- **Ansible**: Automation tool for configuration management, application deployment, and task automation.
- **Inventory**: File defining hosts and groups to manage.
- **Playbook**: YAML file containing tasks to execute on hosts.
- **Module**: Pre-built units of work (e.g., `copy`, `apt`, `service`).
- **Role**: Reusable, modular way to organize tasks, handlers, and templates.
- **Facts**: System information gathered from managed hosts.

## Common Commands
| Command | Description |
|---------|-------------|
| `ansible <host-pattern> -m <module> -a "<arguments>"` | Run a single module against hosts. |
| `ansible-playbook playbook.yml` | Execute a playbook. |
| `ansible-inventory --list` | Display inventory details. |
| `ansible-galaxy init <role_name>` | Create a new role structure. |
| `ansible-vault create secrets.yml` | Create an encrypted vault file. |
| `ansible-vault edit secrets.yml` | Edit an encrypted vault file. |
| `ansible-doc <module_name>` | View module documentation. |
| `ansible <host> -m ping` | Test connectivity to hosts. |

**Options**:
- `-i <inventory_file>`: Specify custom inventory file.
- `-u <user>`: Specify remote user.
- `-k`: Prompt for SSH password.
- `--vault-id @prompt`: Prompt for vault password.
- `-v`, `-vv`, `-vvv`: Increase verbosity for debugging.

## Playbook Structure
```yaml
---
- name: Playbook Name
  hosts: all
  become: yes  # Enable sudo
  vars:
    key: value
  tasks:
    - name: Task Name
      module_name:
        parameter: value
      register: result
      notify: handler_name
  handlers:
    - name: handler_name
      module_name:
        parameter: value
```

- **hosts**: Target hosts or groups from inventory.
- **become**: Run tasks with elevated privileges.
- **vars**: Define playbook-level variables.
- **tasks**: List of actions to perform.
- **handlers**: Tasks triggered by changes (e.g., service restart).
- **register**: Store task output in a variable.

## Common Modules
| Module | Description |
|--------|-------------|
| `apt` / `yum` / `dnf` | Manage packages (Debian/RedHat). |
| `copy` | Copy files to remote hosts. |
| `file` | Manage files and directories (permissions, ownership). |
| `service` | Manage services (start, stop, restart). |
| `template` | Render Jinja2 templates to files. |
| `user` | Manage user accounts. |
| `group` | Manage groups. |
| `shell` / `command` | Run shell commands. |
| `get_url` | Download files from URLs. |
| `cron` | Manage cron jobs. |
| `lineinfile` | Modify specific lines in files. |

**Example**:
```yaml
- name: Install Nginx
  apt:
    name: nginx
    state: present
  become: yes

- name: Copy config file
  copy:
    src: ./nginx.conf
    dest: /etc/nginx/nginx.conf
    mode: '0644'
  notify: Restart Nginx
```

## Variables
- **Defining Variables**:
  ```yaml
  vars:
    http_port: 80
    app_name: myapp
  ```
- **Using Variables**:
  ```yaml
  - name: Use variable in task
    debug:
      msg: "The app {{ app_name }} runs on port {{ http_port }}"
  ```
- **Variable Precedence** (highest to lowest):
  1. Command-line (`-e "var=value"`)
  2. Role defaults
  3. Inventory vars
  4. Playbook vars
  5. Facts
- **Facts**:
  ```yaml
  - name: Display hostname
    debug:
      msg: "{{ ansible_facts['hostname'] }}"
  ```

## Loops
- **Loop over list**:
  ```yaml
  - name: Install multiple packages
    apt:
      name: "{{ item }}"
      state: present
    loop:
      - vim
      - git
      - curl
  ```
- **Loop over dictionary**:
  ```yaml
  - name: Create users
    user:
      name: "{{ item.name }}"
      state: present
      groups: "{{ item.groups }}"
    loop:
      - { name: 'alice', groups: 'admin' }
      - { name: 'bob', groups: 'users' }
  ```

## Conditionals
- **When clause**:
  ```yaml
  - name: Install package on Ubuntu
    apt:
      name: nginx
      state: present
    when: ansible_facts['os_family'] == 'Debian'
  ```
- **Multiple conditions**:
  ```yaml
  - name: Do something
    command: /usr/bin/some_command
    when:
      - ansible_facts['os_family'] == 'RedHat'
      - ansible_facts['distribution_major_version'] | int >= 8
  ```

## Roles
- **Role Directory Structure**:
  ```
  roles/
    my_role/
      defaults/main.yml      # Default variables
      tasks/main.yml        # Main tasks
      handlers/main.yml     # Handlers
      templates/            # Jinja2 templates
      files/                # Static files
      vars/main.yml         # Role-specific variables
      meta/main.yml         # Role metadata
  ```
- **Using Roles**:
  ```yaml
  - hosts: webservers
    roles:
      - my_role
      - { role: another_role, vars: { custom_var: value } }
  ```

## Inventory
- **INI Format**:
  ```
  [webservers]
  web1.example.com ansible_host=192.168.1.10
  web2.example.com ansible_host=192.168.1.11

  [dbservers]
  db1.example.com

  [all:vars]
  ansible_user=admin
  ansible_ssh_private_key_file=~/.ssh/id_rsa
  ```
- **YAML Format**:
  ```yaml
  all:
    vars:
      ansible_user: admin
    hosts:
      web1:
        ansible_host: 192.168.1.10
    children:
      webservers:
        hosts:
          web1:
          web2:
      dbservers:
        hosts:
          db1:
  ```
- **Dynamic Inventory**: Use scripts or plugins (e.g., AWS, GCP) to generate inventory dynamically.

## Best Practices
- Use roles for modularity and reusability.
- Store sensitive data in Ansible Vault.
- Use `state` parameter in modules (e.g., `present`, `absent`).
- Leverage `become` for privilege escalation instead of running Ansible as root.
- Use meaningful task names for clarity.
- Group hosts logically in inventory.
- Validate playbooks with `ansible-playbook --syntax-check`.
- Use version control (e.g., Git) for playbooks and roles.
- Avoid `shell`/`command` modules when specific modules exist.

## Debugging
- **Verbose Output**:
  ```bash
  ansible-playbook playbook.yml -vvv
  ```
- **Debug Module**:
  ```yaml
  - name: Print variable
    debug:
      var: my_variable
  ```
- **Check Mode** (Dry Run):
  ```bash
  ansible-playbook playbook.yml --check
  ```
- **Syntax Check**:
  ```bash
  ansible-playbook playbook.yml --syntax-check
  ```
- **List Tasks**:
  ```bash
  ansible-playbook playbook.yml --list-tasks
  ```

---

This cheatsheet covers the essentials for using Ansible effectively. For more details, refer to the [Ansible Documentation](https://docs.ansible.com/).
