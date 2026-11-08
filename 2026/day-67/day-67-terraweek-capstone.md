# Day 67 – TerraWeek Capstone
## Multi-Environment AWS Infrastructure with Terraform Workspaces and Custom Modules

**Project:** terraweek-capstone  
**Technology:** Terraform, AWS, HCL  
**Environments:** Dev, Staging, Production

---

# 1. Project Overview

The objective is to build reusable AWS infrastructure using Terraform.

Instead of writing separate Terraform configurations for development, staging, and production, we use:

- Terraform workspaces to separate environments.
- Custom modules to organize reusable infrastructure.
- Variables to customize each environment.
- Outputs to share resource information between modules.
- Locals to define common configuration and naming conventions.
- Tags to identify and organize AWS resources.

### Architecture

One Terraform codebase manages three environments:

| Environment | VPC | EC2 | Purpose |
|---|---|---|---|
| Dev | 10.0.0.0/16 | t2.micro | Development |
| Staging | 10.1.0.0/16 | t2.small | Testing |
| Production | 10.2.0.0/16 | t3.small | Production |

Each environment gets its own VPC, subnet, security group, and EC2 instance.

---

# 2. Terraform Workspaces

## What are workspaces?

Terraform workspaces allow us to manage multiple instances of the same Terraform configuration.

For example:

- Dev workspace maintains development infrastructure.
- Staging workspace maintains staging infrastructure.
- Prod workspace maintains production infrastructure.

All three use the same Terraform code but maintain separate state.

## Important commands

```bash
# Initialize Terraform
terraform init

# Check current workspace
terraform workspace show

# Create workspaces
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod

# List workspaces
terraform workspace list

# Switch workspace
terraform workspace select dev

# Delete a workspace
terraform workspace delete dev
```

## What does terraform.workspace return?

`terraform.workspace` is a built-in Terraform expression that returns the name of the currently selected workspace.

Example:

```hcl
locals {
  environment = terraform.workspace
}
```

If the current workspace is `dev`, then:

```hcl
local.environment = "dev"
```

If the current workspace is `prod`, then:

```hcl
local.environment = "prod"
```

This allows us to dynamically configure resource names and tags.

## Where is workspace state stored?

With the local backend, Terraform stores non-default workspace state under:

```text
terraform.tfstate.d/
    dev/
        terraform.tfstate
    staging/
        terraform.tfstate
    prod/
        terraform.tfstate
```

The default workspace uses the root `terraform.tfstate`.

With an S3 backend, workspace state uses workspace-specific state paths.

**Important:** Workspaces provide separate state, but they are not a complete security boundary. For strongly isolated production environments, separate accounts, permissions, and backend configurations may be more appropriate.

---

# 3. Project Structure

Our project follows a modular Terraform structure.

```text
terraweek-capstone/
│
├── main.tf
├── variables.tf
├── outputs.tf
├── providers.tf
├── locals.tf
│
├── dev.tfvars
├── staging.tfvars
├── prod.tfvars
│
├── .gitignore
│
└── modules/
    │
    ├── vpc/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    │
    ├── security-group/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    │
    └── ec2-instance/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

## Purpose of each root file

| File | Responsibility |
|---|---|
| main.tf | Calls custom modules |
| variables.tf | Declares input variables |
| outputs.tf | Exposes useful resource information |
| providers.tf | AWS provider and backend configuration |
| locals.tf | Common local values and workspace configuration |
| dev.tfvars | Development values |
| staging.tfvars | Staging values |
| prod.tfvars | Production values |
| .gitignore | Excludes generated files and sensitive data |

## Why separate modules?

Modules help us:

1. Avoid repeating infrastructure code.
2. Reuse the same infrastructure across environments.
3. Keep configurations organized.
4. Make troubleshooting easier.
5. Maintain infrastructure consistently.

---

# 4. Understanding Terraform Modules

A Terraform module is a collection of Terraform configuration files grouped together to perform a particular task.

Our project has three custom modules.

## Module 1: VPC

Location:

```text
modules/vpc/
```

### Purpose

Creates the AWS networking infrastructure.

### Resources

- VPC
- Public subnet
- Internet gateway
- Route table
- Route table association

### variables.tf

```hcl
variable "cidr" {
  type = string
}

variable "public_subnet_cidr" {
  type = string
}

variable "environment" {
  type = string
}

variable "project_name" {
  type = string
}

variable "common_tags" {
  type    = map(string)
  default = {}
}
```

### main.tf

```hcl
resource "aws_vpc" "main" {
  cidr_block           = var.cidr
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = merge(var.common_tags, {
    Name = "${var.project_name}-${var.environment}-vpc"
  })
}

resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnet_cidr
  map_public_ip_on_launch = true

  tags = merge(var.common_tags, {
    Name = "${var.project_name}-${var.environment}-public-subnet"
  })
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = merge(var.common_tags, {
    Name = "${var.project_name}-${var.environment}-igw"
  })
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = merge(var.common_tags, {
    Name = "${var.project_name}-${var.environment}-public-rt"
  })
}

resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}
```

### outputs.tf

```hcl
output "vpc_id" {
  description = "VPC ID"
  value       = aws_vpc.main.id
}

output "subnet_id" {
  description = "Public subnet ID"
  value       = aws_subnet.public.id
}
```

### Important concepts

**Resource references**

```hcl
aws_vpc.main.id
```

This references the ID of the VPC resource named `main`.

**Resource dependency**

```hcl
vpc_id = aws_vpc.main.id
```

Terraform understands that the subnet depends on the VPC and creates the VPC first.

**Outputs**

Outputs expose information from a module so other modules can consume it.

For example:

```hcl
module.vpc.vpc_id
```

Returns the VPC ID exposed by the VPC module.

---

## Module 2: Security Group

Location:

```text
modules/security-group/
```

### Purpose

Controls inbound and outbound network traffic for EC2.

### variables.tf

```hcl
variable "vpc_id" {
  type = string
}

variable "ingress_ports" {
  type = list(number)
}

variable "environment" {
  type = string
}

variable "project_name" {
  type = string
}

