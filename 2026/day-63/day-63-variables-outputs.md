### Task 1: Extract Variables
Take your Day 62 infrastructure config and refactor it:

1. Create a `variables.tf` file with input variables for:
   - `region` (string, default: your preferred region)
   - `vpc_cidr` (string, default: `"10.0.0.0/16"`)
   - `subnet_cidr` (string, default: `"10.0.1.0/24"`)
   - `instance_type` (string, default: `"t2.micro"`)
   - `project_name` (string, no default -- force the user to provide it)
   - `environment` (string, default: `"dev"`)
   - `allowed_ports` (list of numbers, default: `[22, 80, 443]`)
   - `extra_tags` (map of strings, default: `{}`)

2. Replace every hardcoded value in `main.tf` with `var.<name>` references
3. Run `terraform plan` -- it should prompt you for `project_name` since it has no default
<img width="1234" height="762" alt="image" src="https://github.com/user-attachments/assets/130bce27-2934-48b2-97e9-7cd3f974166f" />

**Document:** What are the five variable types in Terraform? (`string`, `number`, `bool`, `list`, `map`)
```
Terraform supports these core variable types:

1) string → Text values
Example: "ap-south-1"

2) number → Numeric values
Example: 10

3) bool → Boolean values
Example: true, false

4) list → Ordered collection
Example: [22, 80, 443]

5) map → Key-value pairs
Example:

{
  env = "dev"
  team = "infra"
}
```
---

### Task 2: Variable Files and Precedence
1. Create `terraform.tfvars`:
```hcl
project_name = "terraweek"
environment  = "dev"
instance_type = "t2.micro"
```

2. Create `prod.tfvars`:
```hcl
project_name = "terraweek"
environment  = "prod"
instance_type = "t3.small"
vpc_cidr     = "10.1.0.0/16"
subnet_cidr  = "10.1.1.0/24"
```

3. Apply with the default file:
```bash
terraform plan                              # Uses terraform.tfvars automatically
```

4. Apply with the prod file:
```bash
terraform plan -var-file="prod.tfvars"      # Uses prod.tfvars
```
<img width="1053" height="500" alt="image" src="https://github.com/user-attachments/assets/8598cb33-8fa2-40e4-89fb-52c4dffa61a2" />

5. Override with CLI:
```bash
terraform plan -var="instance_type=t2.nano"  # CLI overrides everything
```
<img width="1100" height="321" alt="image" src="https://github.com/user-attachments/assets/8027ad9f-cb43-4eea-a6c3-b2405e11065f" />

6. Set an environment variable:
```bash
export TF_VAR_environment="staging"
terraform plan                              # env var overrides default but not tfvars
```

**Document:** Write the variable precedence order from lowest to highest priority.
```
variables.tf (default)
↓
TF_VAR_* (environment variables)
↓
terraform.tfvars
↓
*.auto.tfvars
↓
-var-file
↓
-var (CLI)  [highiest]
```
---

### Task 3: Add Outputs
Create an `outputs.tf` file with outputs for:

1. `vpc_id` -- the VPC ID
2. `subnet_id` -- the public subnet ID
3. `instance_id` -- the EC2 instance ID
4. `instance_public_ip` -- the public IP of the EC2 instance
5. `instance_public_dns` -- the public DNS name
6. `security_group_id` -- the security group ID

Apply your config and verify the outputs are printed at the end:
```bash
terraform apply

# After apply, you can also run:
terraform output                          # Show all outputs
terraform output instance_public_ip       # Show a specific output
terraform output -json                    # JSON format for scripting
```
<img width="799" height="244" alt="image" src="https://github.com/user-attachments/assets/07025767-e4e7-4af9-b2b7-52a42cba1a57" />

<img width="889" height="169" alt="image" src="https://github.com/user-attachments/assets/6649878c-0455-445e-9f94-123de1296669" />

<img width="1042" height="60" alt="image" src="https://github.com/user-attachments/assets/c1ffc9dc-79c0-4142-aadd-0112d156e71a" />

<img width="953" height="735" alt="image" src="https://github.com/user-attachments/assets/d107d84a-56dd-4e9d-aa2c-081c11255c1a" />


