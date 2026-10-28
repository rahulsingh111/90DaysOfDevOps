# Day 61 — Introduction to Terraform and Your First AWS Infrastructure

## Task 1: Understand Infrastructure as Code

**1. What is IaC? Why does it matter in DevOps?**

Infrastructure as Code (IaC) means creating and managing infrastructure using code instead of manual configuration. It helps DevOps teams automate deployments, maintain consistency, and reduce human errors.

**2. What problems does IaC solve compared to manual AWS console setup?**

IaC reduces manual errors, saves time, ensures consistent environments, and allows infrastructure changes to be tracked and reviewed using Git. It also makes recreating infrastructure easier.

**3. Terraform vs CloudFormation, Ansible, and Pulumi**

- **Terraform:** Manages infrastructure across multiple cloud providers using declarative configuration.
- **CloudFormation:** AWS-native service for provisioning AWS resources.
- **Ansible:** Primarily automates server configuration, application deployment, and system management.
- **Pulumi:** Manages infrastructure using programming languages such as Python, TypeScript, and Go.

**4. What do declarative and cloud-agnostic mean?**

Declarative means defining the desired infrastructure state and letting Terraform determine the required changes. Cloud-agnostic means Terraform supports multiple cloud providers, although each provider has its own resource configurations.

## Task 3: Create an S3 Bucket

### What does `terraform init` download?

It downloads the required provider plugins, such as the AWS provider, and initializes the Terraform working directory.

### What does the `.terraform/` directory contain?

It contains downloaded provider plugins and other local Terraform working data needed to manage the infrastructure.

## Task 4: Add an EC2 Instance

### How does Terraform know the S3 bucket already exists and only the EC2 instance needs to be created?

Terraform compares the configuration with its state file, which tracks the resources it manages. Since the S3 bucket is already recorded and unchanged, Terraform plans to create only the new EC2 instance.

## Task 5: Understand the State File

### What information does the state file store?

It stores resource mappings, resource IDs, attributes, and metadata that Terraform uses to track and manage infrastructure.

### Why should you never manually edit the state file?

Manual edits can corrupt the state or cause Terraform to misunderstand the actual infrastructure. Terraform state commands should be used for state management.

### Why should the state file not be committed to Git?

It may contain sensitive information about infrastructure and resource attributes. Keeping it out of Git reduces the risk of exposing sensitive data.

## Task 6: Modify, Plan, and Destroy

### What do `~`, `+`, and `-` mean?

- `+` means a resource or value will be added.
- `-` means a resource or value will be removed.
- `~` means an existing resource or value will be modified in place.

### Is changing the EC2 Name tag an in-place update or destroy-and-recreate?

Changing only the EC2 `Name` tag is normally an in-place update. Terraform updates the tag without replacing the instance.

## Important Terraform Commands

- `terraform init` — Initializes the project and downloads providers.
- `terraform plan` — Shows the proposed infrastructure changes.
- `terraform apply` — Applies the changes and creates or updates resources.
- `terraform destroy` — Destroys managed resources.
- `terraform show` — Displays the current Terraform state in a readable format.
- `terraform state list` — Lists resources tracked in the state file.

