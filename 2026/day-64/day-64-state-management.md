# Day 64 — Terraform State Management and Remote Backends

**Challenge:** 90 Days of DevOps – TerraWeek  
**Topic:** Terraform State Management and Remote Backends  
**Terraform Version:** 1.14.0  
**AWS Provider:** 6.71.0  
**AWS Region:** ap-south-1  
**Project:** terraform-aws-infra

---

## 1. Overview

Terraform state is the source of truth that maps Terraform configuration to real-world infrastructure.

In this challenge, I explored:

- Terraform state inspection.
- S3 remote backend configuration.
- DynamoDB state locking.
- Importing existing AWS resources.
- State manipulation using `terraform state mv` and `terraform state rm`.
- Detecting and resolving infrastructure drift.

## 2. Local State vs Remote State

### Local State Architecture

```text
Terraform Configuration
          |
          v
      Terraform
          |
          v
   terraform.tfstate
    (Local Machine)
```

### Remote State Architecture

```text
Developer / CI Pipeline
          |
          v
      Terraform CLI
          |
          v
      AWS Backend
          |
     +----+-----+
     |          |
     v          v
    S3       DynamoDB
  State       Locking
  Storage
     |
     v
  Versioning
```

### Why Remote State?

- Centralized state storage.
- Team collaboration.
- State locking.
- State versioning and recovery.
- Reduced risk of losing local state files.

## 3. Task 1 — Inspect Terraform State

Commands used:

```bash
terraform show
terraform state list
terraform state show aws_instance.terra-iac-instance
terraform state show aws_vpc.my-terra-vpc
```

### Resources tracked

My Terraform state contained 11 addresses:

```text
data.aws_ami.amazon_linux
data.aws_availability_zones.available
aws_instance.terra-iac-instance
aws_internet_gateway.my-terra-gateway
aws_route_table.my-terra-route_table
aws_route_table_association.my-terra-route_table_association
aws_s3_bucket.logs_bucket
aws_s3_bucket.reference-bucket
aws_security_group.terraweek_sg
aws_subnet.my-terra-subnet
aws_vpc.my-terra-vpc
```

This includes 9 managed resources and 2 data sources.

### What does Terraform store for EC2?

Terraform state stores more attributes than those explicitly declared in the configuration.

Examples:

- Instance ID.
- AMI ID.
- Instance type.
- Availability zone.
- Subnet ID.
- VPC ID.
- Security group IDs.
- Public and private IP addresses.
- Network interface details.
- Root block device information.
- Tags.
- Instance ARN.

### What is the serial number?

The `serial` field represents the revision number of the Terraform state snapshot.

It increases whenever Terraform writes an updated state snapshot.

It helps Terraform identify stale state updates and maintain state consistency.

---

## 4. Task 2 — Configure S3 Remote Backend

Created an S3 bucket for Terraform state storage.

**Backend details:**

| Configuration | Value |
|---|---|
| Backend | S3 |
| Bucket | terraweek-state-rahul-6-11-26 |
| State Key | dev/terraform.tfstate |
| Region | ap-south-1 |
| Encryption | Enabled |
| Locking | DynamoDB |
| DynamoDB Table | terraweek-state-lock |

### Backend configuration

File: `backend.tf`

```hcl
terraform {
  backend "s3" {
    bucket         = "terraweek-state-rahul-6-11-26"
    key            = "dev/terraform.tfstate"
    region         = "ap-south-1"
    dynamodb_table = "terraweek-state-lock"
    encrypt        = true
  }
}
```

### Initialize Terraform

```bash
terraform init
```

Terraform initialized successfully.

Verified the backend configuration using:

```bash
cat .terraform/terraform.tfstate
```

Confirmed:

```json
{
  "backend": {
    "type": "s3",
    "config": {
      "bucket": "terraweek-state-rahul-6-11-26",
      "key": "dev/terraform.tfstate",
      "region": "ap-south-1",
      "dynamodb_table": "terraweek-state-lock",
      "encrypt": true
    }
  }
}
```

**Important:** `.terraform/terraform.tfstate` contains local backend metadata, not the actual infrastructure state stored in S3.

### Verify remote state

```bash
aws s3 ls s3://terraweek-state-rahul-6-11-26/dev/
```

Expected object:

```text
terraform.tfstate
```

Note: Verify the S3 object separately. Successful backend initialization alone does not prove that an existing local state was migrated.

---

## 5. Task 3 — State Locking

State locking prevents multiple Terraform operations from modifying the same state simultaneously.

### How it works

When Terraform starts an operation:

1. Terraform requests a state lock.
2. The backend establishes the lock.
3. Other operations must wait or fail while the lock is held.
4. Terraform releases the lock after completing the operation.

