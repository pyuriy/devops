# Comprehensive Ansible Role Cheat Sheet

This cheat sheet covers the essentials of Ansible roles for reusable automation. Based on official documentation as of 2025.

## Overview
- Roles bundle vars, files, tasks, handlers, and modules into a standard directory structure for reuse.
- Enable sharing via Ansible Galaxy or local paths.
- Roles load automatically based on structure; unused dirs can be omitted.

## Directory Structure
```
roles/
└── <role_name>/
    ├── tasks/          # Main tasks (main.yml)
    ├── handlers/       # Handlers (main.yml)
    ├── templates/      # Jinja2 templates (*.j2)
    ├── files/          # Static files for copy/script
    ├── vars/           # High-precedence vars (main.yml)
    ├── defaults/       # Low-precedence vars (main.yml)
    ├── meta/           # Dependencies & metadata (main.yml)
    ├── library/        # Custom modules (standalone roles)
    ├── module_utils/   # Custom module utils
    └── lookup_plugins/ # Custom lookups
```
- Default entry: `main.yml` or `main.yaml`.
- `vars/` & `defaults/` can be dirs (files loaded alphabetically).
- Set `roles_path` in `ansible.cfg` for custom locations (default: `~/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles`).

## Best Practices
- Use OS-specific task files (e.g., `redhat.yml`) in `tasks/main.yml` with conditionals.
- Embed custom modules/plugins in subdirs for standalone roles.
- Tag roles/tasks for selective execution; tag `pre_tasks`, `post_tasks`, & dependencies.
- Use `meta/argument_specs.yml` for param validation (since 2.11).
- Avoid var scope leakage: Use `vars` under role; in ansible-core 2.15+, vars are private by default (`DEFAULT_PRIVATE_ROLE_VARS`).
- Allow duplicates in `meta/main.yml` for multiple runs.

## Tasks
- Define in `tasks/main.yml`; import others for modularity.
```yaml
# tasks/main.yml
- name: Install for RedHat
  import_tasks: redhat.yml
  when: ansible_facts['os_family']|lower == 'redhat'

# tasks/redhat.yml
- name: Install httpd
  ansible.builtin.yum:
    name: httpd
    state: present
```
- Use `tasks_from: other.yml` in role calls to override entry point.

## Handlers
- Define in `handlers/main.yml`; notify from tasks to trigger.
```yaml
# handlers/main.yml
- name: Restart httpd
  ansible.builtin.service:
    name: httpd
    state: restarted
```
- Handlers imported to play scope; run at end of play (or on flush).

## Variables & Defaults
- `vars/main.yml`: High precedence (play scope by default).
- `defaults/main.yml`: Low precedence (easily overridden).
```yaml
# defaults/main.yml
web_port: 80
```
- Pass via `vars` in role calls; use `public: no` to skip import.

## Files & Templates
- `files/`: Static content for `copy`/`script`.
  - e.g., Copy `bar.txt` with `ansible.builtin.copy: src: bar.txt dest: /path`.
- `templates/`: Jinja2 for `template`.
  - e.g., `ntp.conf.j2` with `{{ web_port }}`.

## Meta
- Define dependencies & flags in `meta/main.yml`.
```yaml
# meta/main.yml
dependencies:
  - role: common
    vars:
      param: 3
allow_duplicates: true  # For multiple runs in one play
galaxy_info:
  author: your_name
  description: Role desc
  min_ansible_version: 2.9
```

## Role Argument Validation (Since 2.11)
- In `meta/argument_specs.yml`; validates params before run (tagged `always`).
```yaml
# meta/argument_specs.yml
argument_specs:
  main:
    options:
      my_int:
        type: int
        required: false
        default: 42
        description: Integer param
```

## Using Roles in Playbooks

### At Play Level (Static)
- In `roles:` section; runs after `pre_tasks`, before `tasks`.
```yaml
- hosts: webservers
  roles:
    - common
    - role: app
      vars:
        dir: /opt/a
        port: 5000
      tags: deploy
```
- Tags apply to all role tasks.

### include_role (Dynamic)
- In `tasks:`; runtime evaluation, runs in task order.
```yaml
- hosts: webservers
  tasks:
    - name: Include app
      include_role:
        name: app
        vars:
          dir: /opt/a
          port: 5000
        tags: deploy
      when: ansible_facts['os_family'] == 'RedHat'
```
- Tags apply only to include; use `--tags` for role tasks.

### import_role (Static)
- In `tasks:`; parse-time, like play-level.
```yaml
- hosts: webservers
  tasks:
    - name: Import app
      import_role:
        name: app
        vars:
          dir: /opt/a
          port: 5000
        tags: deploy
```
- Tags apply to all role tasks.

## Key Differences
| Aspect          | Play-Level / import_role | include_role     |
|-----------------|---------------------------|------------------|
| Timing         | Static (parse-time)      | Dynamic (runtime)|
| Order          | Before play tasks        | In task sequence |
| Tag Scope      | All role tasks           | Only include     |
| Conditionals   | Limited                  | Full support     |

For more, see Ansible Galaxy for publishing roles.
