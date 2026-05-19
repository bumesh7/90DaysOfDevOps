### Task 1: Inspect Your Current State
Use your Day 63 config (or create a small config with a VPC and EC2 instance). Apply it and then explore the state:

```bash
terraform show                                    # Full state in human-readable format
terraform state list                              # All resources tracked by Terraform
terraform state show aws_instance.<name>          # Every attribute of the instance
terraform state show aws_vpc.<name>               # Every attribute of the VPC
```
<img width="902" height="292" alt="image" src="https://github.com/user-attachments/assets/cf3ce10b-353d-4fb0-9fe6-377714646ebc" />

<img width="1092" height="757" alt="image" src="https://github.com/user-attachments/assets/c00547b0-68b5-4e9f-95c9-e6aa0ce86b7a" />

Answer:
1. How many resources does Terraform track?
```
Terraform track 12 resources
```
2. What attributes does the state store for an EC2 instance? (hint: way more than what you defined)
```
Terraform stores many attributes, including:

Instance ID
AMI ID
Instance type
Public & private IP
Availability zone
Subnet ID
Security groups
Tags
ARN
Disk (block device) details

additionally with aws generated details
```
3. Open `terraform.tfstate` in an editor -- find the `serial` number. What does it represent?
```
It is the state file version number

Increases every time Terraform updates the state
Used to track changes and ensure consistency
```
---

### Task 2: Set Up S3 Remote Backend
Storing state locally is dangerous -- one deleted file and you lose everything. Time to move it to S3.

1. First, create the backend infrastructure (do this manually or in a separate Terraform config):
```bash
# Create S3 bucket for state storage
aws s3api create-bucket \
  --bucket terraweek-state-<yournname> \
  --region ap-south-1 \
  --create-bucket-configuration LocationConstraint=ap-south-1

# Enable versioning (so you can recover previous state)
aws s3api put-bucket-versioning \
  --bucket terraweek-state-<yourname> \
  --versioning-configuration Status=Enabled

# Create DynamoDB table for state locking
aws dynamodb create-table \
  --table-name terraweek-state-lock \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --region ap-south-1
```
Run below code in terminal
```
aws s3api create-bucket \
  --bucket terraweek-state-umesh \
  --region ap-south-1 \
  --create-bucket-configuration LocationConstraint=ap-south-1

aws s3api put-bucket-versioning \
  --bucket terraweek-state-umesh \
  --versioning-configuration Status=Enabled

aws dynamodb create-table \
  --table-name terraweek-state-lock \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --region ap-south-1
```
2. Add the backend block to your Terraform config:

main.tf
```hcl
# -------------------------
# 10. Create S3 Bucket
# -------------------------

terraform {
  backend "s3" {
    bucket         = "terraweek-state-umesh"
    key            = "dev/terraform.tfstate"
    region         = "ap-south-1"
    dynamodb_table = "terraweek-state-lock"
    encrypt        = true
  }
}
```

3. Run:
```bash
terraform init
```
Terraform will ask: "Do you want to copy existing state to the new backend?" -- say yes.

4. Verify:
   - Check the S3 bucket -- you should see `dev/terraform.tfstate`
     <img width="1760" height="433" alt="image" src="https://github.com/user-attachments/assets/ba128984-910a-4fb3-8ae1-979ac86a2aeb" />
   - Your local `terraform.tfstate` should now be empty or gone
   ```
   Local .tfstate is empty
   ```
   - Run `terraform plan` -- it should show no changes (state migrated correctly)
   ```
   No changes in local file
   ```
---

### Task 3: Test State Locking
State locking prevents two people from running `terraform apply` at the same time and corrupting the state.

1. Open **two terminals** in the same project directory
2. In Terminal 1, run:
```bash
terraform apply
```
3. While Terminal 1 is waiting for confirmation, in Terminal 2 run:
```bash
terraform plan
```
4. Terminal 2 should show a **lock error** with a Lock ID

**Document:** What is the error message? Why is locking critical for team environments?
```
Warning: Deprecated Parameter
│ 
│ The parameter "dynamodb_table" is deprecated. Use
│ parameter "use_lockfile" instead.
╵
╷
│ Error: Error acquiring the state lock
│ 
│ Error message: operation error DynamoDB: PutItem,
│ https response error StatusCode: 400, RequestID:
│ G1IQMFVIN3NJVL4GEMH7EJG39FVV4KQNSO5AEMVJF66Q9ASUAAJG,
│ ConditionalCheckFailedException: The conditional
│ request failed
│ Lock Info:
│   ID:        f8291ed0-016d-91b1-fb76-80b3b87b3fca
│   Path:      terraweek-state-umesh/dev/terraform.tfstate
│   Operation: OperationTypeApply
│   Who:       umesh@umesh-Aspire-A515-57G
│   Version:   1.15.2
│   Created:   2026-05-19 17:23:36.741602206 +0000 UTC
│   Info:      
│ 
│ 
│ Terraform acquires a state lock to protect the
│ state from being written
│ by multiple users at the same time. Please resolve
│ the issue above and try
│ again. For most commands, you can disable locking
│ with the "-lock=false"
│ flag, but this is not recommended.
```
5. After the test, if you get stuck with a stale lock:
```bash
terraform force-unlock <LOCK_ID>
```