### Testing procedure

Open two terminals in the same project directory.

Terminal 1:

```bash
terraform apply
```

Leave it waiting for approval.

Terminal 2:

```bash
terraform plan
```

Observe whether the second operation waits or reports a state-lock error.

### Why is locking important?

Without locking, concurrent Terraform operations could overwrite each other's state updates.

This can lead to:

- State corruption.
- Inconsistent infrastructure tracking.
- Lost state updates.
- Unexpected infrastructure changes.

### Force unlock

If a lock becomes stale:

```bash
terraform force-unlock <LOCK_ID>
```

Only use this after confirming that no Terraform operation is still running.

**Note:** The two-terminal locking test and its screenshot still need to be captured.

---

## 6. Task 4 — Import Existing AWS Resources

Not all infrastructure is initially created using Terraform.

Sometimes resources already exist in AWS and need to be brought under Terraform management.

For this exercise, I used an existing S3 bucket.

**Bucket name:**

```text
terratest-61126
```

The bucket was confirmed to exist in AWS region `ap-south-1`.

### Terraform configuration

File: `s3.tf`

```hcl
resource "aws_s3_bucket" "name" {
  bucket = "terratest-61126"
}
```

### Import command

```bash
terraform import aws_s3_bucket.name terratest-61126
```

The resource was successfully imported and appeared in Terraform state.

### Verify import

```bash
terraform state list
```

Expected entry:

```text
aws_s3_bucket.name
```

### Difference between import and creation

**Terraform Import:**

- Brings an existing AWS resource into Terraform state.
- Does not create the resource.
- Requires a corresponding resource block.
- Does not automatically generate complete Terraform configuration.

**Terraform Apply:**

- Creates new infrastructure based on Terraform configuration.
- Updates existing infrastructure when required.

### Import syntax

```bash
terraform import <RESOURCE_ADDRESS> <RESOURCE_ID>
```

Example:

```bash
terraform import aws_s3_bucket.name terratest-61126
```

The Terraform resource name can be anything valid. It does not need to match the actual AWS bucket name.

---

## 7. Task 5 — State Surgery: MV and RM

Terraform provides commands to manipulate resource addresses in state without directly deleting the underlying AWS infrastructure.

### A. Rename a resource using state mv

Initially, the resource address was:

```text
aws_s3_bucket.name
```

Command:

```bash
terraform state mv aws_s3_bucket.name aws_s3_bucket.logs_bucket
```

The resource address was successfully changed.

Updated configuration:

```hcl
resource "aws_s3_bucket" "logs_bucket" {
  bucket = "terratest-61126"
}
```

Verify:

```bash
terraform state list
```

Expected:

```text
aws_s3_bucket.logs_bucket
```

**When to use state mv?**

- Renaming Terraform resources.
- Moving resources into modules.
- Refactoring Terraform configurations.
- Avoiding unnecessary infrastructure recreation.

### B. Remove a resource using state rm

Command:

```bash
terraform state rm aws_s3_bucket.logs_bucket
```

This removes the resource from Terraform state without deleting the actual S3 bucket.

Verify:

```bash
terraform state list
```

The resource should no longer appear.

**When to use state rm?**

- When Terraform should stop managing a resource.
- When transferring resource management to another Terraform state.
- When preparing for a controlled re-import.

Important: If the resource block remains in your configuration, a subsequent plan may propose creating the resource again.

### C. Re-import the resource

After removing the resource from state, it can be imported again:

```bash
terraform import aws_s3_bucket.logs_bucket terratest-61126
```

Verify:

```bash
terraform state list
```

Expected:

```text
aws_s3_bucket.logs_bucket
```

### Important distinction

| Command | Purpose |
|---|---|
| terraform import | Import existing infrastructure |
| terraform state mv | Change resource address |
| terraform state rm | Remove resource from state |
| terraform force-unlock | Release a stale state lock |

Neither `state mv` nor `state rm` directly deletes the AWS resource.

---

## 8. Task 6 — State Drift

State drift happens when infrastructure is modified outside Terraform.

For example:

- Someone changes an EC2 instance tag through AWS Console.
- Someone modifies the instance type.
- A security group rule is changed manually.
- An S3 bucket configuration is modified outside Terraform.

### Example: EC2 instance

My EC2 resource configuration:

```hcl
resource "aws_instance" "terra-iac-instance" {
  ami                         = data.aws_ami.amazon_linux.id
  instance_type               = var.instance_type
  vpc_security_group_ids      = [aws_security_group.terraweek_sg.id]
  associate_public_ip_address = true
  subnet_id                   = aws_subnet.my-terra-subnet.id

  lifecycle {
    create_before_destroy = true
  }

  tags = {
    Name = "${local.name_prefix}-server"
  }
}
```

