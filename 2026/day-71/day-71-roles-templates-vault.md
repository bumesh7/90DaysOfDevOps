### Task 1: Jinja2 Templates
Templates let you generate config files dynamically using variables and facts.

1. Create `templates/nginx-vhost.conf.j2`:
```
mkdir templates
cd templates
vim nginx-vhost.conf.j2
```
```jinja2
# Managed by Ansible -- do not edit manually
server {
    listen {{ http_port | default(80) }};
    server_name {{ ansible_facts['hostname'] }};

    root /var/www/{{ app_name }};
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    access_log /var/log/nginx/{{ app_name }}_access.log;
    error_log /var/log/nginx/{{ app_name }}_error.log;
}
```

2. Create a playbook `template-demo.yml`:
```yaml
- name: Deploy Nginx with template
  hosts: web
  become: true
  vars:
    app_name: terraweek-app
    http_port: 80

  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Create web root
      file:
        path: "/var/www/{{ app_name }}"
        state: directory
        mode: '0755'

    - name: Deploy vhost config from template
      template:
        src: /home/ubuntu/ansible-practice/playbooks/templates/nginx-vhost.conf.j2
        dest: "/etc/nginx/conf.d/{{ app_name }}.conf"
        owner: root
        mode: '0644'
      notify: Restart Nginx

    - name: Deploy index page
      copy:
        content: "<h1>{{ app_name }}</h1><p>Host: {{ ansible_facts['hostname'] }} | IP: {{ ansible_facts['default_ipv4']['address'] }}</p>"
        dest: "/var/www/{{ app_name }}/index.html"

  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

Run it with `--diff` to see the rendered template:
```bash
ansible-playbook -i ../inventory.ini template-demo.yml --diff
```

<img width="1659" height="944" alt="image" src="https://github.com/user-attachments/assets/b7c14a29-a689-45c7-a68e-6cd3e2249b77" />


**Verify:** SSH into the web server and read the generated config. Are the variables replaced with actual values?
```
cat /var/www/terraweek-app/index.html

<h1>terraweek-app</h1><p>Host: ip-172-31-37-27 | IP: 172.31.37.27</p>
```
<img width="1039" height="73" alt="image" src="https://github.com/user-attachments/assets/f6c60106-ee1e-4aa3-bcf9-ec5936a4ba9a" />

---

### Task 2: Understand the Role Structure
An Ansible role has a fixed directory structure. Each directory has a specific purpose:

```
roles/
  webserver/
    tasks/
      main.yml         # The main task list
    handlers/
      main.yml         # Handlers (restart services, etc.)
    templates/
      nginx.conf.j2    # Jinja2 templates
    files/
      index.html       # Static files to copy
    vars/
      main.yml         # Role variables (high priority)
    defaults/
      main.yml         # Default variables (low priority, easily overridden)
    meta/
      main.yml         # Role metadata and dependencies
```

Every directory contains a `main.yml` that Ansible loads automatically. You only create the directories you need.

Generate a skeleton with:
```bash
ansible-galaxy init roles/webserver
```

Explore the generated directory. Read the README.md that Galaxy creates.

**Document:** What is the difference between `vars/main.yml` and `defaults/main.yml`?
```
defaults/main.yml

Contains default values
Easily overridden by:
group_vars
host_vars
playbook variables
extra vars (-e)
Used for configurable settings

example:
http_port: 80
app_name: my-app

vars/main.yml

Contains fixed/internal variables
Hard to override (higher precedence than most variables)
Used for internal logic or constants

example:
nginx_package: nginx
config_path: /etc/nginx/nginx.conf

```
---

### Task 3: Build a Custom Webserver Role
Build a complete `webserver` role from scratch:

**`roles/webserver/defaults/main.yml`:**
```yaml
---
http_port: 80
app_name: myapp
max_connections: 512
```

**`roles/webserver/tasks/main.yml`:**
```yaml
---
- name: Install Nginx
  apt:
    name: nginx
    state: present

- name: Remove old nginx config
  file:
    path: /etc/nginx/conf.d/terraweek-app.conf
    state: absent
  notify: Restart Nginx

- name: Deploy Nginx config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    owner: root
    mode: '0644'
  notify: Restart Nginx

- name: Deploy vhost config
  template:
    src: vhost.conf.j2
    dest: "/etc/nginx/conf.d/{{ app_name }}.conf"
    owner: root
    mode: '0644'
  notify: Restart Nginx

- name: Create web root
  file:
    path: "/var/www/{{ app_name }}"
    state: directory
    mode: '0755'

- name: Deploy index page
  template:
    src: index.html.j2
    dest: "/var/www/{{ app_name }}/index.html"
    mode: '0644'

