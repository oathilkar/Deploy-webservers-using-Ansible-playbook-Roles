# Deploy-webservers-using-Ansible-playbook-Roles

This project is used for;
Deploying web servers using Ansible involves setting up a project structure with playbooks, roles, and configuration files. 
Below, I'll guide you through creating a project to deploy Nginx and Apache HTTP Server using Ansible playbooks.

### Project Structure

Create the following directory structure for your Ansible project:

```
ansible-webserver-deployment/
├── inventories/
│   └── production/
│       └── hosts.ini       # Inventory file
├── playbooks/
│   ├── nginx-install.yml   # Nginx playbook
│   └── apache-install.yml  # Apache playbook
└── roles/
    ├── nginx/
    │   ├── tasks/
    │   │   └── main.yml    # Nginx role tasks
    │   ├── templates/
    │   │   └── nginx.conf.j2  # Nginx configuration template
    │   └── meta/
    │       └── main.yml    # Nginx role metadata
    └── apache/
        ├── tasks/
        │   └── main.yml    # Apache role tasks
        ├── templates/
        │   └── httpd.conf.j2  # Apache configuration template
        └── meta/
            └── main.yml    # Apache role metadata
```

### Step-by-Step Guide

#### 1. Inventory File

Create an inventory file (`hosts.ini`) under `inventories/production/`:

```ini
[webservers]
server1 ansible_host=192.168.1.10
server2 ansible_host=192.168.1.11
```

#### 2. Nginx Playbook

Create a playbook (`nginx-install.yml`) under `playbooks/`:

```yaml
---
- name: Install and configure Nginx
  hosts: webservers
  become: true  # Run tasks with root privileges

  roles:
    - nginx
```

#### 3. Apache Playbook

Create a playbook (`apache-install.yml`) under `playbooks/`:

```yaml
---
- name: Install and configure Apache HTTP Server
  hosts: webservers
  become: true  # Run tasks with root privileges

  roles:
    - apache
```

#### 4. Nginx Role

Create an Ansible role for Nginx under `roles/nginx/`:

**`roles/nginx/meta/main.yml`**:

```yaml
---
dependencies: []
```

**`roles/nginx/tasks/main.yml`**:

```yaml
---
- name: Install Nginx
  apt:
    name: nginx
    state: present
  tags: [nginx]

- name: Copy Nginx configuration file
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: restart nginx
  tags: [nginx]

- name: Ensure Nginx service is enabled and started
  systemd:
    name: nginx
    enabled: yes
    state: started
  tags: [nginx]

- name: Enable Nginx service on system boot
  systemd:
    name: nginx
    enabled: yes
    daemon_reload: yes
  tags: [nginx]

- name: Allow Nginx through the firewall
  ufw:
    rule: allow
    name: 'Nginx Full'
    state: enabled
  tags: [nginx]

- name: Reload ufw
  ufw:
    state: reloaded
  tags: [nginx]

handlers:
  - name: restart nginx
    service:
      name: nginx
      state: restarted
```

**`roles/nginx/templates/nginx.conf.j2`**:

```nginx
# Nginx configuration file
user www-data;
worker_processes auto;
pid /run/nginx.pid;

events {
    worker_connections 768;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;

    keepalive_timeout 65;
    types_hash_max_size 2048;

    include /etc/nginx/conf.d/*.conf;

    server {
        listen 80;
        server_name {{ ansible_hostname }};

        location / {
            root /usr/share/nginx/html;
            index index.html index.htm;
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
            root /usr/share/nginx/html;
        }
    }
}
```

#### 5. Apache Role

Create an Ansible role for Apache under `roles/apache/`:

**`roles/apache/meta/main.yml`**:

```yaml
---
dependencies: []
```

**`roles/apache/tasks/main.yml`**:

```yaml
---
- name: Install Apache HTTP Server
  apt:
    name: apache2
    state: present
  tags: [apache]

- name: Copy Apache configuration file
  template:
    src: httpd.conf.j2
    dest: /etc/apache2/apache2.conf
  notify: restart apache
  tags: [apache]

- name: Ensure Apache service is enabled and started
  systemd:
    name: apache2
    enabled: yes
    state: started
  tags: [apache]

- name: Enable Apache service on system boot
  systemd:
    name: apache2
    enabled: yes
    daemon_reload: yes
  tags: [apache]

- name: Allow Apache through the firewall
  ufw:
    rule: allow
    name: 'Apache Full'
    state: enabled
  tags: [apache]

- name: Reload ufw
  ufw:
    state: reloaded
  tags: [apache]

handlers:
  - name: restart apache
    service:
      name: apache2
      state: restarted
```

**`roles/apache/templates/httpd.conf.j2`**:

```apacheconf
# Apache configuration file
ServerName {{ ansible_hostname }}
```

### Running the Playbooks

To deploy Nginx and Apache HTTP Server to your servers, run the following commands:

```bash
# Deploy Nginx
ansible-playbook -i inventories/production/hosts.ini playbooks/nginx-install.yml

# Deploy Apache HTTP Server
ansible-playbook -i inventories/production/hosts.ini playbooks/apache-install.yml
```

### Summary

- **Project Structure**: Organize your project with inventories, playbooks, and roles.
- **Ansible Playbooks**: Define tasks to install and configure Nginx and Apache HTTP Server.
- **Roles**: Use roles to encapsulate tasks and templates for Nginx and Apache HTTP Server.
- **Templates**: Use Jinja2 templates for configuring Nginx (`nginx.conf.j2`) and Apache (`httpd.conf.j2`).

This setup provides a flexible and scalable approach to deploying web servers using Ansible. 
Customize the configurations and tasks as needed for your environment and specific requirements.
