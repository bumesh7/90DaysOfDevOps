### Task 1: Understand Ansible
Research and write short notes on:

1. What is configuration management? Why do we need it?
```
Configuration Management (CM) is the process of maintaining systems (servers, software, settings) in a consistent, desired state.
Consistency → Same setup across all servers 
Automation → No manual setup → saves time
Scalability → Easily manage 10 → 1000 servers
Version control → Track infra changes like code
Reduced errors → Avoid human mistakes
```
2. How is Ansible different from Chef, Puppet, and Salt?
```
Ansible
Simple, human-readable (YAML)
No agents → lightweight
Push model (you control execution)

Chef/Puppet
Require agents installed on nodes
More setup & maintenance
Better for very large enterprise automation
Pull based
Puppet DSL

Salt
Fast & flexible
Supports both push & pull
YAML/Python
```
3. What does "agentless" mean? How does Ansible connect to managed nodes?
```
Agentless = No software installation required on managed nodes

How Ansible connects:
Uses SSH (Linux) or WinRM (Windows)
Executes tasks remotely from control node
No background daemon needed
```
4. Draw or describe the Ansible architecture:
   - **Control Node** -- the machine where Ansible runs (your laptop or a jump server)
   - **Managed Nodes** -- the servers Ansible configures (your EC2 instances)
   - **Inventory** -- the list of managed nodes
   - **Modules** -- units of work Ansible executes (install a package, copy a file, start a service)
   - **Playbooks** -- YAML files that define what to do on which hosts
```
        [ Control Node ]
               |
        (SSH Connection)
               |
    -------------------------
    |         |            |
[Node1]    [Node2]     [Node3]


Control Node
Machine where Ansible is installed
Example: Your laptop or a jump server
Runs playbooks and sends instructions

Managed Nodes
Target servers (EC2, VMs, etc.)
No Ansible installation needed
Just need SSH access

Inventory
File that lists all servers
Example
[web]
192.168.1.10
192.168.1.11

[db]
192.168.1.20

Modules
Small tasks Ansible executes
Examples:

Install package → apt, yum
Copy file → copy
Start service → service

Modules are the actual work units

Playbooks
YAML files defining automation steps

```
---

### Task 2: Set Up Your Lab Environment
You need 2-3 EC2 instances to practice on. Choose one approach:

**Option A: Use Terraform (recommended -- you just learned this)**
Use your TerraWeek skills to provision 3 EC2 instances with:
- Amazon Linux 2 or Ubuntu 22.04
- `t2.micro` instance type
- A security group allowing SSH (port 22)
- A key pair for SSH access

**Option B: Launch manually from AWS Console**
Create 3 instances with the same specs above.

Label them mentally:
- **Instance 1:** web server
- **Instance 2:** app server
- **Instance 3:** db server

