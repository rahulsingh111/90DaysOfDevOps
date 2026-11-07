# Day 65 – Terraform Modules: Build Reusable Infrastructure

**Challenge:** 90 Days of DevOps – Day 65  
**Topic:** Terraform Modules  
**Category:** Infrastructure as Code (IaC) | AWS | Terraform

---

## 1. Objective

The goal of this exercise is to understand Terraform modules and learn how to build reusable infrastructure components.

Previously, we managed infrastructure using a single `main.tf` file. While this works for small projects, maintaining large infrastructure configurations in one file becomes difficult.

Terraform modules solve this problem by allowing us to organize infrastructure into reusable components.

Think of Terraform modules like functions in programming:

- Write infrastructure code once.
- Pass different inputs whenever required.
- Reuse the same module across multiple environments.
- Maintain infrastructure more efficiently.

### What I built

1. A custom EC2 module from scratch.
2. A custom Security Group module.
3. Connected the Security Group module with the EC2 module.
4. Deployed two EC2 instances using the same custom module.
5. Used the official AWS VPC module from Terraform Registry.
6. Learned module versioning and module best practices.

---

## 2. Understanding Terraform Modules

A Terraform module is a collection of `.tf` files stored in a directory.

Every Terraform configuration starts with a root module. Other modules called by the root module are known as child modules.

### Root Module vs Child Module

| Root Module | Child Module |
|---|---|
| Main entry point of Terraform configuration | Reusable component called by another module |
| Contains provider configuration | Usually does not configure providers |
| Calls other modules | Defines specific resources |
| Manages overall infrastructure | Manages a particular infrastructure component |
| Executed directly using Terraform commands | Executed through the root module |

**Example:**

The root module manages the overall infrastructure:

- VPC
- Subnets
- Security Groups
- EC2 instances

Child modules handle individual components:

- EC2 module creates EC2 instances.
- Security Group module creates firewall rules.

### Terraform Module Architecture

```text
terraform-modules/
│
├── main.tf
├── variables.tf
├── outputs.tf
├── providers.tf
│
└── modules/
    │
    ├── ec2-instance/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    │
    └── security-group/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

**Important:** The root module is the directory where Terraform commands are executed. Child modules are called using the `module` block.

---

## 3. Task 1 – Create the Module Structure

Create the directories and files:

```bash
mkdir -p terraform-modules/modules/ec2-instance
mkdir -p terraform-modules/modules/security-group

cd terraform-modules

touch main.tf
touch variables.tf
touch outputs.tf
touch providers.tf

touch modules/ec2-instance/main.tf
touch modules/ec2-instance/variables.tf
touch modules/ec2-instance/outputs.tf

touch modules/security-group/main.tf
touch modules/security-group/variables.tf
touch modules/security-group/outputs.tf
```

Verify the directory structure:

```bash
tree terraform-modules
```

At this point, we have created the basic structure for a modular Terraform project.

---

## 4. Task 2 – Build a Custom EC2 Module

The EC2 module is responsible for creating EC2 instances.

Instead of defining `aws_instance` multiple times in the root configuration, we define it once inside our custom module.

### 4.1 variables.tf

File: `modules/ec2-instance/variables.tf`

```hcl
variable "ami_id" {
  description = "AMI ID for the EC2 instance"
  type        = string
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

variable "subnet_id" {
  description = "Subnet ID where EC2 will be deployed"
  type        = string
}

variable "security_group_ids" {
  description = "List of security group IDs"
  type        = list(string)
}

variable "instance_name" {
  description = "Name of the EC2 instance"
  type        = string
}

variable "tags" {
  description = "Additional resource tags"
  type        = map(string)
  default     = {}
}
```

**Explanation:**

Variables allow us to customize the EC2 instance without modifying the module's resource configuration.

For example, we can change:

- AMI ID
- Instance type
- Subnet
- Security Groups
- Instance name
- Tags

### 4.2 main.tf

File: `modules/ec2-instance/main.tf`

```hcl
resource "aws_instance" "this" {
  ami                    = var.ami_id
  instance_type          = var.instance_type
  subnet_id              = var.subnet_id
  vpc_security_group_ids = var.security_group_ids

  tags = merge(
    var.tags,
    {
      Name = var.instance_name
    }
  )
}
```

**Explanation:**

- `aws_instance.this` creates the EC2 instance.
- Resource properties are populated using input variables.
- `merge()` combines additional tags with the instance Name tag.
- The module can create multiple instances with different configurations.

The Name tag is explicitly set after the additional tags, ensuring that the module's `instance_name` takes precedence.

### 4.3 outputs.tf

File: `modules/ec2-instance/outputs.tf`

```hcl
output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.this.id
}