- name: Start and enable Nginx
  service:
    name: nginx
    state: started
    enabled: true
```

**`roles/webserver/handlers/main.yml`:**
```yaml
---
- name: Restart Nginx
  service:
    name: nginx
    state: restarted
```

**`roles/webserver/templates/index.html.j2`:**
```html
<h1>{{ app_name }}</h1>
<p>Server: {{ ansible_hostname }}</p>
<p>IP: {{ ansible_default_ipv4.address }}</p>
<p>Environment: {{ app_env | default('development') }}</p>
<p>Managed by Ansible</p>
```

Create the `vhost.conf.j2` and `nginx.conf.j2` templates yourself based on what you learned in Task 1.

vim roles/webserver/templates/index.html.j2
```
<h1>{{ app_name }}</h1>
<p>Server: {{ ansible_facts['hostname'] }}</p>
<p>IP: {{ ansible_facts['default_ipv4']['address'] }}</p>
<p>Environment: {{ app_env | default('development') }}</p>
<p>Managed by Ansible</p>
```
```
vim roles/webserver/templates/vhost.conf.j2

server {
    listen {{ http_port }};
    server_name {{ ansible_facts['hostname'] }};

    root /var/www/{{ app_name }};
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    access_log /var/log/nginx/{{ app_name }}_access.log;
    error_log /var/log/nginx/{{ app_name }}_error.log;
}
```
vim roles/webserver/templates/nginx.conf.j2
```
user www-data;
worker_processes auto;
pid /run/nginx.pid;

events {
    worker_connections {{ max_connections }};
}

