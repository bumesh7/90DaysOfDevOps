### Task 1: Understand Infrastructure as Code
Before touching the terminal, research and write short notes on:

1. What is Infrastructure as Code (IaC)? Why does it matter in DevOps?
```
Infrastructure as Code (IaC) means managing and provisioning infrastructure using code instead of manual steps.
```

2. What problems does IaC solve compared to manually creating resources in the AWS console?
```
In AWS console we need to manually click and create infrastructure, where as IAC solves issue like manuall process, fast automation
and single source of truth.
```
3. How is Terraform different from AWS CloudFormation, Ansible, and Pulumi?
```
Terraform => Declarative, Cloud-agnostic (works with AWS, Azure, GCP), Uses HCL (simple syntax), Maintains state file

Cloud Formation => Declarative, AWS-only, Uses JSON/YAML

Ansible => Procedural (step-by-step), Focuses on configuration, not infra creation

Pulumi => Uses programming languages (Python, JS, Go), Declarative + imperative mix
```
4. What does it mean that Terraform is "declarative" and "cloud-agnostic"?
```
Declarative = You tell what you want not how to do
Cloud-agnostic = Works across multiple cloud providers
```
---

### Task 2: Install Terraform and Configure AWS
1. Install Terraform:
```bash
# macOS
brew tap hashicorp/tap
brew install hashicorp/tap/terraform

# Linux (amd64)
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

# Windows
choco install terraform
```

2. Verify:
```bash
terraform -version
```
<img width="539" height="117" alt="image" src="https://github.com/user-attachments/assets/03840abb-4805-4054-acf0-f4a40dacb10a" />

3. Install and configure the AWS CLI:
```bash
aws configure
# Enter your Access Key ID, Secret Access Key, default region (e.g., ap-south-1), output format (json)
```
<img width="760" height="142" alt="image" src="https://github.com/user-attachments/assets/ad8b53b3-2b73-4909-ace3-c0acc12d0f55" />

4. Verify AWS access:
```bash
aws sts get-caller-identity
```
```
{
    "UserId": "AID*******************DEW",
    "Account": "**************",
    "Arn": "arn:aws:iam::493467536664:user/test1"
}
```
---

### Task 3: Your First Terraform Config -- Create an S3 Bucket
Create a project directory and write your first Terraform config:

```bash
mkdir terraform-basics && cd terraform-basics
```

Create a file called `main.tf` with:
1. A `terraform` block with `required_providers` specifying the `aws` provider
2. A `provider "aws"` block with your region
3. A `resource "aws_s3_bucket"` that creates a bucket with a globally unique name

```
$ vim s3.tf

resource "aws_s3_bucket" "example" {
  bucket = "umesh-s3-bucket-true"
}

$ vim providers.tf

provider "aws" {
  region = "ap-south-1"
}

$ terraform init
$ terraform fmt
$ terraform plan
$ terraform apply -auto-approve
```
Run the Terraform lifecycle:
```bash
terraform init      # Download the AWS provider
terraform plan      # Preview what will be created
terraform apply     # Create the bucket (type 'yes' to confirm)
```

Go to the AWS S3 console and verify your bucket exists.

<img width="1043" height="67" alt="image" src="https://github.com/user-attachments/assets/092153a6-18d6-49f5-9e30-4c61a32927b0" />

**Document:** What did `terraform init` download? What does the `.terraform/` directory contain?
```
terraform init downloads => providers, modules if used and Backend Initilization

Dependency Lock File (.terraform.lock.hcl)
Stores exact provider versions
Ensures consistency across machines

.terraform/
├── providers/
├── modules/
├── terraform.tfstate (sometimes, local backend)
└── other metadata
```
---

### Task 4: Add an EC2 Instance
In the same `main.tf`, add:
1. A `resource "aws_instance"` using AMI `ami-0f5ee92e2d63afc18` (Amazon Linux 2 in ap-south-1 -- use the correct AMI for your region)
2. Set instance type to `t2.micro`
3. Add a tag: `Name = "TerraWeek-Day1"`

```
resource "aws_instance" "terraweek_day1" {
  ami           = "ami-0f5ee92e2d63afc18"  # Amazon Linux 2 (ap-south-1)
  instance_type = "t2.micro"

  tags = {
    Name = "TerraWeek-Day1"
  }
}
```

Run:
```bash
terraform plan      # You should see 1 resource to add (bucket already exists)
terraform apply
```

Go to the AWS EC2 console and verify your instance is running with the correct name tag.
<img width="1572" height="81" alt="image" src="https://github.com/user-attachments/assets/ad3809bb-bf91-47e1-8a16-69c5c9c1b7a1" />

**Document:** How does Terraform know the S3 bucket already exists and only the EC2 instance needs to be created?
```
terraform.tfstate file manages which resource terraform already created and also fetch the terraform configuration.
```
---

### Task 5: Understand the State File
Terraform tracks everything it creates in a state file. Time to inspect it.

1. Open `terraform.tfstate` in your editor -- read the JSON structure
2. Run these commands and document what each returns:
```bash
terraform show                          # Human-readable view of current state
terraform state list                    # List all resources Terraform manages
terraform state show aws_s3_bucket.<name>   # Detailed view of a specific resource
terraform state show aws_instance.<name>
```
<img width="1153" height="979" alt="image" src="https://github.com/user-attachments/assets/cab02450-3867-42c4-b385-f35c64e556de" />

3. Answer these questions in your notes:
   - What information does the state file store about each resource?
```
Terraform state stores:

Resource type & name
Real-world ID (like EC2 instance ID i-xxxxx)
All attributes (IP, tags, config)
Dependency relationships
Provider details
```
   - Why should you never manually edit the state file?
```
You can corrupt the file
Terraform may lose track of resources
Can cause duplicate creation or deletion
```
   - Why should the state file not be committed to Git?
```
It contains Sensitive data like
Access keys
Passwords
Private IPs / infrastructure details

Collaboration issues

Multiple people editing → conflicts 
No locking → race conditions
```
---

### Task 6: Modify, Plan, and Destroy
1. Change the EC2 instance tag from `"TerraWeek-Day1"` to `"TerraWeek-Modified"` in your `main.tf`
2. Run `terraform plan` and read the output carefully:
   - What do the `~`, `+`, and `-` symbols mean?
```
Symbol	Meaning
+	Resource will be created
-	Resource will be destroyed
~	Resource will be updated in-place
```
   - Is this an in-place update or a destroy-and-recreate?
```
The resouce is updated
```
3. Apply the change
4. Verify the tag changed in the AWS console
5. Finally, destroy everything:
```bash
terraform destroy
```
6. Verify in the AWS console -- both the S3 bucket and EC2 instance should be gone

<img width="1507" height="659" alt="image" src="https://github.com/user-attachments/assets/8c874802-26f1-4a6d-9798-28397f83866a" />

<img width="1607" height="145" alt="image" src="https://github.com/user-attachments/assets/6cc56ba5-0ece-40b6-a716-60b4fff79ebe" />

<img width="1018" height="218" alt="image" src="https://github.com/user-attachments/assets/fa69b4b0-499f-43b1-9662-5e50fac4f46d" />