**Verify:** Does `terraform output instance_public_ip` return the correct IP?
<img width="435" height="73" alt="image" src="https://github.com/user-attachments/assets/229a3443-2c84-478c-b3ca-3cb3d033163f" />

---

### Task 4: Use Data Sources
Stop hardcoding the AMI ID. Use a data source to fetch it dynamically.

1. Add a `data "aws_ami"` block that:
   - Filters for Amazon Linux 2 images
   - Filters for `hvm` virtualization and `gp2` root device
   - Uses `owners = ["amazon"]`
   - Sets `most_recent = true`

2. Replace the hardcoded AMI in your `aws_instance` with `data.aws_ami.amazon_linux.id`

3. Add a `data "aws_availability_zones"` block to fetch available AZs in your region

4. Use the first AZ in your subnet: `data.aws_availability_zones.available.names[0]`

Apply and verify -- your config now works in any region without changing the AMI.

**Document:** What is the difference between a `resource` and a `data` source?
```
Resource

Creates or manages infrastructure
Terraform controls lifecycle (create, update, delete)

Data Source

Fetches existing information
Does NOT create anything
```
---

### Task 5: Use Locals for Dynamic Values
1. Add a `locals` block:
```hcl
locals {
  name_prefix = "${var.project_name}-${var.environment}"
  common_tags = {
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}
```

2. Replace all Name tags with `local.name_prefix`:
   - VPC: `"${local.name_prefix}-vpc"`
   - Subnet: `"${local.name_prefix}-subnet"`
   - Instance: `"${local.name_prefix}-server"`

3. Merge common tags with resource-specific tags:
```hcl
tags = merge(local.common_tags, {
  Name = "${local.name_prefix}-server"
})
```

Apply and check the tags in the AWS console -- every resource should have consistent tagging.
<img width="989" height="315" alt="image" src="https://github.com/user-attachments/assets/cf0c668c-21dc-4209-9d93-256518614717" />

---

### Task 6: Built-in Functions and Conditional Expressions
Practice these in `terraform console`:
```bash
terraform console
```

1. **String functions:**
   - `upper("terraweek")` -> `"TERRAWEEK"`
   - `join("-", ["terra", "week", "2026"])` -> `"terra-week-2026"`
   - `format("arn:aws:s3:::%s", "my-bucket")`

2. **Collection functions:**
   - `length(["a", "b", "c"])` -> `3`
   - `lookup({dev = "t2.micro", prod = "t3.small"}, "dev")` -> `"t2.micro"`
   - `toset(["a", "b", "a"])` -> removes duplicates
<img width="984" height="441" alt="image" src="https://github.com/user-attachments/assets/8b25a31b-87f6-4c6f-8125-1513f84bf7ee" />

3. **Networking function:**
   - `cidrsubnet("10.0.0.0/16", 8, 1)` -> `"10.0.1.0/24"`

4. **Conditional expression** -- add this to your config:
```hcl
instance_type = var.environment == "prod" ? "t3.small" : "t2.micro"
```

Apply with `environment = "prod"` and verify the instance type changes.
<img width="819" height="215" alt="image" src="https://github.com/user-attachments/assets/c7bc70ef-9c2e-46d6-8a67-197376258ab9" />
<img width="993" height="567" alt="image" src="https://github.com/user-attachments/assets/847a7256-f5e1-4b2b-8029-52d691663bd1" />

**Document:** Pick five functions you find most useful and explain what each does.

---