### Simulating drift

1. Open AWS Management Console.
2. Navigate to EC2.
3. Select the instance.
4. Change its Name tag to:

```text
ManuallyChanged
```

5. Run:

```bash
terraform plan
```

Terraform should detect the difference between the actual AWS tag and the configured tag.

### Reconcile drift

I chose Option A: Restore the infrastructure to match Terraform configuration.

Run:

```bash
terraform apply
```

Then verify:

```bash
terraform plan
```

Expected output:

```text
No changes. Your infrastructure matches the configuration.
```

**Note:** Complete this exercise after resolving the EC2 subnet issue. The actual drift simulation and reconciliation still need verification.

### How teams prevent drift in production

- Restrict direct AWS Console access.
- Use CI/CD pipelines for infrastructure changes.
- Review Terraform plans before applying.
- Implement IAM least-privilege policies.
- Monitor infrastructure changes using AWS CloudTrail.
- Perform regular drift detection.
- Maintain versioned remote state.

---

## 9. Important Troubleshooting Notes

### A. Route Table CIDR Error

Initially, I configured:

```hcl
route {
  cidr_block = var.vpc_cidr
  gateway_id = aws_internet_gateway.my-terra-gateway.id
}
```

AWS returned an error because the destination CIDR matched the VPC's own CIDR.

Correct configuration:

```hcl
route {
  cidr_block = "0.0.0.0/0"
  gateway_id = aws_internet_gateway.my-terra-gateway.id
}
```

AWS automatically creates a local route for communication within the VPC.

### B. EC2 Subnet ID Not Found

Error:

```text
InvalidSubnetID.NotFound:
The subnet ID 'subnet-060c0c0398d7092df' does not exist
```

My EC2 configuration already referenced the Terraform subnet:

```hcl
subnet_id = aws_subnet.my-terra-subnet.id
```

The next step is to inspect the subnet state and verify that the subnet exists in the correct AWS account and region.

Commands:

```bash
terraform state show aws_subnet.my-terra-subnet
```

```bash
aws ec2 describe-subnets \
  --region ap-south-1 \
  --subnet-ids subnet-060c0c0398d7092df
```

Possible causes include:

- Subnet deleted manually.
- Stale subnet ID in Terraform state.
- Wrong AWS account.
- Provider region mismatch.

### C. S3 Import Region Mismatch

During troubleshooting, I discovered that:

- S3 bucket region: `ap-south-1`
- Terraform provider region: `us-east-1`

My provider configuration:

```hcl
provider "aws" {
  region = var.region
}
```

I corrected the region variable:

```hcl
region = "ap-south-1"
```

This highlights the difference between the S3 backend region and the AWS provider region.

---

## 10. Useful Terraform Commands

| Command | Description |
|---|---|
| `terraform show` | Display current state |
| `terraform state list` | List tracked resources |
| `terraform state show` | Inspect a resource |
| `terraform init` | Initialize Terraform |
| `terraform init -migrate-state` | Migrate existing state |
| `terraform import` | Import existing infrastructure |
| `terraform state mv` | Rename or move state addresses |
| `terraform state rm` | Remove resource tracking |
| `terraform plan` | Preview infrastructure changes |
| `terraform apply` | Apply infrastructure changes |
| `terraform plan -refresh-only` | Preview state refresh |
| `terraform apply -refresh-only` | Update state from remote infrastructure |
| `terraform force-unlock` | Release a stale lock |

---

## 11. Completion Checklist

- [x] Inspected Terraform state.
- [x] Identified managed resources and data sources.
- [x] Configured S3 remote backend.
- [x] Verified backend metadata.
- [x] Imported an existing S3 bucket.
- [x] Renamed a resource using `terraform state mv`.
- [ ] Verified local-to-S3 state migration.
- [ ] Tested DynamoDB state locking with two terminals.
- [ ] Completed state removal and re-import verification.
- [ ] Simulated EC2 drift.
- [ ] Reconciled drift and verified a no-changes plan.
- [ ] Captured required screenshots.

---

## 12. Key Learnings

1. Terraform state is critical for infrastructure management.
2. Remote backends provide centralized state storage.
3. State locking prevents concurrent operations from corrupting state.
4. Importing existing infrastructure avoids recreating resources.
5. `terraform state mv` helps refactor configurations without recreating infrastructure.
6. `terraform state rm` removes tracking without deleting the real resource.
7. Terraform detects drift by comparing actual infrastructure with the desired configuration.
8. Always review `terraform plan` before applying changes.

---