---

### Task 4: Import an Existing Resource
Not everything starts with Terraform. Sometimes resources already exist in AWS and you need to bring them under Terraform management.

1. Manually create an S3 bucket in the AWS console -- name it `terraweek-import-test-<yourname>`
2. Write a `resource "aws_s3_bucket"` block in your config for this bucket (just the bucket name, nothing else)

main.tf
```
# -------------------------
# 11. Import existing S3 Bucket
# -------------------------


resource "aws_s3_bucket" "imported" {
  bucket = "terraweek-import-test-umesh"
}
```

3. Import it:
```bash
terraform import aws_s3_bucket.imported terraweek-import-test-<yourname>
```
4. Run `terraform plan`:
   - If you see "No changes" -- the import was perfect
   - If you see changes -- your config does not match reality. Update your config to match, then plan again until you get "No changes"

5. Run `terraform state list` -- the imported bucket should now appear alongside your other resources
<img width="1128" height="317" alt="image" src="https://github.com/user-attachments/assets/40c517b3-10ee-44ae-8584-e2e81706119a" />
<img width="1140" height="834" alt="image" src="https://github.com/user-attachments/assets/2b512500-db66-408a-9f62-32c9c0b8b626" />

**Document:** What is the difference between `terraform import` and creating a resource from scratch?
```
terraform import
----------------

Used for existing resources in AWS
Does NOT create anything
Only adds the resource to Terraform state
You must write the configuration manually

Creating resource (terraform apply)
-----------------------------------

Used for new resources
Terraform creates the resource in AWS
Config → Terraform builds infrastructure
```
---

### Task 5: State Surgery -- mv and rm
Sometimes you need to rename a resource or remove it from state without destroying it in AWS.

1. **Rename a resource in state:**
```bash
terraform state list                              # Note the current resource names
terraform state mv aws_s3_bucket.imported aws_s3_bucket.logs_bucket
```
Update your `.tf` file to match the new name. Run `terraform plan` -- it should show no changes.
<img width="1259" height="625" alt="image" src="https://github.com/user-attachments/assets/7ad95bab-aeae-418d-8bcd-28989bd0f25f" />

2. **Remove a resource from state (without destroying it):**
```bash
terraform state rm aws_s3_bucket.logs_bucket
```
Run `terraform plan` -- Terraform no longer knows about the bucket, but it still exists in AWS.
<img width="1258" height="767" alt="image" src="https://github.com/user-attachments/assets/0d57f883-6ee9-413f-a160-417162f999e5" />
<img width="1103" height="298" alt="image" src="https://github.com/user-attachments/assets/f74804ae-0ade-4830-b364-5cbc8216f009" />

3. **Re-import it** to bring it back:
```bash
terraform import aws_s3_bucket.logs_bucket terraweek-import-test-<yourname>
```
re-imported
<img width="1104" height="313" alt="image" src="https://github.com/user-attachments/assets/53f02593-5ea3-4005-849e-48119446ec1a" />

**Document:** When would you use `state mv` in a real project? When would you use `state rm`?
```
You use state mv when you want to change how Terraform organizes or names a resource without touching the real infrastructure in AWS
```
---

### Task 6: Simulate and Fix State Drift
State drift happens when someone changes infrastructure outside of Terraform -- through the AWS console, CLI, or another tool.

1. Apply your full config so everything is in sync
2. Go to the **AWS console** and manually:
   - Change the Name tag of your EC2 instance to `"ManuallyChanged"`
   - Change the instance type if it's stopped (or add a new tag)
3. Run:
```bash
terraform plan
```
You should see a **diff** -- Terraform detects that reality no longer matches the desired state.
<img width="1104" height="313" alt="image" src="https://github.com/user-attachments/assets/73870a86-2edc-469f-a6fe-13dcc6d4a929" />

4. You have two choices:
   - **Option A:** Run `terraform apply` to force reality back to match your config (reconcile)
   - **Option B:** Update your `.tf` files to match the manual change (accept the drift)

5. Choose Option A -- apply and verify the tags are restored.

6. Run `terraform plan` again -- it should show "No changes." Drift resolved.
<img width="1564" height="100" alt="image" src="https://github.com/user-attachments/assets/c72bfc61-cd82-46e4-9399-6266425dbfc9" />

**Document:** How do teams prevent state drift in production? (hint: restrict console access, use CI/CD for all changes)
```
1. Restrict AWS Console access
Only DevOps/IAM admins allowed
Developers use Terraform only

2. Use CI/CD pipelines
All infra changes go through GitHub Actions / Jenkins
No manual console changes allowed

3. Enable audit & monitoring
AWS CloudTrail logs all changes
Alerts on manual modifications

4. Use policy enforcement
AWS IAM policies or SCPs to block manual edits
Tools like Sentinel / OPA
```
---
