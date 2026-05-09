### Task 1: Variables in Playbooks
Create `variables-demo.yml`:

```yaml
---
- name: Variable demo
  hosts: all
  become: true

  vars:
    app_name: terraweek-app
    app_port: 8080
    app_dir: "/opt/{{ app_name }}"
    packages:
      - git
      - curl
      - wget

  tasks:
    - name: Print app details
      debug:
        msg: "Deploying {{ app_name }} on port {{ app_port }} to {{ app_dir }}"

    - name: Create application directory
      file:
        path: "{{ app_dir }}"
        state: directory
        mode: '0755'

    - name: Install required packages
      apt:
        name: "{{ packages }}"
        state: present
```

Run it and verify the variables resolve correctly.
```
ansible-playbook -i hosts.ini variables-demo.yml
```
<img width="1185" height="457" alt="image" src="https://github.com/user-attachments/assets/61b25484-69d7-45c9-8f48-28ae0dca88ed" />

Now, override a variable from the command line:
```bash
ansible-playbook -i hosts.ini variables-demo.yml -e "app_name=my-custom-app app_port=9090"
```
<img width="1147" height="372" alt="image" src="https://github.com/user-attachments/assets/cc9bd954-4bac-423a-b5df-29ecab92013d" />

**Verify:** Does the CLI variable override the playbook variable?
```
Yes the CLI variavle can override the playbook variable
```
---

### Task 2: group_vars and host_vars
Variables should not live inside playbooks. Move them to dedicated files.

Create this structure:
```
ansible-practice/
  inventory.ini
  ansible.cfg
  group_vars/
    all.yml
    web.yml
    db.yml
  host_vars/
    web-server.yml
  playbooks/
    site.yml
```

**`group_vars/all.yml`** -- applies to every host:
```yaml
---
ntp_server: pool.ntp.org
app_env: development
common_packages:
  - vim
  - htop
  - tree
```

**`group_vars/web.yml`** -- applies only to the web group:
```yaml
---
http_port: 80
max_connections: 1000
web_packages:
  - nginx
```

**`group_vars/db.yml`** -- applies only to the db group:
```yaml
---
db_port: 3306
db_packages:
  - mysql-server
```

**`host_vars/web-server.yml`** -- applies only to this specific host:
```yaml
---
max_connections: 2000
custom_message: "This is the primary web server"
```

Write a playbook `site.yml` that uses these variables:
```yaml
---
- name: Apply common config
  hosts: all
  become: true
  tasks:
    - name: Install common packages
      yum:
        name: "{{ common_packages }}"
        state: present
    - name: Show environment
      debug:
        msg: "Environment: {{ app_env }}"

- name: Configure web servers
  hosts: web
  become: true
  tasks:
    - name: Show web config
      debug:
        msg: "HTTP port: {{ http_port }}, Max connections: {{ max_connections }}"
    - name: Show host-specific message
      debug:
        msg: "{{ custom_message }}"
```

Run it and observe which variables apply to which hosts.
```
mkdir -p ansible-practice/{group_vars,host_vars,playbooks}
cd ansible-practice

touch inventory.ini ansible.cfg
touch group_vars/all.yml group_vars/web.yml group_vars/db.yml
touch host_vars/worker-node.yml
touch playbooks/site.yml
```
**Document:** What is the variable precedence? (hint: host_vars > group_vars > playbook vars, and `-e` overrides everything)
```
Variable precedence defines which value Ansible will use when the same variable is defined in multiple places.

Order (Lowest → Highest Priority)

1. group_vars/all
2. group_vars/<group>
3. host_vars/<host>
4. playbook vars
5. extra vars (-e)  ← highest priority

andible-playbook -i ../inventory.ini site.yml
```
<img width="1101" height="559" alt="image" src="https://github.com/user-attachments/assets/59969890-5141-4328-938e-0883caf8a809" />

---

### Task 3: Ansible Facts -- Gathering System Information
Ansible automatically collects "facts" about each managed node -- OS, IP, memory, CPU, disks, and hundreds more.

1. **See all facts for a host:**
```bash
ansible web-server -m setup

or

ansible -i inventory.ini web -m setup
```

2. **Filter specific facts:**
```bash
ansible web-server -m setup -a "filter=ansible_os_family"
ansible web-server -m setup -a "filter=ansible_distribution*"
ansible web-server -m setup -a "filter=ansible_memtotal_mb"
ansible web-server -m setup -a "filter=ansible_default_ipv4"
```
<img width="1054" height="731" alt="image" src="https://github.com/user-attachments/assets/57ba5c41-78e6-40cf-ae6a-6e1c4b235ca0" />

3. **Use facts in a playbook** -- create `facts-demo.yml`:
```yaml
---
- name: Facts demo
  hosts: all
  tasks:
    - name: Show OS info
      debug:
        msg: >
          Hostname: {{ ansible_facts['hostname'] }},
          OS: {{ ansible_facts['distribution'] }} {{ ansible_facts['distribution_version'] }},
          RAM: {{ ansible_facts['memtotal_mb'] }}MB,
          IP: {{ ansible_facts['default_ipv4']['address'] }}

    - name: Show all network interfaces
      debug:
        var: ansible_facts['interfaces']
```

Run it and observe the facts printed for each host.
```
ansible-playbook -i ../inventory.ini facts-demo.yml
```
<img width="1116" height="420" alt="image" src="https://github.com/user-attachments/assets/310e5087-1aa3-44c8-aa97-c94beb807e45" />

**Document:** Name five facts you would use in real playbooks and why.

---

### Task 4: Conditionals with when
Tasks should not always run on every host. Use `when` to control execution.

Create `conditional-demo.yml`:

