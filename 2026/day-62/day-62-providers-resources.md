### Task 1: Explore the AWS Provider
1. Create a new project directory: `terraform-aws-infra`
2. Write a `providers.tf` file:
   - Define the `terraform` block with `required_providers` pinning the AWS provider to version `~> 5.0`
   - Define the `provider "aws"` block with your region
```
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "ap-south-1"
}
```
3. Run `terraform init` and check the output -- what version was installed?
<img width="890" height="440" alt="image" src="https://github.com/user-attachments/assets/032e364d-3195-4af3-9007-2fc7f97ce34a" />

4. Read the provider lock file `.terraform.lock.hcl` -- what does it do?
<img width="930" height="630" alt="image" src="https://github.com/user-attachments/assets/ac6ce7af-aa67-46e1-a38d-c18e4b9e7f0d" />
```
Locks the exact provider version used
Ensures consistent builds across environments
Prevents unexpected upgrades in future runs
Stores:
provider version
checksums (for security)
```
**Document:** What does `~> 5.0` mean? How is it different from `>= 5.0` and `= 5.0.0`?
```
~> 5.0 → Allows: 5.0.0 → 5.x.x and blocks 6.0.0
>= 5.0 → Allows: 5.0.0 and above
= 5.0.0 → Allows: 5.0
```
---

### Task 2: Build a VPC from Scratch
Create a `main.tf` and define these resources one by one:

1. `aws_vpc` -- CIDR block `10.0.0.0/16`, tag it `"TerraWeek-VPC"`
2. `aws_subnet` -- CIDR block `10.0.1.0/24`, reference the VPC ID from step 1, enable public IP on launch, tag it `"TerraWeek-Public-Subnet"`
3. `aws_internet_gateway` -- attach it to the VPC
4. `aws_route_table` -- create it in the VPC, add a route for `0.0.0.0/0` pointing to the internet gateway
5. `aws_route_table_association` -- associate the route table with the subnet

```
# 1. VPC
resource "aws_vpc" "main_vpc" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "TerraWeek-VPC"
  }
}

# 2. Public Subnet
resource "aws_subnet" "public_subnet" {
  vpc_id                  = aws_vpc.main_vpc.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = true

  tags = {
    Name = "TerraWeek-Public-Subnet"
  }
}

# 3. Internet Gateway
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main_vpc.id

  tags = {
    Name = "TerraWeek-IGW"
  }
}

# 4. Route Table
resource "aws_route_table" "public_rt" {
  vpc_id = aws_vpc.main_vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }

  tags = {
    Name = "TerraWeek-RT"
  }
}

# 5. Route Table Association
resource "aws_route_table_association" "public_assoc" {
  subnet_id      = aws_subnet.public_subnet.id
  route_table_id = aws_route_table.public_rt.id
}
```

Run `terraform plan` -- you should see 5 resources to create.
<img width="770" height="618" alt="image" src="https://github.com/user-attachments/assets/7245c34e-afb4-484f-85cd-f92f89ddb373" />

**Verify:** Apply and check the AWS VPC console. Can you see all five resources connected?
```
Yes all the 5 resources have been created
```
<img width="1020" height="300" alt="image" src="https://github.com/user-attachments/assets/d9794f1c-f28b-46b7-8534-adca9e8340bd" />
<img width="1600" height="380" alt="image" src="https://github.com/user-attachments/assets/53faec1f-42ae-4d16-a0a4-e729421c9db4" />

---

### Task 3: Understand Implicit Dependencies
Look at your `main.tf` carefully:

1. The subnet references `aws_vpc.main.id` -- this is an implicit dependency
2. The internet gateway references the VPC ID -- another implicit dependency
3. The route table association references both the route table and the subnet

Answer these questions:
- How does Terraform know to create the VPC before the subnet?
```
Terraform builds a dependency graph (DAG) automatically.
When you write:
vpc_id = aws_vpc.main_vpc.id

Terraform understands:
“Subnet depends on VPC”

So it will:

Create VPC first
Then create Subnet
```
- What would happen if you tried to create the subnet before the VPC existed?
```
It would fail immediately

Example error:

InvalidVpcID.NotFound: The vpc ID 'vpc-xxxx' does not exist

Because:

Subnet needs a valid VPC ID
Without VPC → resource cannot exist
```
- Find all implicit dependencies in your config and list them
```
vpc_id = aws_vpc.main_vpc.id

Subnet depends on VPC
IGW must attach to VPC
Route table belongs to VPC

gateway_id = aws_internet_gateway.igw.id
Route needs IGW

Route Table Association → Subnet
subnet_id = aws_subnet.public_subnet.id

Route Table Association → Route Table
route_table_id = aws_route_table.public_rt.id
```
```
VPC
 ├── Subnet
 ├── Internet Gateway
 └── Route Table
       └── (depends on IGW)

Route Table Association
 ├── Subnet
 └── Route Table

1. VPC
   ↓
2. Subnet   +   Internet Gateway   +   Route Table   (parallel)
   ↓
3. Route (needs IGW)
   ↓
4. Route Table Association (last)

Terraform creates resources in dependency order:

First VPC
Then Subnet, IGW, and Route Table
Then Route (needs IGW)
Finally Route Table Association
```
---

