### Task 1: Your First Playbook
Create `install-nginx.yml`:

```yaml
---
- name: Install and start Nginx on web servers
  hosts: web
  become: true

  tasks:
    - name: Install Nginx
      yum:
        name: nginx
        state: present

    - name: Start and enable Nginx
      service:
        name: nginx
        state: started
        enabled: true

    - name: Create a custom index page
      copy:
        content: "<h1>Deployed by Ansible - TerraWeek Server</h1>"
        dest: /usr/share/nginx/html/index.html
```

(Use `apt` instead of `yum` if your instances run Ubuntu)

Run it:
```bash
ansible-playbook install-nginx.yml
```
```
- name: Install and start Nginx on web servers
  hosts: web
  become: true

  tasks:
    - name: Install Nginx
      package: # automatically picks apt, yum, etc.
        name: nginx
        state: present

    - name: Start and enable Nginx
      service:
        name: nginx
        state: started
        enabled: true

    - name: Create a custom index page
      copy:
        src: index.html 
        dest: /var/www/html/index.html
```
```
# Run the playbook

ansible-playbook -i hosts.ini install-nginx.yml -v
```
Read the output carefully -- every task shows `changed`, `ok`, or `failed`.

Now run it **again**. Notice that tasks show `ok` instead of `changed`. This is **idempotency** -- Ansible only makes changes when needed.

**Verify:** Curl the web server's public IP. Do you see your custom page?
```
curl <IP-Address>
```
<img width="1063" height="317" alt="image" src="https://github.com/user-attachments/assets/b91cbd2d-7675-4e3f-909d-5eed33ccb9a7" />
<img width="1063" height="317" alt="image" src="https://github.com/user-attachments/assets/5f9997bf-922a-4eb1-985a-0314bdc0ac5c" />
<img width="839" height="85" alt="image" src="https://github.com/user-attachments/assets/e33dd785-e86c-499e-b23b-a3e8f5c9017d" />

---

### Task 2: Understand the Playbook Structure
Open your playbook and annotate each part in your notes:

```yaml
---                                    # YAML document start
- name: Play name                      # PLAY -- targets a group of hosts
  hosts: web                           # Which inventory group to run on
  become: true                         # Run tasks as root (sudo)

  tasks:                               # List of TASKS in this play
    - name: Task name                  # TASK -- one unit of work
      module_name:                     # MODULE -- what Ansible does
        key: value                     # Module arguments
```
```
- name: Install and start Nginx on web servers   # PLAY → defines the goal
  hosts: web                                     # Target group from inventory
  become: true                                   # Run all tasks as root (sudo)

  tasks:                                         # Start of task list

    - name: Install Nginx                        # TASK 1 → install package
      package:                                   # MODULE → generic package manager
        name: nginx                              # Package name
        state: present                           # Ensure it's installed

    - name: Start and enable Nginx               # TASK 2 → manage service
      service:                                   # MODULE → service control
        name: nginx                              # Service name
        state: started                           # Ensure it's running
        enabled: true                            # Enable on boot

    - name: Create a custom index page           # TASK 3 → deploy file
      copy:                                      # MODULE → file copy
        src: index.html                          # Source file (local machine)
        dest: /var/www/html/index.html           # Destination on remote server
```

Answer:
1. What is the difference between a play and a task?
```
Play → defines who and how
(which hosts + privilege + overall execution)
Task → defines what to do
(install package, start service, copy file, etc.)
```
2. Can you have multiple plays in one playbook?
```
Yes we can have multiple playbooks

- name: Setup web servers
  hosts: web
  tasks:
    - ...

- name: Setup database servers
  hosts: db
  tasks:
    - ...
```
3. What does `become: true` do at the play level vs the task level?
```
At Play Level

-> Applies to ALL tasks in that play
-> Cleaner and preferred

At Task Level

-> Only that task runs with sudo

Use task only when few tasks need sudo to run commands
```
4. What happens if a task fails -- do remaining tasks still run?
```
By default:

Playbook STOPS for that host
Remaining tasks do NOT run on that host

BUT:

Other hosts continue (if multiple hosts exist)

It can be skipped/igonerd by: ignore_errors: yes
```
---

### Task 3: Learn the Essential Modules
Practice each of these modules by writing a playbook called `essential-modules.yml` with multiple tasks:

1. **`yum`/`apt`** -- Install and remove packages:
```yaml
- name: Install multiple packages
  yum:
    name:
      - git
      - curl
      - wget
      - tree
    state: present
```

2. **`service`** -- Manage services:
```yaml
- name: Ensure Nginx is running
  service:
    name: nginx
    state: started
    enabled: true
```

3. **`copy`** -- Copy files from control node to managed nodes:
```yaml
- name: Copy config file
  copy:
    src: files/app.conf
    dest: /etc/app.conf
    owner: root
    group: root
    mode: '0644'
```

4. **`file`** -- Create directories and manage permissions:
```yaml
- name: Create application directory
  file:
    path: /opt/myapp
    state: directory
    owner: ec2-user
    mode: '0755'
```

5. **`command`** -- Run a command (no shell features):
```yaml
- name: Check disk space
  command: df -h
  register: disk_output

- name: Print disk space
  debug:
    var: disk_output.stdout_lines
```

6. **`shell`** -- Run a command with shell features (pipes, redirects):
```yaml
- name: Count running processes
  shell: ps aux | wc -l
  register: process_count

- name: Show process count
  debug:
    msg: "Total processes: {{ process_count.stdout }}"
```