http {
    sendfile on;
    tcp_nopush on;
    keepalive_timeout 65;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    include /etc/nginx/conf.d/*.conf;
}
```

Now call the role from a playbook `site-roles.yml`:
```yaml
---
- name: Configure web servers
  hosts: web
  become: true
  roles:
    - role: webserver
      vars:
        app_name: terraweek
        http_port: 80
```

Run it:
```bash
ansible-playbook site.yml
```

**Verify:** Curl the web server. Does the custom page load?
```
curl http://43.204.24.81

<h1>terraweek</h1>
<p>Server: ip-172-31-37-27</p>
<p>IP: 172.31.37.27</p>
<p>Environment: development</p>
<p>Managed by Ansible</p>
```
<img width="1409" height="735" alt="image" src="https://github.com/user-attachments/assets/d492a2e0-b896-4b43-a8ba-946b4b3b9519" />

---

### Task 4: Ansible Galaxy -- Use Community Roles
Ansible Galaxy is a marketplace of pre-built roles.

1. **Search for roles:**
```bash
ansible-galaxy search nginx --platforms EL
ansible-galaxy search mysql
```
<img width="1478" height="993" alt="image" src="https://github.com/user-attachments/assets/488f50e1-cefd-4d14-8d97-4fabb87f0ac0" />


2. **Install a role from Galaxy:**
```bash
ansible-galaxy install geerlingguy.docker
```

3. **Check where it was installed:**
```bash
ansible-galaxy list
```
<img width="1108" height="237" alt="image" src="https://github.com/user-attachments/assets/b53e9af8-0de7-424c-acd0-4cecaf631e36" />

4. **Use the installed role** -- create `docker-setup.yml`:
```yaml
---
- name: Install Docker using Galaxy role
  hosts: app
  become: true
  roles:
    - geerlingguy.docker
```

Run it -- Docker gets installed with a single role call.
<img width="1187" height="739" alt="image" src="https://github.com/user-attachments/assets/132f63d1-0a91-4231-bb44-f036ba5bfd46" />

5. **Use a requirements file** for managing multiple roles. Create `requirements.yml`:
```yaml
---
roles:
  - name: geerlingguy.docker
    version: "7.4.1"
  - name: geerlingguy.ntp
```

Install all at once:
```bash
ansible-galaxy install -r requirements.yml
```
<img width="1149" height="182" alt="image" src="https://github.com/user-attachments/assets/b366b1b0-7c01-4d97-a806-bb550b177fa4" />

**Document:** Why use a `requirements.yml` instead of installing roles manually?
```
Ensures same versions of roles across all environments
All dependencies listed in one file
Easier to update, track, and review

one command install everything = ansible-galaxy install -r requirements.yml
```
---

### Task 5: Ansible Vault -- Encrypt Secrets
Never put passwords, API keys, or tokens in plain text. Ansible Vault encrypts sensitive data.

1. **Create an encrypted file:**
```bash
ansible-vault create group_vars/db/vault.yml
```
It will ask for a vault password, then open an editor. Add:
```yaml
vault_db_password: SuperSecretP@ssw0rd
vault_db_root_password: R00tP@ssw0rd123
vault_api_key: sk-abc123xyz789
```
Save and exit. Open the file with `cat` -- it is fully encrypted.

<img width="1158" height="292" alt="image" src="https://github.com/user-attachments/assets/65ebfdea-98ec-4a53-b3b7-96d173b07edf" />

2. **Edit an encrypted file:**
```bash
ansible-vault edit group_vars/db/vault.yml
```

3. **View without editing:**
```bash
ansible-vault view group_vars/db/vault.yml
```
<img width="1151" height="151" alt="image" src="https://github.com/user-attachments/assets/0124de2c-4679-4432-91b0-f92732e914d6" />

4. **Encrypt an existing file:**
```bash
ansible-vault encrypt group_vars/db/secrets.yml
```

5. **Use vault variables in a playbook** -- create `db-setup.yml`:
```yaml
---
- name: Configure database
  hosts: db
  become: true

  tasks:
    - name: Show DB password (never do this in production)
      debug:
        msg: "DB password is set: {{ vault_db_password | length > 0 }}"
```

Run with the vault password:
```bash
ansible-playbook -i ../nventory.ini db-setup.yml --ask-vault-pass
```
<img width="1200" height="270" alt="image" src="https://github.com/user-attachments/assets/df78fbfb-fac7-48bd-bb2b-51f76edab414" />

6. **Use a password file** (better for CI/CD):
```bash
echo "YourVaultPassword" > .vault_pass
chmod 600 .vault_pass
echo ".vault_pass" >> .gitignore

ansible-playbook db-setup.yml --vault-password-file .vault_pass
```

Or set it in `ansible.cfg`:
```ini
[defaults]
vault_password_file = .vault_pass
```

```
cd ~/ansible-practice/playbooks

ls -l .vault_pass
echo "YourVaultPassword" > .vault_pass
chmod 600 .vault_pass

ansible-vault encrypt group_vars/db/vault.yml --vault-password-file .vault_pass

cat group_vars/db/vault.yml

ansible-vault view group_vars/db/vault.yml --vault-password-file .vault_pass

ansible-playbook -i ../inventory.ini db-setup.yml --vault-password-file .vault_pass
```
<img width="1313" height="571" alt="image" src="https://github.com/user-attachments/assets/17decee9-e2a7-4969-bdcb-6bb4db9f5266" />

**Document:** Why is `--vault-password-file` better than `--ask-vault-pass` for automated pipelines?
```
--ask-vault-pass prompts for input
CI/CD systems (Jenkins, GitHub Actions, GitLab CI, etc.) cannot type passwords
```
---

### Task 6: Combine Roles, Templates, and Vault
Write a complete `site.yml` that uses everything you learned today:
```
playbooks/
├── site.yml
├── templates/
│   └── db-config.j2
├── group_vars/
│   └── db/
│       └── vault.yml   (encrypted)
├── .vault_pass
```

vim site-1.yml

```yaml
---
- name: Configure web servers
  hosts: web
  become: true
  roles:
    - role: webserver
      vars:
        app_name: terraweek
        http_port: 80

- name: Configure app servers with Docker
  hosts: app
  become: true
  roles:
    - geerlingguy.docker

- name: Configure database servers
  hosts: db
  become: true
  tasks:
    - name: Create DB config with secrets
      template:
        src: templates/db-config.j2
        dest: /etc/db-config.env
        owner: root
        mode: '0600'
```

Create `templates/db-config.j2`:
```jinja2
# Database Configuration -- Managed by Ansible
DB_HOST={{ ansible_default_ipv4.address }}
DB_PORT={{ db_port | default(3306) }}
DB_PASSWORD={{ vault_db_password }}
DB_ROOT_PASSWORD={{ vault_db_root_password }}
```

Run:
```bash
ansible-playbook site-1.yml
```
use only one server at a time 
```
ansible-playbook -i ../inventory.ini site-1.yml \
  --limit db \
  --vault-password-file .vault_pass
```
<img width="1828" height="682" alt="image" src="https://github.com/user-attachments/assets/a518aca2-9736-4933-ade4-4cd76afe5443" />

**Verify:** SSH into the db server and check `/etc/db-config.env`. Are the secrets rendered correctly? Is the file permission `600`?
```
ssh -i /home/ubuntu/ansible-key ubuntu@43.204.24.81

sudo ls -l /etc/db-config.env

sudo cat /etc/db-config.env
```
<img width="835" height="172" alt="image" src="https://github.com/user-attachments/assets/bcc95dc2-2c83-45e9-8dc9-a7a4c533d5a3" />

---