### Task 4: Add a Security Group and EC2 Instance
Add to your config:

1. `aws_security_group` in the VPC:
   - Ingress rule: allow SSH (port 22) from `0.0.0.0/0`
   - Ingress rule: allow HTTP (port 80) from `0.0.0.0/0`
   - Egress rule: allow all outbound traffic
   - Tag: `"TerraWeek-SG"`

2. `aws_instance` in the subnet:
   - Use Amazon Linux 2 AMI for your region
   - Instance type: `t2.micro`
   - Associate the security group
   - Set `associate_public_ip_address = true`
   - Tag: `"TerraWeek-Server"`

Apply and verify -- your EC2 instance should have a public IP and be reachable.
```
# 6. Security Group
resource "aws_security_group" "web_sg" {
  name        = "TerraWeek-SG"
  description = "Allow SSH and HTTP"
  vpc_id      = aws_vpc.main_vpc.id

  # SSH
  ingress {
    description = "SSH Access"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # HTTP
  ingress {
    description = "HTTP Access"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Outbound
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "TerraWeek-SG"
  }
}

# 7. Get Latest Amazon Linux 2 AMI
data "aws_ami" "amazon_linux" {
  most_recent = true

  owners = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

# 8. EC2 Instance
resource "aws_instance" "web_server" {
  ami                         = data.aws_ami.amazon_linux.id
  instance_type               = "t2.micro"
  subnet_id                   = aws_subnet.public_subnet.id
  vpc_security_group_ids      = [aws_security_group.web_sg.id]
  associate_public_ip_address = true

  key_name = "day-62.pub" #  Replace with your key

  tags = {
    Name = "TerraWeek-Server"
  }
}
```
```
$ ssh-keygen day-62

$ terraform init

$ terraform validate

$ terraform plan

$ terraform apply -auto-approve
```
---

### Task 5: Explicit Dependencies with depends_on
Sometimes Terraform cannot detect a dependency automatically.

1. Add a second `aws_s3_bucket` resource for application logs
2. Add `depends_on = [aws_instance.main]` to the S3 bucket -- even though there is no direct reference, you want the bucket created only after the instance
3. Run `terraform plan` and observe the order

```
# S3 Bucket for Logs
resource "aws_s3_bucket" "app_logs" {
  bucket = "umesh-terraweek-logs-12345" # must be globally unique

  tags = {
    Name = "TerraWeek-Logs"
  }

  #  Explicit dependency
  depends_on = [aws_instance.web_server]
}
```

Now visualize the entire dependency tree:
```bash
terraform graph | dot -Tpng > graph.png
```
If you don't have `dot` (Graphviz) installed, use:
```bash
terraform graph
```
and paste the output into an online Graphviz viewer.
```
digraph G {
  rankdir = "RL";
  node [shape = rect, fontname = "sans-serif"];
  "data.aws_ami.amazon_linux" [label="data.aws_ami.amazon_linux"];
  "aws_instance.web_server" [label="aws_instance.web_server"];
  "aws_internet_gateway.igw" [label="aws_internet_gateway.igw"];
  "aws_key_pair.my_key_pair" [label="aws_key_pair.my_key_pair"];
  "aws_route_table.public_rt" [label="aws_route_table.public_rt"];
  "aws_route_table_association.public_assoc" [label="aws_route_table_association.public_assoc"];
  "aws_s3_bucket.app_logs" [label="aws_s3_bucket.app_logs"];
  "aws_security_group.web_sg" [label="aws_security_group.web_sg"];
  "aws_subnet.public_subnet" [label="aws_subnet.public_subnet"];
  "aws_vpc.main_vpc" [label="aws_vpc.main_vpc"];
  "aws_instance.web_server" -> "data.aws_ami.amazon_linux";
  "aws_instance.web_server" -> "aws_key_pair.my_key_pair";
  "aws_instance.web_server" -> "aws_security_group.web_sg";
  "aws_instance.web_server" -> "aws_subnet.public_subnet";
  "aws_internet_gateway.igw" -> "aws_vpc.main_vpc";
  "aws_route_table.public_rt" -> "aws_internet_gateway.igw";
  "aws_route_table_association.public_assoc" -> "aws_route_table.public_rt";
  "aws_route_table_association.public_assoc" -> "aws_subnet.public_subnet";
  "aws_s3_bucket.app_logs" -> "aws_instance.web_server";
  "aws_security_group.web_sg" -> "aws_vpc.main_vpc";
  "aws_subnet.public_subnet" -> "aws_vpc.main_vpc";
}

Flow: 
aws_vpc
 ├── aws_subnet
 │    └── aws_instance
 │          └── aws_s3_bucket (depends_on)
 ├── aws_internet_gateway
 └── aws_route_table
```
<img width="900" height="271" alt="image" src="https://github.com/user-attachments/assets/750d9e2f-9a81-4317-ac3a-937894b2ff8e" />