variable "common_tags" {
  type    = map(string)
  default = {}
}
```

### main.tf

```hcl
resource "aws_security_group" "main" {
  name        = "${var.project_name}-${var.environment}-sg"
  description = "Security group for ${var.environment}"
  vpc_id      = var.vpc_id

  dynamic "ingress" {
    for_each = var.ingress_ports

    content {
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

  tags = merge(var.common_tags, {
    Name = "${var.project_name}-${var.environment}-sg"
  })
}
```

### outputs.tf

```hcl
output "sg_id" {
  description = "Security Group ID"
  value       = aws_security_group.main.id
}
```

### Important concepts

**Dynamic blocks**

Dynamic blocks allow us to generate repeated configuration blocks programmatically.

Instead of writing separate ingress rules for ports 22, 80, and 443, we use:

```hcl
dynamic "ingress" {
  for_each = var.ingress_ports
}
```

If:

```hcl
ingress_ports = [22, 80, 443]
```

Terraform creates three ingress rules.

**Ingress:** Incoming network traffic.

**Egress:** Outgoing network traffic.

**Security warning:** The example allows inbound traffic from `0.0.0.0/0`. This is not appropriate for unrestricted SSH access. Restrict port 22 to trusted IP addresses and consider separate ingress CIDR configuration.

---

## Module 3: EC2 Instance

Location:

```text
modules/ec2-instance/
```

### Purpose

Creates an EC2 instance using the VPC subnet and security group.

### variables.tf

```hcl
variable "ami_id" {
  type = string
}

variable "instance_type" {
  type    = string
  default = "t2.micro"
}

variable "subnet_id" {
  type = string
}

variable "security_group_ids" {
  type = list(string)
}

variable "environment" {
  type = string
}

variable "project_name" {
  type = string
}

variable "common_tags" {
  type    = map(string)
  default = {}
}
```

### main.tf

```hcl
resource "aws_instance" "main" {
  ami                    = var.ami_id
  instance_type          = var.instance_type
  subnet_id              = var.subnet_id
  vpc_security_group_ids = var.security_group_ids

  associate_public_ip_address = true

  tags = merge(var.common_tags, {
    Name = "${var.project_name}-${var.environment}-ec2"
  })
}
```

### outputs.tf

```hcl
output "instance_id" {
  description = "EC2 Instance ID"
  value       = aws_instance.main.id
}

output "public_ip" {
  description = "EC2 Public IP"
  value       = aws_instance.main.public_ip
}
```

### Important concepts

- AMI: Amazon Machine Image used to launch an EC2 instance.
- Instance type: Defines compute capacity.
- Subnet ID: Determines where the instance is launched.
- Security group IDs: Defines the instance's network access.
- Public IP: Allows public IPv4 addressing when configured.

An EC2 instance also needs a valid AMI for the selected AWS region.

---

# 5. Workspace-Aware Configuration

We use locals to define environment-specific values.

## locals.tf

```hcl
locals {
  environment = terraform.workspace

  name_prefix = "${var.project_name}-${local.environment}"

  common_tags = {
    Project     = var.project_name
    Environment = local.environment
    ManagedBy   = "Terraform"
    Workspace   = terraform.workspace
  }
}
```

### What are locals?

Locals are named expressions that help avoid repeating the same values throughout Terraform configurations.

For example:

```hcl
environment = terraform.workspace
```

Automatically identifies the current workspace.

```hcl
name_prefix = "${var.project_name}-${local.environment}"
```

Combines the project name and environment.

If the project name is `terraweek` and workspace is `dev`, the result is:

```text
terraweek-dev
```

### What are common tags?

Tags help identify AWS resources.

Example:

```text
Project     = terraweek
Environment = dev
ManagedBy   = Terraform
Workspace   = dev
```

We use `merge()` to combine common tags with resource-specific tags.

Example:

```hcl
tags = merge(var.common_tags, {
  Name = "terraweek-dev-vpc"
})
```

Note: Route table associations do not support resource tags.

---

# 6. Root Module Configuration

The root module connects all three custom modules.

## Root variables.tf

```hcl
variable "project_name" {
  type    = string
  default = "terraweek"
}

variable "vpc_cidr" {
  type = string
}

variable "subnet_cidr" {
  type = string
}

variable "instance_type" {
  type = string
}

variable "ingress_ports" {
  type    = list(number)
  default = [22, 80]
}

variable "ami_id" {
  description = "AMI ID for the selected AWS region"
  type        = string
}
```

## Root main.tf

### Module 1: VPC

```hcl
module "vpc" {
  source = "./modules/vpc"

  cidr               = var.vpc_cidr
  public_subnet_cidr = var.subnet_cidr
  environment        = local.environment
  project_name       = var.project_name
  common_tags        = local.common_tags
}
```

### Module 2: Security Group

```hcl
module "security_group" {
  source = "./modules/security-group"

  vpc_id        = module.vpc.vpc_id
  ingress_ports = var.ingress_ports
  environment   = local.environment
  project_name  = var.project_name
  common_tags   = local.common_tags
}
```

### Module 3: EC2

```hcl
module "ec2_instance" {
  source = "./modules/ec2-instance"

  ami_id             = var.ami_id
  instance_type      = var.instance_type
  subnet_id          = module.vpc.subnet_id
  security_group_ids = [module.security_group.sg_id]
  environment        = local.environment
  project_name       = var.project_name
  common_tags        = local.common_tags
}
```

## How do modules communicate?

The data flow is:

```text
                 ROOT MODULE
                      |
                      v
                 VPC MODULE
                      |
              +-------+-------+
              |               |
           vpc_id          subnet_id
              |               |
              v               |
        SECURITY GROUP        |
              |               |
            sg_id             |
              |               |
              +-------+-------+
                      |
                      v
                 EC2 MODULE
```

The important syntax is:

```hcl
module.vpc.vpc_id
```

Meaning:

- `module`: Access a child module.
- `vpc`: Module block name in root main.tf.
- `vpc_id`: Output declared in the child module.

The output name must match exactly.

---

# 7. Environment-Specific tfvars

Terraform variable files allow us to provide different values without changing the module code.

## dev.tfvars

```hcl
vpc_cidr      = "10.0.0.0/16"
subnet_cidr   = "10.0.1.0/24"
instance_type = "t2.micro"
ingress_ports = [22, 80]

ami_id = "ami-REPLACE_WITH_VALID_AMI"
```

## staging.tfvars

```hcl
vpc_cidr      = "10.1.0.0/16"
subnet_cidr   = "10.1.1.0/24"
instance_type = "t2.small"
ingress_ports = [22, 80, 443]

ami_id = "ami-REPLACE_WITH_VALID_AMI"
```

## prod.tfvars

```hcl
vpc_cidr      = "10.2.0.0/16"
subnet_cidr   = "10.2.1.0/24"
instance_type = "t3.small"
ingress_ports = [80, 443]

ami_id = "ami-REPLACE_WITH_VALID_AMI"
```

### Environment comparison

| Configuration | Dev | Staging | Prod |
|---|---|---|---|
| VPC CIDR | 10.0.0.0/16 | 10.1.0.0/16 | 10.2.0.0/16 |
| Subnet | 10.0.1.0/24 | 10.1.1.0/24 | 10.2.1.0/24 |
| Instance | t2.micro | t2.small | t3.small |
| SSH (22) | Yes | Yes | No |
| HTTP (80) | Yes | Yes | Yes |
| HTTPS (443) | No | Yes | Yes |

The different CIDR ranges prevent these networks from overlapping with each other.

Note: The example's ingress rules use public access for all configured ports. Production should also have appropriate source CIDR restrictions.

---

# 8. Terraform Outputs

Root outputs expose information returned by the child modules.

## outputs.tf

```hcl
output "vpc_id" {
  value = module.vpc.vpc_id
}

output "subnet_id" {
  value = module.vpc.subnet_id
}

output "security_group_id" {
  value = module.security_group.sg_id
}

output "instance_id" {
  value = module.ec2_instance.instance_id
}

output "public_ip" {
  value = module.ec2_instance.public_ip
}
```

Run:

```bash
terraform output
```

This displays the output values for the currently selected workspace.

To display an individual output:

```bash
terraform output public_ip
```

---

# 9. Terraform Deployment Workflow

## Step 1: Initialize

```bash
terraform init
```

Downloads provider plugins and initializes the backend.

## Step 2: Format

```bash
terraform fmt -recursive
```

Formats Terraform files consistently.

## Step 3: Validate

```bash
terraform validate
```

Checks Terraform configuration syntax and internal consistency.

## Step 4: Select workspace

```bash
terraform workspace select dev
```

## Step 5: Plan

```bash
terraform plan -var-file="dev.tfvars"
```

Shows the proposed infrastructure changes without applying them.

## Step 6: Apply

```bash
terraform apply -var-file="dev.tfvars"
```

Creates or updates infrastructure after approval.

## Step 7: Verify

```bash
terraform output
```

Check the AWS console for the created resources.

### Repeat for staging

```bash
terraform workspace select staging

terraform plan -var-file="staging.tfvars"

terraform apply -var-file="staging.tfvars"
```

### Repeat for production

```bash
terraform workspace select prod

terraform plan -var-file="prod.tfvars"

terraform apply -var-file="prod.tfvars"
```

---

# 10. Troubleshooting: Errors Encountered During Implementation

These are particularly important because they demonstrate how Terraform module inputs, outputs, and resource references work.

## Error 1: Value for undeclared variable

Error:

```text
Warning: Value for undeclared variable

The root module does not declare a variable named "ami_id"
```

### Why?

The `dev.tfvars` file contained:

```hcl
ami_id = "ami-xxxxxxxx"
```

But root `variables.tf` did not declare `ami_id`.

### Fix

Add:

```hcl
variable "ami_id" {
  type = string
}
```

### Lesson

Every variable supplied through a root `.tfvars` file must be declared in the root module.

Child module variables do not automatically become root module variables.

---

## Error 2: Unsupported argument common_tags

Error:

```text
Error: Unsupported argument

An argument named "common_tags" is not expected here.
```

### Why?

The root module passed:

```hcl
common_tags = local.common_tags
```

But the child module did not declare a variable called `common_tags`.

### Fix

Add this to the relevant child module's `variables.tf`:

```hcl
variable "common_tags" {
  type    = map(string)
  default = {}
}
```

### Lesson

Whenever you pass an argument to a module, the child module must declare the corresponding input variable.

---

## Error 3: Unsupported attribute vpc_id

Error:

```text
This object does not have an attribute named "vpc_id".
```

### Why?

Root main.tf referenced:

```hcl
module.vpc.vpc_id
```

But the VPC child module did not expose an output named `vpc_id`.

### Fix

In `modules/vpc/outputs.tf`:

```hcl
output "vpc_id" {
  value = aws_vpc.main.id
}
```

### Lesson

A resource ID inside a child module is not automatically available to the root module.

It must be exposed through an output.

---

## Error 4: Reference to undeclared resource

Error:

```text
A managed resource "aws_vpc" "main"
has not been declared in module.vpc.
```

### Why?

The output referenced:

```hcl
aws_vpc.main.id
```

But the resource was either missing or declared using a different resource name.

### Fix

Ensure the resource exists:

```hcl
resource "aws_vpc" "main" {
  cidr_block = var.cidr
}
```

Similarly:

```hcl
resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id
  cidr_block = var.public_subnet_cidr
}
```

### Lesson

Terraform resource references follow this pattern:

```hcl
resource_type.resource_name.attribute
```

For example:

```hcl
aws_vpc.main.id
aws_subnet.public.id
aws_instance.main.public_ip
```

The resource type and resource name must match the actual declaration exactly.

---

# 11. Terraform Best Practices

## 1. File structure

Keep providers, variables, outputs, locals, and module calls logically separated.

## 2. State management

Use a remote backend for team environments.

Recommended practices:

- Enable encryption at rest.
- Enable S3 bucket versioning.
- Restrict state access.
- Configure state locking using S3 lockfiles or an appropriate supported locking mechanism.
- Never commit state files.

## 3. Variables

Avoid hardcoding environment-specific values.

Use:

- `variables.tf`
- Environment-specific `.tfvars`
- Variable validation blocks
- Sensitive variables where appropriate

## 4. Modules

Build focused modules with clear responsibilities.

Every reusable module should have:

- main.tf
- variables.tf
- outputs.tf

## 5. Workspaces

Use `terraform.workspace` to identify the active workspace.

Always verify the current workspace before deployment:

```bash
terraform workspace show
```

## 6. Security

Use `.gitignore` to exclude generated and sensitive files.

Encrypt remote state and restrict access to it.

Avoid exposing SSH to the public internet.

## 7. Deployment commands

Use the standard workflow:

```bash
terraform fmt -recursive
terraform validate
terraform plan -var-file="dev.tfvars"
terraform apply -var-file="dev.tfvars"
```

Never blindly apply without reviewing the plan.

## 8. Tagging

Tag AWS resources with:

- Project
- Environment
- ManagedBy
- Workspace

## 9. Naming conventions

Use consistent resource names:

```text
<project>-<environment>-<resource>
```

Example:

```text
terraweek-dev-vpc
terraweek-staging-sg
terraweek-prod-ec2
```

## 10. Cleanup

Destroy non-production infrastructure when it is no longer needed.

Always select the correct workspace before destroying infrastructure.

---

# 12. Destroy All Environments

After verification, clean up all three environments.

### Production

```bash
terraform workspace select prod

terraform destroy -var-file="prod.tfvars"
```

### Staging

```bash
terraform workspace select staging

terraform destroy -var-file="staging.tfvars"
```

### Development

```bash
terraform workspace select dev

terraform destroy -var-file="dev.tfvars"
```

### Delete workspaces

Switch to default first:

```bash
terraform workspace select default
```

Then:

```bash
terraform workspace delete dev
terraform workspace delete staging
terraform workspace delete prod
```

Verify the AWS console to ensure the resources have been removed.

---

# 13. TerraWeek: Seven-Day Learning Summary

| Day | Concepts learned |
|---|---|
| 61 | Infrastructure as Code, HCL, init, plan, apply, destroy, state basics |
| 62 | Providers, resources, dependencies, lifecycle |
| 63 | Variables, outputs, data sources, locals, functions |
| 64 | Remote backend, locking, import, drift |
| 65 | Custom modules, registry modules, versioning |
| 66 | EKS with modules, real-world provisioning |
| 67 | Workspaces, multi-environment infrastructure, capstone |

---

# 14. Important Questions for Revision

### Workspaces

1. What is a Terraform workspace?
2. How does `terraform.workspace` work?
3. How does Terraform maintain separate state for each workspace?
4. What is the difference between workspaces and separate environment directories?

### Modules

5. What is a Terraform module?
6. What is the difference between root and child modules?
7. Why do we need `variables.tf` in a child module?
8. How does a child module expose resource IDs to the root module?
9. What is the purpose of `source` in a module block?

### Terraform configuration

10. What is the difference between variables and locals?
11. What is the purpose of `.tfvars` files?
12. What does `merge()` do?
13. What is a dynamic block?
14. What is the difference between ingress and egress?

### Deployment

15. What does `terraform init` do?
16. What is the difference between `terraform plan` and `terraform apply`?
17. What happens when you switch workspaces?
18. Why should we review a plan before applying?

---

# 15. Final Takeaways

The biggest lesson from this capstone is how Terraform components work together.

**Workspaces** allow the same configuration to manage multiple environments.

**Modules** allow us to reuse infrastructure code.

**Variables** allow us to customize infrastructure.

**Outputs** allow modules to communicate.

**Locals** allow us to define reusable expressions.

**State** helps Terraform track the infrastructure it manages.

The overall architecture is:

```text
One Terraform Codebase
        |
        v
Terraform Workspace
        |
        +------ dev
        |
        +------ staging
        |
        +------ prod
        |
        v
Root Module
        |
        +------ VPC Module
        |
        +------ Security Group Module
        |
        +------ EC2 Module
        |
        v
AWS Infrastructure
```

**Core takeaway:**

One reusable Terraform codebase can provision multiple environments with different infrastructure configurations, while maintaining consistent architecture and resource management.

---

**Project:** TerraWeek Day 67 Capstone  
**Learning objective:** Build reusable, workspace-aware AWS infrastructure using Terraform custom modules.