Verify you can SSH into each one from your control node:
```bash
ssh -i ~/your-key.pem ec2-user@<public-ip-1>
ssh -i ~/your-key.pem ec2-user@<public-ip-2>
ssh -i ~/your-key.pem ec2-user@<public-ip-3>
```
```
#ec2.tf

# We have created the ssh-keygen named "terraform-automate-key" and used in  the public_key 

# Region

provider "aws" {
  region = "ap-south-1"
}

# key value pair

resource "aws_key_pair" "my_key_pair" {
  key_name = "terra-automate-key"
  #  public_key = "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINZkVYM8zeQ11CVAGh5/cJHais7karO7JjpM67rdoO4y umesh@umesh-Aspire-A515-57G"
  public_key = file("terra-automate-key.pub")
}

# Default VPC

resource "aws_default_vpc" "default" {
}

# Security Group

resource "aws_security_group" "my_security_group" {

  name        = "terra-security-group"
  vpc_id      = aws_default_vpc.default.id # interpolation
  description = "this is inbound and outbound rule for instance SG"

}

# Inbound & Outbound port rules

resource "aws_vpc_security_group_ingress_rule" "allow_http" {
  security_group_id = aws_security_group.my_security_group.id # changed
  cidr_ipv4         = "0.0.0.0/0"                             # changed
  from_port         = 80
  ip_protocol       = "tcp"
  to_port           = 80
}

resource "aws_vpc_security_group_ingress_rule" "allow_ssh" {
  security_group_id = aws_security_group.my_security_group.id # changed
  cidr_ipv4         = "0.0.0.0/0"                             # changed
  from_port         = 22
  ip_protocol       = "tcp"
  to_port           = 22
}


resource "aws_vpc_security_group_egress_rule" "allow_all_traffic" {
  security_group_id = aws_security_group.my_security_group.id
  cidr_ipv4         = "0.0.0.0/0"
  ip_protocol       = "-1" # semantically equivalent to all ports
}



# Ec2 Instance

resource "aws_instance" "my_instance" {

  for_each = var.instances # loop through the map of instances defined in variables.tf

  ami = each.value.ami # ami id

  instance_type = each.value.instance_type # instance type

  key_name = aws_key_pair.my_key_pair.key_name # key pair

  vpc_security_group_ids = [aws_security_group.my_security_group.id] # vpc and security group


  # root storage (EBS)
  root_block_device {
    volume_size = 10
    volume_type = "gp3"

  }


  tags = {
    Name      = each.key             # Instance name
    os_family = each.value.os_family # OS family

  }
}

resource "aws_ec2_instance_state" "stop_instances" {
  for_each = aws_instance.my_instance

  instance_id = each.value.id
  state       = "stopped"
}
```
```
# variables.tf
variable "instances" {



  description = "Map of instance names to AMI IDs, SSH users, and OS family"



  type = map(object({

    ami = string

    user = string

    os_family = string

    instance_type = string

  }))



  # by deafult value to be put in the variable

  default = {

    "control-node-ubuntu" = {

      ami = "ami-0f58b397bc5c1f2e8" # Ubuntu Server 24.04 LTS

      user = "ubuntu"

      os_family = "ubuntu"

      instance_type = "t3.micro"

    }

    "worker-ubuntu" = {

      ami = "ami-0f58b397bc5c1f2e8" # Ubuntu Server 24.04 LTS

      user = "ubuntu"

      os_family = "ubuntu"

      instance_type = "t3.micro"

    }

    "worker-redhat" = {

      ami = "ami-03793655b06c6e29a" # RHEL 9

      user = "ec2-user"

      os_family = "redhat"

      instance_type = "t3.micro"

    }

    "worker-amazon" = {

      ami = "ami-045443a70fafb8bbc" # Amazon Linux 2023 — update for your region

      user = "ec2-user"

      os_family = "amazon"

      instance_type = "t3.micro"

    }

  }

}

```
---

### Task 3: Install Ansible
Install Ansible on your **control node** (your laptop or one dedicated EC2 instance):

```bash
# macOS
brew install ansible

# Ubuntu/Debian
sudo apt update
sudo apt install ansible -y

# Amazon Linux / RHEL
sudo yum install ansible -y
# or
pip3 install ansible

# Verify
ansible --version
```

Confirm the output shows the Ansible version, config file path, and Python version.

**Document:** On which machine did you install Ansible? Why is it only needed on the control node?

---

### Task 4: Create Your Inventory File
The inventory tells Ansible which servers to manage. Create a project directory and your first inventory:

```bash
mkdir ansible-practice && cd ansible-practice
```

Create a file called `inventory.ini`:
```ini
[web]
web-server ansible_host=<PUBLIC_IP_1>

[app]
app-server ansible_host=<PUBLIC_IP_2>

[db]
db-server ansible_host=<PUBLIC_IP_3>

[all:vars]
ansible_user=ec2-user
ansible_ssh_private_key_file=~/your-key.pem
```

Verify Ansible can reach all hosts:
```bash
ansible all -i inventory.ini -m ping
```