output "public_ip" {
  description = "Public IP of the EC2 instance"
  value       = aws_instance.this.public_ip
}

output "private_ip" {
  description = "Private IP of the EC2 instance"
  value       = aws_instance.this.private_ip
}
```

**Why outputs?**

Outputs allow the root module to access information generated by child modules.

For example:

```hcl
module.web_server.instance_id
module.web_server.public_ip
```

---

## 5. Task 3 – Build a Custom Security Group Module

Next, we create a reusable Security Group module.

A Security Group acts as a virtual firewall controlling inbound and outbound traffic for AWS resources.

### 5.1 variables.tf

File: `modules/security-group/variables.tf`

```hcl
variable "vpc_id" {
  description = "VPC ID where the security group will be created"
  type        = string
}

variable "sg_name" {
  description = "Security group name"
  type        = string
}

variable "ingress_ports" {
  description = "List of allowed inbound TCP ports"
  type        = list(number)
  default     = [22, 80]
}

variable "tags" {
  description = "Additional resource tags"
  type        = map(string)
  default     = {}
}

variable "ingress_cidr_blocks" {
  description = "CIDR ranges allowed to access the security group"
  type        = list(string)
  default     = ["0.0.0.0/0"]
}
```

The additional `ingress_cidr_blocks` variable allows us to control which IP addresses can access the instances.

For actual deployments, restrict SSH access to your own public IP address using a `/32` CIDR rather than allowing the entire internet.

### 5.2 main.tf

File: `modules/security-group/main.tf`

```hcl
resource "aws_security_group" "this" {
  name        = var.sg_name
  description = "Managed by Terraform"
  vpc_id      = var.vpc_id

  dynamic "ingress" {
    for_each = var.ingress_ports

    content {
      description = "Allow TCP port ${ingress.value}"
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"

      cidr_blocks = var.ingress_cidr_blocks
    }
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(
    var.tags,
    {
      Name = var.sg_name
    }
  )
}
```

### Understanding Dynamic Blocks

This is the first time we are using Terraform's `dynamic` block.

A dynamic block generates repeated nested configuration blocks from a collection.

Example:

```hcl
dynamic "ingress" {
  for_each = var.ingress_ports

  content {
    from_port = ingress.value
    to_port   = ingress.value
    protocol  = "tcp"
  }
}
```

If we pass:

```hcl
ingress_ports = [22, 80, 443]
```

Terraform generates three ingress rules:

| Port | Purpose |
|---|---|
| 22 | SSH |
| 80 | HTTP |
| 443 | HTTPS |

Instead of writing three separate ingress blocks, we write one dynamic block.

**Remember:**

- `for_each` iterates through the collection.
- `ingress.value` accesses the current item.
- `content` defines the generated nested block.

### 5.3 outputs.tf

File: `modules/security-group/outputs.tf`

```hcl
output "sg_id" {
  description = "Security Group ID"
  value       = aws_security_group.this.id
}
```

The root module can now access the Security Group ID using:

```hcl
module.web_sg.sg_id
```

---

## 6. Task 4 – Configure the Root Module

Now we connect everything from the root module.

The root module will:

1. Configure AWS.
2. Create the VPC using the official registry module.
3. Create the Security Group using our custom module.
4. Create two EC2 instances using the same custom EC2 module.

### 6.1 providers.tf

File: `providers.tf`

```hcl
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}
```

### 6.2 variables.tf

File: `variables.tf`

```hcl
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "ap-south-1"
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