```
*************************************************C O D E********************************************************
```
nain.tf
```
# -------------------------
# Locals (Task 5)
# -------------------------
locals {
  name_prefix = "${var.project_name}-${var.environment}"

  common_tags = {
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}

# -------------------------
# Data Source: Availability Zones
# -------------------------
data "aws_availability_zones" "available" {
  state = "available"
}

# -------------------------
# Data Source: Latest Amazon Linux 2 AMI
# -------------------------
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }

  filter {
    name   = "root-device-type"
    values = ["ebs"]
  }
}

# -------------------------
# Random ID
# -------------------------
resource "random_id" "suffix" {
  byte_length = 4
}

# -------------------------
# 1. VPC (FIXED DNS SUPPORT)
# -------------------------
resource "aws_vpc" "main_vpc" {
  cidr_block = var.vpc_cidr

  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-vpc"
  })
}

# -------------------------
# 2. Public Subnet
# -------------------------
resource "aws_subnet" "public_subnet" {
  vpc_id                  = aws_vpc.main_vpc.id
  cidr_block              = var.subnet_cidr
  availability_zone       = data.aws_availability_zones.available.names[0]
  map_public_ip_on_launch = true

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-subnet"
  })
}

# -------------------------
# 3. Internet Gateway
# -------------------------
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main_vpc.id

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-igw"
  })
}

# -------------------------
# 4. Route Table
# -------------------------
resource "aws_route_table" "public_rt" {
  vpc_id = aws_vpc.main_vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-rt"
  })
}

# -------------------------
# 5. Route Table Association
# -------------------------
resource "aws_route_table_association" "public_assoc" {
  subnet_id      = aws_subnet.public_subnet.id
  route_table_id = aws_route_table.public_rt.id
}

# -------------------------
# 6. Security Group
# -------------------------
resource "aws_security_group" "web_sg" {
  name        = "${local.name_prefix}-sg"
  description = "Allow dynamic ports"
  vpc_id      = aws_vpc.main_vpc.id

  dynamic "ingress" {
    for_each = var.allowed_ports
    content {
      description = "Port ${ingress.value}"
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-sg"
  })
}

# -------------------------
# 7. Key Pair
# -------------------------
resource "aws_key_pair" "my_key_pair" {
  key_name   = "${var.project_name}-key"
  public_key = file("/home/umesh/Documents/Terraform_Practise/Day-61/day-62.pub")
}

# -------------------------
# 8. EC2 Instance (FIXED LOGIC)
# -------------------------
resource "aws_instance" "web_server" {
  ami = data.aws_ami.amazon_linux.id

  instance_type = var.environment == "prod" ? "t3.small" : var.instance_type

  subnet_id                   = aws_subnet.public_subnet.id
  vpc_security_group_ids      = [aws_security_group.web_sg.id]
  associate_public_ip_address = true

  key_name = aws_key_pair.my_key_pair.key_name

  lifecycle {
    create_before_destroy = true
  }

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-server"
  })
}

# -------------------------
# 9. S3 Bucket for Logs
# -------------------------
resource "aws_s3_bucket" "app_logs" {
  bucket = "${local.name_prefix}-logs-${random_id.suffix.hex}"

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-logs"
  })

  depends_on = [aws_instance.web_server]
}
```
outputs.tf

```
output "vpc_id" {
  description = "The VPC ID"
  value       = aws_vpc.main_vpc.id  # <RESOURCE_TYPE>.<RESOURCE_NAME>.<ATTRIBUTE>  resource "aws_security_group" "web_sg"
}

output "subnet_id" {
  description = "The public subnet ID"
  value       = aws_subnet.public_subnet.id
}

output "instance_id" {
  description = "The EC2 instance ID"
  value       = aws_instance.web_server.id
}

output "instance_public_ip" {
  description = "Public IP of EC2 instance"
  value       = aws_instance.web_server.public_ip
}

output "instance_public_dns" {
  description = "Public DNS of EC2 instance"
  value       = aws_instance.web_server.public_dns
}

output "security_group_id" {
  description = "Security group ID"
  value       = aws_security_group.web_sg.id
}
```
prod.tfvars

```
project_name = "terraweek"
environment  = "prod"
instance_type = "t3.small"
vpc_cidr     = "10.1.0.0/16"
subnet_cidr  = "10.1.1.0/24"
```
providers.tf

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
  region = var.region
}
```
terrafor.tfvars

```
project_name = "terraweek"
environment  = "dev"
instance_type = "t2.micro"
```
variabels.tf

```
variable "region" {
  type    = string
  default = "ap-south-1"
}

variable "vpc_cidr" {
  type    = string
  default = "10.0.0.0/16"
}

variable "subnet_cidr" {
  type    = string
  default = "10.0.1.0/24"
}

variable "instance_type" {
  type    = string
  default = "t2.nano"
}

variable "project_name" {
  type = string
}

variable "environment" {
  type    = string
  default = "dev"
}

variable "allowed_ports" {
  type    = list(number)
  default = [22, 80, 443]
}

variable "extra_tags" {
  type    = map(string)
  default = {}
}
```