You should see green `SUCCESS` with `"ping": "pong"` for each host.

**Troubleshoot:** If ping fails:
- Check the SSH key path and permissions (`chmod 400 your-key.pem`)
- Check the security group allows SSH from your IP
- Check the `ansible_user` matches your AMI (ec2-user for Amazon Linux, ubuntu for Ubuntu)

---

### Task 5: Run Ad-Hoc Commands
Ad-hoc commands let you run quick one-off tasks without writing a playbook.

1. **Check uptime on all servers:**
```bash
ansible all -i inventory.ini -m command -a "uptime"
```

2. **Check free memory on web servers only:**
```bash
ansible web -i inventory.ini -m command -a "free -h"
```

3. **Check disk space on all servers:**
```bash
ansible all -i inventory.ini -m command -a "df -h"
```

4. **Install a package on the web group:**
```bash
ansible web -i inventory.ini -m yum -a "name=git state=present" --become
```
(Use `apt` instead of `yum` if running Ubuntu)

5. **Copy a file to all servers:**
```bash
echo "Hello from Ansible" > hello.txt
ansible all -i inventory.ini -m copy -a "src=hello.txt dest=/tmp/hello.txt"
```

6. **Verify the file was copied:**
```bash
ansible all -i inventory.ini -m command -a "cat /tmp/hello.txt"
```

**Document:** What does `--become` do? When do you need it?

---

### Task 6: Explore Inventory Groups and Patterns
1. **Create a group of groups** -- add this to your `inventory.ini`:
```ini
[application:children]
web
app

[all_servers:children]
application
db
```

2. Run commands against different groups:
```bash
ansible application -i inventory.ini -m ping     # web + app servers
ansible db -i inventory.ini -m ping               # only db server
ansible all_servers -i inventory.ini -m ping      # everything
```

3. **Use patterns:**
```bash
ansible 'web:app' -i inventory.ini -m ping        # OR: web or app
ansible 'all:!db' -i inventory.ini -m ping        # NOT: all except db
```

4. **Create an `ansible.cfg`** to avoid typing `-i inventory.ini` every time:
```ini
[defaults]
inventory = inventory.ini
host_key_checking = False
remote_user = ec2-user
private_key_file = ~/your-key.pem
```

Now you can simply run:
```bash
ansible all -m ping
```

**Verify:** Does `ansible all -m ping` work without specifying the inventory file?

---

## Hints
- Ansible uses SSH by default -- no agent installation needed on managed nodes
- `ansible.cfg` is read from the current directory first, then `~/.ansible.cfg`, then `/etc/ansible/ansible.cfg`
- `-m` specifies the module, `-a` specifies the module arguments
- `--become` escalates to root (like `sudo`) -- needed for package installation and service management
- `command` module runs simple commands, `shell` module supports pipes and redirects
- Host key checking can cause issues on first connection -- `host_key_checking = False` in config helps during practice
- Ad-hoc commands are great for quick tasks, but playbooks are better for anything repeatable

---

## Documentation
Create `day-68-ansible-intro.md` with:
- Ansible architecture in your own words
- How you set up your lab (Terraform or manual, with instance details)
- Your `inventory.ini` file (redact IPs if sharing publicly)
- Screenshot of `ansible all -m ping` with all green results
- Five ad-hoc commands you ran and their outputs
- Difference between `command` and `shell` modules

---

## Submission
1. Add `day-68-ansible-intro.md` to `2026/day-68/`
2. Commit and push to your fork

---

## Learn in Public
Share on LinkedIn: "Started the Ansible journey today -- set up a control node, created an inventory with three EC2 instances, and ran ad-hoc commands to manage all servers from one terminal. No agents installed anywhere. Ansible just works over SSH."

`#90DaysOfDevOps` `#DevOpsKaJosh` `#TrainWithShubham`

Happy Learning!
**TrainWithShubham**