**Document:** When would you use `depends_on` in real projects? Give two examples.
```
Use depends_on when:

There is NO direct reference
But order still matters

Ex 1:
Ensure EC2 is ready before logging system setup
Avoid race conditions in automation scripts

Ex 2:
App needs DB to be fully available before starting
Otherwise app crashes on startup
```
---

### Task 6: Lifecycle Rules and Destroy
1. Add a `lifecycle` block to your EC2 instance:
```hcl
lifecycle {
  create_before_destroy = true
}
```
2. Change the AMI ID to a different one and run `terraform plan` -- observe that Terraform plans to create the new instance before destroying the old one

3. Destroy everything:
```bash
terraform destroy
```
4. Watch the destroy order -- Terraform destroys in reverse dependency order. Verify in the AWS console that everything is cleaned up.

**Document:** What are the three lifecycle arguments (`create_before_destroy`, `prevent_destroy`, `ignore_changes`) and when would you use each?

```
-> Create new resource before deleting old one

When to use:
Zero downtime deployments
Updating EC2, Load Balancer, ASG

Purpose:

-> Prevent accidental deletion of resource

When to use:
Critical infrastructure
Example:
Production database (RDS)
Important S3 bucket

If terraform destroy is run → Terraform throws error

-> Ignore changes made outside Terraform

lifecycle {
  ignore_changes = [tags, instance_type]
}lifecycle {
  ignore_changes = [tags, instance_type]
}

When to use:
External systems modify resources
Example:
Auto Scaling changes instance count
Dev team manually updates tags

Terraform will NOT try to revert those changes
```
```
# 1. VPC
resource "aws_vpc" "main_vpc" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "TerraWeek-VPC"
  }
}

# 2. Public Subnet
resource "aws_subnet" "public_subnet" {
  vpc_id                  = aws_vpc.main_vpc.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "ap-south-1a"
  map_public_ip_on_launch = true

  tags = {
    Name = "TerraWeek-Public-Subnet"
  }
}

# 3. Internet Gateway
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main_vpc.id

  tags = {
    Name = "TerraWeek-IGW"
  }
}

# 4. Route Table
resource "aws_route_table" "public_rt" {
  vpc_id = aws_vpc.main_vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }

  tags = {
    Name = "TerraWeek-RT"
  }
}

# 5. Route Table Association
resource "aws_route_table_association" "public_assoc" {
  subnet_id      = aws_subnet.public_subnet.id
  route_table_id = aws_route_table.public_rt.id
}

# 6. Security Group
resource "aws_security_group" "web_sg" {
  name        = "TerraWeek-SG"
  description = "Allow SSH and HTTP"
  vpc_id      = aws_vpc.main_vpc.id

  # SSH
  ingress {
    description = "SSH Access"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # HTTP
  ingress {
    description = "HTTP Access"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # Outbound
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "TerraWeek-SG"
  }
}

# 7. Get Latest Amazon Linux 2 AMI
data "aws_ami" "amazon_linux" {
  most_recent = true

  owners = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-*"] # chnaged from gp3 to *
  }
}

# 8. EC2 Instance
resource "aws_instance" "web_server" {
  ami                         = data.aws_ami.amazon_linux.id
  instance_type               = "t3.micro"
  subnet_id                   = aws_subnet.public_subnet.id
  vpc_security_group_ids      = [aws_security_group.web_sg.id]
  associate_public_ip_address = true

  key_name = aws_key_pair.my_key_pair.key_name # Replace key

  lifecycle {
    create_before_destroy = true
  }
  tags = {
    Name = "TerraWeek-Server"
  }
}

# 9. Keypair

resource "aws_key_pair" "my_key_pair" {
  key_name   = "day-62"
  public_key = file("day-62.pub")
}

# 10. S3 Bucket for Logs
resource "aws_s3_bucket" "app_logs" {
  bucket = "umesh-terraweek-logs-12345" # must be globally unique

  tags = {
    Name = "TerraWeek-Logs"
  }

  #  Explicit dependency
  depends_on = [aws_instance.web_server]
}

```

---