7. **`lineinfile`** -- Add or modify a single line in a file:
```yaml
- name: Set timezone in environment
  lineinfile:
    path: /etc/environment
    line: 'TZ=Asia/Kolkata'
    create: true
```
Create a `files/` directory with a sample `app.conf` file for the copy task. Run the playbook against all servers.
```
cd playbooks
touch essential-modules.yml
mkdir files
touch files/app.conf
```
```
cd files
vim files/app.conf

[app]
name=myapp
version=1.0
```
$ vim essential-modules.yml
```
- name: Practice essential Ansible modules
  hosts: all
  become: true

  tasks:

    # Install packages
    - name: Install multiple packages
      package:
        name:
          - git
          - curl
          - wget
          - tree
        state: present

    # Ensure nginx is running
    - name: Ensure Nginx is running
      service:
        name: nginx
        state: started
        enabled: true

    # Copy config file
    - name: Copy config file
      copy:
        src: files/app.conf
        dest: /etc/app.conf
        owner: root
        group: root
        mode: '0644'

    # Create directory
    - name: Create application directory
      file:
        path: /opt/myapp
        state: directory
        owner: ubuntu   # change if needed (ubuntu → ubuntu)
        mode: '0755'

    # Command module
    - name: Check disk space
      command: df -h
      register: disk_output

    - name: Print disk space
      debug:
        var: disk_output.stdout_lines

    # Shell module
    - name: Count running processes
      shell: ps aux | wc -l
      register: process_count

    - name: Show process count
      debug:
        msg: "Total processes: {{ process_count.stdout }}"

    # lineinfile module
    - name: Set timezone in environment
      lineinfile:
        path: /etc/environment
        line: 'TZ=Asia/Kolkata'
        create: true
```
Run a playbook
```
ansible-playbook -i hosts.ini essential-modules.yml
```

**Document:** What is the difference between `command` and `shell`? When should you use each?
```
Command
Runs simple commands
No pipes (|), redirects (>), variables

Shell
Runs inside a shell (bash)
Supports pipes, redirects, variables
```
<img width="1089" height="988" alt="image" src="https://github.com/user-attachments/assets/f9a2e628-d40d-4864-ac3f-ce1777d3edea" />

---

### Task 4: Handlers -- Restart Services Only When Needed
Handlers are tasks that run only when triggered by a `notify`. This avoids unnecessary service restarts.

Create `nginx-config.yml`:
```yaml
---
- name: Configure Nginx with a custom config
  hosts: web
  become: true

  tasks:
    - name: Install Nginx
      yum:
        name: nginx
        state: present

    - name: Deploy Nginx config
      copy:
        src: files/nginx.conf
        dest: /etc/nginx/nginx.conf
        owner: root
        mode: '0644'
      notify: Restart Nginx

    - name: Deploy custom index page
      copy:
        content: "<h1>Managed by Ansible</h1><p>Server: {{ inventory_hostname }}</p>"
        dest: /usr/share/nginx/html/index.html

    - name: Ensure Nginx is running
      service:
        name: nginx
        state: started
        enabled: true

  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

Create `files/nginx.conf` with a basic Nginx config.

Run the playbook:
- First run: handler triggers because the config file is new
- Second run: handler does NOT trigger because nothing changed

**Verify:** Run it twice and compare the output. Does the handler run both times?

---

### Task 5: Dry Run, Diff, and Verbosity
Before running playbooks on production, always preview changes first.

1. **Dry run (check mode)** -- shows what would change without changing anything:
```bash
ansible-playbook install-nginx.yml --check
```

2. **Diff mode** -- shows the actual file differences:
```bash
ansible-playbook nginx-config.yml --check --diff
```

3. **Verbosity** -- increase output detail for debugging:
```bash
ansible-playbook install-nginx.yml -v       # verbose
ansible-playbook install-nginx.yml -vv      # more verbose
ansible-playbook install-nginx.yml -vvv     # connection debugging
```

4. **Limit to specific hosts:**
```bash
ansible-playbook install-nginx.yml --limit web-server
```

5. **List what would be affected without running:**
```bash
ansible-playbook install-nginx.yml --list-hosts
ansible-playbook install-nginx.yml --list-tasks
```

**Document:** Why is `--check --diff` the most important flag combination for production use?

---

### Task 6: Multiple Plays in One Playbook
Write `multi-play.yml` with separate plays for each server group:

```yaml
---
- name: Configure web servers
  hosts: web
  become: true
  tasks:
    - name: Install Nginx
      yum:
        name: nginx
        state: present
    - name: Start Nginx
      service:
        name: nginx
        state: started
        enabled: true

- name: Configure app servers
  hosts: app
  become: true
  tasks:
    - name: Install Node.js dependencies
      yum:
        name:
          - gcc
          - make
        state: present
    - name: Create app directory
      file:
        path: /opt/app
        state: directory
        mode: '0755'

- name: Configure database servers
  hosts: db
  become: true
  tasks:
    - name: Install MySQL client
      yum:
        name: mysql
        state: present
    - name: Create data directory
      file:
        path: /var/lib/appdata
        state: directory
        mode: '0700'
```

Run it:
```bash
ansible-playbook multi-play.yml
```

Watch the output -- each play targets a different group, and tasks run only on the relevant hosts.

**Verify:** Is Nginx only installed on web servers? Is MySQL only on db servers?

**TrainWithShubham**
