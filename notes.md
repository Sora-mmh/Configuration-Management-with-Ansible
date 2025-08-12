# Configuration Management with Ansible 

Ansible is an **agentless, YAML-based automation tool** used to configure systems, deploy software, and orchestrate complex workflows across many servers from a central control node.

***

## 1. Ansible Overview

### Key Use Cases
- Repetitive admin: updates, backups, user creation, permission assignment, reboots
- Ensuring identical configuration across multiple servers
- Supporting full stack management from OS to cloud resources

### Advantages
- No need to SSH into each remote host manually
- Store config/installation/deployment tasks in a **single YAML file**
- Reuse automation for multiple environments
- **Idempotency**: repeat execution leads to the same result
- Works across OS, virtualized/cloud infrastructure

**Agentless**: uses SSH, no extra agent on managed hosts.

***

## 2. Core Concepts

### Modules
- Small, reusable scripts handling a single task (create/copy file, install package, start service, launch Docker container, create cloud VM, etc.).
- Copied to and executed on target, then removed.

### Playbooks
- Group multiple modules/tasks into *plays* executed in order.
- YAML files describing which tasks run on which hosts and in what order.
- Each **play** = target hosts + tasks + (optional) vars.

**Example**:  
```yaml
- name: Install and start nginx
  hosts: webservers
  remote_user: root
  tasks:
    - name: Create nginx dir
      file:
        path: /opt/nginx
        state: directory
    - name: Install nginx latest version
      yum:
        name: nginx
        state: latest
    - name: Start nginx
      service:
        name: nginx
        state: started
```

### Inventory
- Maps **groups** (`[groupname]`) to hostnames/IPs.
- Can define vars per group or per host.
```ini
[webservers]
web1.example.com
web2.example.com

[webservers:vars]
ansible_ssh_private_key_file=~/.ssh/id_ed25519
ansible_user=root
```

**Groups** can be functional (db, web), geographic (region), or environmental (stage, prod). Hosts can be in multiple groups.

***

## 3. Installing Ansible

**Mac**:
```bash
brew update
brew install ansible
```
**Python pip**:
```bash
pip install ansible
```

📄 Docs: [Installation Guide](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

**Control Node**: Linux/Unix (Windows not supported).

***

## 4. Ad-hoc Commands

Syntax:
```bash
ansible  -i  -m  -a ""
```
Examples:
```bash
ansible 192.168.10.5 -i hosts -m ping
ansible webservers -i hosts -m ping
ansible all -i hosts -m ping
```

***

## 5. Managing AWS EC2 with Ansible

Example `inventory`:
```ini
[ec2]
ec2-1-2-3-4.compute.amazonaws.com
ec2-5-6-7-8.compute.amazonaws.com

[ec2:vars]
ansible_ssh_private_key_file=~/.ssh/ec2-key.pem
ansible_user=ec2-user
ansible_python_interpreter=/usr/bin/python3.9
```
- SSH into EC2 instances once manually to accept fingerprints or disable host key checking (see below).

***

## 6. SSH Host Key Checking

- **Long-lived hosts**: add fingerprints using `ssh-keyscan` and authorized keys with `ssh-copy-id`.
- **Ephemeral hosts**: disable host key checking in `~/.ansible.cfg`:
```ini
[defaults]
host_key_checking=False
```
📄 [Config Docs](https://docs.ansible.com/ansible-core/2.15/reference_appendices/config.html)

***

## 7. Playbooks in Practice

### Example: Configure nginx
`ansible.cfg`:
```ini
[defaults]
inventory=./hosts
host_key_checking=False
```

Playbook:
```yaml
- name: Configure nginx web server
  hosts: webserver
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: latest
    - name: Start nginx
      service:
        name: nginx
        state: started
```

Run:
```bash
ansible-playbook simple-playbook.yaml
```
**Idempotency**: running again without changes → `changed=0`.

***

## 8. Modules Reference

- Built-in: [Module Index](https://docs.ansible.com/ansible/latest/collections/index_module.html)
- Grouped by category: [By Category](https://docs.ansible.com/ansible/2.9/modules/modules_by_category.html)

***

## 9. Collections

- Packaged playbooks, modules, plugins.
- Built-in: `ansible.builtin`
- External: [Ansible Galaxy](https://galaxy.ansible.com/)
```bash
ansible-galaxy collection list
ansible-galaxy collection install amazon.aws
```
- Create your own collections for large projects.

***

## 10. Variables in Ansible

### Defining in Playbook
```yaml
vars:
  version: 1.0.0
  user_home: /home/demo
```

### Registered Variables
Capture command output:
```yaml
- shell: ps aux | grep node
  register: app_status
- debug:
    msg: "{{ app_status.stdout_lines }}"
```

### Passing with CLI
```bash
ansible-playbook playbook.yaml -e "version=1.0.0 user_home=/home/demo"
```

### Variables from File
```yaml
vars_files:
  - project-vars
```

***

## 11. Ansible Roles

Purpose: structure, modularity, reuse.

**Directory Structure**:
```
roles/
  role_name/
    tasks/       # mandatory
    handlers/
    files/
    templates/
    vars/
    defaults/
    meta/
```
- Roles have default vars that can be overridden.
- Can be pulled from [Ansible Galaxy](https://galaxy.ansible.com/) or GitHub.

Usage:
```yaml
- hosts: all
  roles:
    - role_name
```
📄 Docs: [Using Roles](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html)

***

## 12. Best Practices

- Keep Ansible configs/playbooks in Git.
- Use **inventory groups** to target roles/playbooks efficiently.
- Prefer vars files/extra-vars over hardcoding.
- Leverage idempotent modules; avoid unnecessary `shell` calls.
- Use roles/collections for large codebases.
- Disable host key checking only for ephemeral infra.