variable "allowed_ingress_cidrs" {
  description = "CIDR ranges allowed to access the instances"
  type        = list(string)

  # Learning example only.
  # Restrict SSH access before deploying in a real environment.
  default = ["0.0.0.0/0"]
}
```

### 6.3 Root main.tf

File: `main.tf`

```hcl
locals {
  common_tags = {
    Project     = "TerraWeek"
    Environment = "Learning"
    ManagedBy   = "Terraform"
  }
}

# Find the latest Amazon Linux 2023 AMI

data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-2023.*-x86_64"]
  }

  filter {
    name   = "architecture"
    values = ["x86_64"]
  }

  filter {
    name   = "root-device-type"
    values = ["ebs"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

# VPC - Official Terraform Registry Module

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"

  name = "terraweek-vpc"
  cidr = "10.0.0.0/16"

  azs = [
    "ap-south-1a",
    "ap-south-1b"
  ]

  public_subnets = [
    "10.0.1.0/24",
    "10.0.2.0/24"
  ]

  private_subnets = [
    "10.0.3.0/24",
    "10.0.4.0/24"
  ]

  enable_nat_gateway     = false
  enable_dns_hostnames   = true
  enable_dns_support     = true
  map_public_ip_on_launch = true

  tags = local.common_tags
}

# Security Group - Custom Module

module "web_sg" {
  source = "./modules/security-group"

  vpc_id  = module.vpc.vpc_id
  sg_name = "terraweek-web-sg"

  ingress_ports = [22, 80, 443]

  ingress_cidr_blocks = var.allowed_ingress_cidrs

  tags = local.common_tags
}

# EC2 Instance - Web Server

module "web_server" {
  source = "./modules/ec2-instance"

  ami_id             = data.aws_ami.amazon_linux.id
  instance_type      = var.instance_type
  subnet_id          = module.vpc.public_subnets[0]
  security_group_ids = [module.web_sg.sg_id]

  instance_name = "terraweek-web"

  tags = local.common_tags
}

# EC2 Instance - API Server

module "api_server" {
  source = "./modules/ec2-instance"

  ami_id             = data.aws_ami.amazon_linux.id
  instance_type      = var.instance_type
  subnet_id          = module.vpc.public_subnets[0]
  security_group_ids = [module.web_sg.sg_id]

  instance_name = "terraweek-api"

  tags = local.common_tags
}
```

**Important observations:**

- Both EC2 instances use the same custom module.
- Each module call has different input values.
- Both instances share the same Security Group.
- Both instances are deployed in the same public subnet.
- The VPC is created using an official public registry module.

The VPC module creates public and private subnets. NAT Gateway is disabled to avoid NAT Gateway charges.

The private subnets do not have internet access through a NAT Gateway in this configuration. The EC2 instances are deployed in public subnets.

### 6.4 Root outputs.tf

File: `outputs.tf`

```hcl
output "vpc_id" {
  description = "VPC ID"
  value       = module.vpc.vpc_id
}

output "security_group_id" {
  description = "Web Security Group ID"
  value       = module.web_sg.sg_id
}

output "web_server_id" {
  description = "Web Server Instance ID"
  value       = module.web_server.instance_id
}

output "web_server_ip" {
  description = "Web Server Public IP"
  value       = module.web_server.public_ip
}

output "api_server_id" {
  description = "API Server Instance ID"
  value       = module.api_server.instance_id
}

output "api_server_ip" {
  description = "API Server Public IP"
  value       = module.api_server.public_ip
}
```

Now the root module can expose outputs from all its child modules.

---

## 7. Task 5 – Use a Public Terraform Registry Module

Terraform Registry provides reusable modules created by HashiCorp, AWS partners, and the community.

Instead of manually writing all VPC resources, we use:

**Module:** `terraform-aws-modules/vpc/aws`

**Registry:** https://registry.terraform.io/modules/terraform-aws-modules/vpc/aws/latest

### Why use a registry module?

A VPC involves several resources:

- VPC
- Internet Gateway
- Public subnets
- Private subnets
- Route tables
- Route table associations
- Network ACLs
- Default Security Group

Writing and maintaining all these resources manually takes time.

A registry module packages the functionality into a reusable component.

### Module Configuration

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"

  name = "terraweek-vpc"
  cidr = "10.0.0.0/16"

  azs = ["ap-south-1a", "ap-south-1b"]

  public_subnets = [
    "10.0.1.0/24",
    "10.0.2.0/24"
  ]

  private_subnets = [
    "10.0.3.0/24",
    "10.0.4.0/24"
  ]

  enable_nat_gateway      = false
  enable_dns_hostnames    = true
  enable_dns_support      = true
  map_public_ip_on_launch = true

  tags = local.common_tags
}
```

### Important: Accessing Registry Module Outputs

Once the VPC module is created, we can access its outputs.

VPC ID:

```hcl
module.vpc.vpc_id
```

Public subnet IDs:

```hcl
module.vpc.public_subnets
```

First public subnet:

```hcl
module.vpc.public_subnets[0]
```

We use these outputs in our custom modules:

```hcl
vpc_id = module.vpc.vpc_id
```

```hcl
subnet_id = module.vpc.public_subnets[0]
```

### Where Does Terraform Download Registry Modules?

After running:

```bash
terraform init
```

Terraform downloads registry modules into:

```text
.terraform/modules/
```

Check the downloaded modules:

```bash
ls -la .terraform/modules/
```

You can also inspect:

```bash
cat .terraform/modules/modules.json
```

**Important:** The `.terraform` directory contains downloaded modules and other initialization data. It should generally not be committed to GitHub.

---

## 8. Task 6 – Module Versioning

Versioning is important because infrastructure modules can change over time.

Terraform allows us to specify version constraints for registry modules.

### Exact Version

```hcl
version = "= 5.1.0"
```

Downloads exactly version 5.1.0.

### Pessimistic Version Constraint

```hcl
version = "~> 5.0"
```

Allows compatible versions from 5.0.0 up to, but not including, 6.0.0.

### Version Range

```hcl
version = ">= 5.0, < 6.0"
```

Allows versions greater than or equal to 5.0 and below 6.0.

### Upgrade Modules

```bash
terraform init -upgrade
```

This checks for newer module versions permitted by the configured constraints and updates dependency selections.

**Best practice:** Use a deliberate version constraint in production rather than blindly upgrading dependencies.

---

## 9. Terraform Execution Workflow

After completing all the files, execute the following commands from the root module directory.

### Step 1: Initialize Terraform

```bash
terraform init
```

This command:

- Initializes the working directory.
- Downloads the AWS provider.
- Downloads registry modules.
- Initializes local module references.

### Step 2: Format Terraform Files

```bash
terraform fmt -recursive
```

Formats Terraform files in the root module and child module directories.

### Step 3: Validate Configuration

```bash
terraform validate
```

Checks whether the Terraform configuration is syntactically valid and internally consistent.

### Step 4: Generate Execution Plan

```bash
terraform plan
```

Displays the infrastructure changes Terraform intends to make.

Review the plan before applying it.

### Step 5: Apply Configuration

```bash
terraform apply
```

Creates the infrastructure after approval.

### Step 6: Check Outputs

```bash
terraform output
```

Displays the VPC ID, Security Group ID, and EC2 instance details.

### Step 7: Check Terraform State

```bash
terraform state list
```

Example resource addresses will look similar to:

```text
module.vpc.aws_vpc.this[0]
module.web_sg.aws_security_group.this
module.web_server.aws_instance.this
module.api_server.aws_instance.this
```

Actual resource addresses depend on the registry module version and its implementation.

**Understanding module prefixes:**

| Prefix | Meaning |
|---|---|
| `module.vpc` | Registry VPC module |
| `module.web_sg` | Custom Security Group module |
| `module.web_server` | Web EC2 module instance |
| `module.api_server` | API EC2 module instance |

Terraform state identifies resources by their module path.

### Step 8: Destroy Infrastructure

After verification:

```bash
terraform destroy
```

This removes the infrastructure managed by the root module.

Always review the destruction plan before confirming.

---

## 10. Verify the Deployment

After applying the configuration, verify the resources in AWS.

### EC2 Verification

Navigate to:

AWS Console → EC2 → Instances

Expected instances:

| Instance Name | Instance Type | Security Group |
|---|---|---|
| terraweek-web | t2.micro | terraweek-web-sg |
| terraweek-api | t2.micro | terraweek-web-sg |

Both instances should:

- Be in the same VPC.
- Use the same Security Group.
- Have different Name tags.
- Be deployed using the same custom EC2 module.
- Have public IP addresses assigned.

### Security Group Verification

Navigate to:

AWS Console → EC2 → Security Groups

Expected inbound rules:

| Protocol | Port | Purpose |
|---|---:|---|
| TCP | 22 | SSH |
| TCP | 80 | HTTP |
| TCP | 443 | HTTPS |

Outbound traffic:

- All traffic allowed.

**Security note:** The example CIDR configuration allows access from `0.0.0.0/0`. Restrict SSH port 22 to your trusted public IP before deploying. The example is for learning, not a production security baseline.

### VPC Verification

Navigate to:

AWS Console → VPC → Your VPCs

Verify:

- VPC CIDR: `10.0.0.0/16`
- Two public subnets.
- Two private subnets.
- Internet Gateway attached.
- Public route tables configured.

---

## 11. Screenshot – EC2 Instances

Add your actual AWS Console screenshot here after deployment.

Suggested screenshot:

**AWS Console → EC2 → Instances**

The screenshot should clearly show:

- `terraweek-web`
- `terraweek-api`
- Both instance states as Running.
- Instance types.
- Availability Zones.
- Public IP addresses.

Save the screenshot in your repository:

```text
2026/
└── day-65/
    ├── day-65-modules.md
    └── screenshots/
        └── ec2-instances.png
```

Embed it in this document:

```markdown
![Day 65 - EC2 Instances](screenshots/ec2-instances.png)
```

*Note: Add your own deployment screenshot; do not use a sample image as proof of deployment.*

---

## 12. Comparison – Handwritten VPC vs Registry VPC Module

One of the major advantages of Terraform modules is that they reduce repetitive configuration.

| Feature | Handwritten VPC | Registry VPC Module |
|---|---|---|
| VPC creation | Manually configured | Module-managed |
| Subnets | Manually configured | Configured through inputs |
| Internet Gateway | Manually configured | Module-managed |
| Route tables | Manually configured | Module-managed |
| Route associations | Manually configured | Module-managed |
| Reusability | Requires additional abstraction | Reusable module |
| Maintenance | More manual effort | Centralized module implementation |

### Resource Count Comparison

The exact number of resources depends on the configuration and module version.

To compare the actual resource counts:

**Before using the registry module:**

```bash
terraform state list
```

Record the resources created by your Day 62 handwritten VPC.

**After using the registry module:**

```bash
terraform state list
```

Identify resources belonging to:

```text
module.vpc.
```

Count the resources:

```bash
terraform state list | grep '^module.vpc\.' | wc -l
```

Record your actual results:

| Implementation | Resource Count |
|---|---:|
| Day 62 handwritten VPC | Add actual count |
| Day 65 registry VPC | Add actual count |

**Observation:**

The registry module does not necessarily create fewer AWS resources. Its main advantage is that it abstracts resource definitions, provides reusable configuration, and reduces the amount of infrastructure code we need to maintain.

---

## 13. Five Terraform Module Best Practices

### 1. Pin Module Versions

Always specify version constraints for registry modules.

This helps prevent unexpected infrastructure changes when newer module versions are released.

### 2. Keep Modules Focused

Each module should have one clear responsibility.

Examples:

- EC2 module handles compute resources.
- Security Group module handles firewall rules.
- VPC module handles networking.

Avoid creating unnecessarily large modules that manage unrelated infrastructure.

### 3. Use Variables Instead of Hardcoding

Make modules configurable through input variables.

Avoid hardcoding:

- AMI IDs.
- Instance types.
- Subnet IDs.
- Resource names.
- Environment-specific values.

This makes modules reusable across different environments.

### 4. Define Outputs

Every useful custom module should expose the information that other modules or root configurations need.

Examples:

- EC2 instance ID.
- EC2 public IP.
- Security Group ID.
- VPC ID.

Outputs allow modules to communicate without tightly coupling their internal resource definitions.

### 5. Add README Files

Every reusable custom module should contain a README explaining:

- What the module does.
- Required input variables.
- Optional input variables.
- Available outputs.
- Usage examples.
- Dependencies and requirements.

This makes modules easier for other developers to understand and reuse.

---

## 14. Important Terraform Commands – Quick Reference

| Command | Purpose |
|---|---|
| `terraform init` | Initialize Terraform and download dependencies |
| `terraform get` | Download and update modules |
| `terraform fmt -recursive` | Format Terraform files |
| `terraform validate` | Validate configuration |
| `terraform plan` | Preview infrastructure changes |
| `terraform apply` | Create or update infrastructure |
| `terraform output` | Display outputs |
| `terraform state list` | List resources in state |
| `terraform init -upgrade` | Upgrade dependencies within constraints |
| `terraform destroy` | Destroy managed infrastructure |

---

## 15. Common Learning Points

### What is a Terraform module?

A reusable collection of Terraform configuration files that manages a specific part of infrastructure.

### Why do we need modules?

To avoid duplicate code, improve maintainability, and reuse infrastructure configurations.

### Can we call the same module multiple times?

Yes. We can call a module multiple times with different input values.

In this exercise, the EC2 module is called twice:

```hcl
module "web_server" {
  source = "./modules/ec2-instance"

  instance_name = "terraweek-web"
  # Other inputs
}

module "api_server" {
  source = "./modules/ec2-instance"

  instance_name = "terraweek-api"
  # Other inputs
}
```

### What is a dynamic block?

A dynamic block generates repeated nested blocks based on a list, set, or map.

### Where are registry modules downloaded?

Terraform downloads registry modules into:

```text
.terraform/modules/
```

### How do we access module outputs?

Using:

```hcl
module.<module_name>.<output_name>
```

Example:

```hcl
module.web_server.public_ip
```

---

## 16. Key Takeaways

Today's exercise helped me understand how Terraform modules make infrastructure code reusable and maintainable.

What I learned:

- The difference between root and child modules.
- How to create custom Terraform modules.
- How to pass inputs using variables.
- How to expose resource details using outputs.
- How to use dynamic blocks for repeated configuration.
- How to reuse one EC2 module for multiple instances.
- How to integrate custom modules with registry modules.
- How Terraform manages module dependencies and state.

**The biggest takeaway:** Instead of writing infrastructure repeatedly, we can create reusable modules and build complex infrastructure by combining smaller components.

This is an important step towards writing production-quality Infrastructure as Code.

---

## 17. GitHub Submission

Repository structure:

```text
2026/
└── day-65/
    ├── day-65-modules.md
    ├── terraform-modules/
    │   ├── main.tf
    │   ├── variables.tf
    │   ├── outputs.tf
    │   ├── providers.tf
    │   └── modules/
    │       ├── ec2-instance/
    │       │   ├── main.tf
    │       │   ├── variables.tf
    │       │   └── outputs.tf
    │       └── security-group/
    │           ├── main.tf
    │           ├── variables.tf
    │           └── outputs.tf
    └── screenshots/
        └── ec2-instances.png
```

### Git Commands

```bash
git add 2026/day-65/

git commit -m "Day 65: Learn Terraform Modules"

git push origin main
```

Make sure to exclude the following from Git:

```text
.terraform/
*.tfstate
*.tfstate.*
*.tfplan
```

Keep Terraform state files and sensitive credentials out of your public repository.

A `.gitignore` example:

```gitignore
.terraform/
*.tfstate
*.tfstate.*
*.tfplan
crash.log
*.tfvars
!example.tfvars
```

---

## 18. Learn in Public – LinkedIn Post

Built my first custom Terraform modules today as part of the #90DaysOfDevOps challenge!

Created reusable EC2 and Security Group modules, called the same EC2 module multiple times with different configurations, and integrated everything with an official Terraform Registry VPC module.

Key learnings:
- Root vs Child Modules
- Custom Terraform Modules
- Dynamic Blocks
- Module Inputs and Outputs
- Terraform Registry Modules
- Module Versioning and Best Practices

One important lesson: **Write once, reuse everywhere.**

Terraform modules make infrastructure code cleaner, scalable, and easier to maintain.

#90DaysOfDevOps #TerraWeek #DevOpsKaJosh #TrainWithShubham #Terraform #AWS #InfrastructureAsCode #DevOps

---

**Day 65 Completed – Terraform Modules: Build Reusable Infrastructure**

Happy Learning! 🚀