```yaml
---
- name: Conditional tasks demo
  hosts: all
  become: true

  tasks:
    - name: Install Nginx (only on web servers)
      apt:
        name: nginx
        state: present
        update_cache: yes
      when: "'web' in group_names"

    - name: Install MySQL (only on db servers)
      apt:
        name: mysql-server
        state: present
        update_cache: yes
      when: "'db' in group_names"

    - name: Show warning on low memory hosts
      debug:
        msg: "WARNING: This host has less than 1GB RAM"
      when: ansible_facts['memtotal_mb'] < 1024

    - name: Run only on Amazon Linux
      debug:
        msg: "This is an Amazon Linux machine"
      when: ansible_facts['distribution'] == "Amazon"

    - name: Run only on Ubuntu
      debug:
        msg: "This is an Ubuntu machine"
      when: ansible_facts['distribution'] == "Ubuntu"

    - name: Run only in production
      debug:
        msg: "Production settings applied"
      when: app_env == "production"

    - name: Multiple conditions (AND)
      debug:
        msg: "Web server with enough memory"
      when:
        - "'web' in group_names"
        - ansible_facts['memtotal_mb'] >= 512

    - name: OR condition
      debug:
        msg: "Either web or app server"
      when: "'web' in group_names or 'app' in group_names"
```

Run it and observe which tasks are skipped on which hosts.
<img width="1352" height="746" alt="image" src="https://github.com/user-attachments/assets/b86a30db-d526-48d3-b29b-dc43bfe27e2a" />

**Verify:** Are tasks correctly skipping on hosts that don't match the condition?
```
ansible-playbook -i ../inventory.ini condition-demo.yml

Yes skipping correctly
```
---

### Task 5: Loops
Create `loops-demo.yml`:

```yaml
---
- name: Loops demo
  hosts: all
  become: true

  vars:
    users:
      - name: deploy
        groups: sudo
      - name: monitor
        groups: sudo
      - name: appuser
        groups: users

    directories:
      - /opt/app/logs
      - /opt/app/config
      - /opt/app/data
      - /opt/app/tmp

  tasks:
    - name: Create multiple users
      user:
        name: "{{ item.name }}"
        groups: "{{ item.groups }}"
        state: present
      loop: "{{ users }}"

    - name: Create multiple directories
      file:
        path: "{{ item }}"
        state: directory
        mode: '0755'
      loop: "{{ directories }}"

    - name: Install multiple packages
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - git
        - curl
        - unzip
        - jq

    - name: Print each user created
      debug:
        msg: "Created user {{ item.name }} in group {{ item.groups }}"
      loop: "{{ users }}"
```

Run it and observe the loop output -- each iteration is shown separately.
```
ansible-playbook -i ../inventory.ini loop-demo.yml
```
<img width="1121" height="651" alt="image" src="https://github.com/user-attachments/assets/2c54cc17-62d7-4edb-868c-a7cef6399de2" />

**Document:** What is the difference between `loop` and the older `with_items`? (hint: `loop` is the modern recommended syntax)
```
Use loop for all new playbooks — it’s cleaner, more powerful, and future-proof.

Feature	with_items	loop
Status	Deprecated / legacy	Recommended
Syntax	Multiple with_* variants	Single unified syntax
Readability	Less consistent	Cleaner and clearer
```
---

### Task 6: Register, Debug, and Combine Everything
Build a real-world playbook `server-report.yml` that combines variables, facts, conditionals, and register:

```yaml
---
- name: Server Health Report
  hosts: all

  tasks:
    - name: Check disk space
      command: df -h /
      register: disk_result

    - name: Check memory
      command: free -m
      register: memory_result

    - name: Check running services
      shell: systemctl list-units --type=service --state=running | head -20
      register: services_result

    - name: Generate report
      debug:
        msg:
          - "========== {{ inventory_hostname }} =========="
          - "OS: {{ ansible_facts['distribution'] }} {{ ansible_facts['distribution_version'] }}"
          - "IP: {{ ansible_facts['default_ipv4']['address'] }}"
          - "RAM: {{ ansible_facts['memtotal_mb'] }}MB"
          - "Disk: {{ disk_result.stdout_lines[1] }}"
          - "Running services (first 20): {{ services_result.stdout_lines | length }}"

    - name: Flag if disk is critically low
      debug:
        msg: "ALERT: Check disk space on {{ inventory_hostname }}"
      when: disk_result.stdout is search('9[0-9]%|100%')

    - name: Save report to file
      copy:
        content: |
          Server: {{ inventory_hostname }}
          OS: {{ ansible_facts['distribution'] }} {{ ansible_facts['distribution_version'] }}
          IP: {{ ansible_facts['default_ipv4']['address'] }}
          RAM: {{ ansible_facts['memtotal_mb'] }}MB
          Disk: {{ disk_result.stdout }}
          Checked at: {{ ansible_facts['date_time']['iso8601'] }}
        dest: "/tmp/server-report-{{ inventory_hostname }}.txt"
      become: true
```

Run it and verify the report file is created on each server.
```
ansible-playbook -i ../inventory.ini server-report.yml 
```
<img width="1121" height="651" alt="image" src="https://github.com/user-attachments/assets/bbd67d09-c762-4a1c-a838-07f073f22ef3" />

**Verify:** SSH into a server and read `/tmp/server-report-*.txt`. Does it contain accurate information?
<img width="1421" height="244" alt="image" src="https://github.com/user-attachments/assets/01bece69-a40a-423c-b876-8ad942c11057" />

```
cd tmp
ls
cat server-report-worker-node.txt

Server: worker-node
OS: Ubuntu 26.04
IP: 172.31.37.27
RAM: 908MB
Disk: Filesystem      Size  Used Avail Use% Mounted on
/dev/root        19G  3.0G   16G  17% /
Checked at: 2026-05-09T10:50:48Z
```
---
