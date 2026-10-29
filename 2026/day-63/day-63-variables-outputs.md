# Day 63 — Variables, Outputs, Data Sources and Expressions

## Objective

Make Terraform configurations dynamic, reusable, and environment-aware by using input variables, variable files, outputs, data sources, locals, built-in functions, and conditional expressions.

## 1. Input Variables

Terraform variables eliminate hardcoded values and allow the same infrastructure configuration to be reused across projects and environments.

### `variables.tf`

```hcl
variable "region" {
  type    = string
  default = "us-east-1"
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
  default = "t2.micro"
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

### Five Common Terraform Variable Types

| Type     | Description                                | Example                   |
| -------- | ------------------------------------------ | ------------------------- |
| `string` | Represents text                            | `"dev"`                   |
| `number` | Represents numeric values                  | `8080`                    |
| `bool`   | Represents `true` or `false`               | `true`                    |
| `list`   | Represents an ordered collection of values | `["dev", "prod"]`         |
| `map`    | Represents key-value pairs                 | `{ Environment = "dev" }` |

Terraform also supports more specific collection types, such as `list(string)` and `map(string)`.

### Referencing Variables

Use `var.<variable_name>` to reference an input variable.

```hcl
region        = var.region
cidr_block    = var.vpc_cidr
instance_type = var.instance_type
```

**Key takeaway:** Variables make configurations reusable without modifying resource definitions for every environment.

---

## 2. Variable Files and Precedence

Terraform supports variable files to provide different configuration values for different environments.

### `terraform.tfvars` — Development

```hcl
project_name = "terraweek"
environment  = "dev"
instance_type = "t2.micro"
```

Terraform automatically loads `terraform.tfvars`.

### `prod.tfvars` — Production

```hcl
project_name  = "terraweek"
environment   = "prod"
instance_type = "t3.small"
vpc_cidr      = "10.1.0.0/16"
subnet_cidr   = "10.1.1.0/24"
```

Custom variable files must be specified explicitly.

### Running Terraform with Different Environments

**Development:**

```bash
terraform plan
```

**Production:**

```bash
terraform plan -var-file="prod.tfvars"
```

**Override the instance type through the CLI:**

```bash
terraform plan -var-file="prod.tfvars" -var="instance_type=t2.nano"
```

**Set an environment variable:**

Linux/macOS:

```bash
export TF_VAR_environment="staging"
terraform plan
```

PowerShell:

```powershell
$env:TF_VAR_environment = "staging"
terraform plan
```

### Variable Precedence

From lowest to highest priority, the general order is:

1. Variable default values.
2. `TF_VAR_` environment variables.
3. `terraform.tfvars` and `terraform.tfvars.json`.
4. Automatically loaded `*.auto.tfvars` and `*.auto.tfvars.json` files.
5. Explicit `-var-file` and `-var` CLI arguments.

When multiple variable files or CLI arguments of the same category are provided, later assignments generally take precedence.

**Important:** Environment variables do not override values explicitly supplied through `terraform.tfvars`. CLI arguments have higher precedence than automatically loaded variable files.

---

## 3. Terraform Outputs

Outputs expose useful information about resources after Terraform creates or manages them.

### `outputs.tf`

The following resource names are examples. Replace them if your actual resource names differ.

```hcl
output "vpc_id" {
  description = "The ID of the VPC"
  value       = aws_vpc.main.id
}

output "subnet_id" {
  description = "The ID of the public subnet"
  value       = aws_subnet.my-terra-subnet.id
}

output "instance_id" {
  description = "The ID of the EC2 instance"
  value       = aws_instance.terra-iac-instance.id
}

output "instance_public_ip" {
  description = "The public IP address of the EC2 instance"
  value       = aws_instance.terra-iac-instance.public_ip
}

output "instance_public_dns" {
  description = "The public DNS name of the EC2 instance"
  value       = aws_instance.terra-iac-instance.public_dns
}

output "security_group_id" {
  description = "The ID of the security group"
  value       = aws_security_group.terraweek_sg.id
}
```

### Viewing Outputs

Apply the configuration:

```bash
terraform apply
```

Display all outputs:

```bash
terraform output
```

Display a specific output:

```bash
terraform output instance_public_ip
```

Display outputs in JSON format:

```bash
terraform output -json
```

### Verification

After a successful apply, verify that:

* The VPC, subnet, instance, and security group IDs are displayed.
* The EC2 instance public IP is correct.
* The public DNS output corresponds to the EC2 instance.

**Note:** A public IP or DNS name may be empty if the instance does not have a public address or public DNS name assigned.

---

## 4. Data Sources

Data sources allow Terraform to retrieve information about existing infrastructure or available resources without creating them.

### `data.tf` — Amazon Linux 2 AMI

```hcl
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

  filter {
    name   = "block-device-mapping.volume-type"
    values = ["gp2"]
  }
}
```

This data source looks up the most recent matching Amazon Linux 2 AMI owned by Amazon.

The `gp2` filter requires an AMI whose root block device mapping uses the specified volume type.

### Fetching Availability Zones

```hcl
data "aws_availability_zones" "available" {
  state = "available"
}
```

Reference the first available Availability Zone using:

```hcl
data.aws_availability_zones.available.names[0]
```

**Important:** An EC2 instance launched into a specific subnet must use that subnet's Availability Zone. Do not independently select the first Availability Zone if it does not match the subnet's Availability Zone. When `subnet_id` is specified, AWS automatically determines the instance's Availability Zone from the subnet.

### Updating the EC2 Instance

Replace a hardcoded AMI ID with:

```hcl
ami = data.aws_ami.amazon_linux.id
```

### Resource vs. Data Source

| Resource                                | Data Source                                              |
| --------------------------------------- | -------------------------------------------------------- |
| Creates or manages infrastructure.      | Reads information about existing or available resources. |
| Can change infrastructure during apply. | Does not create or modify the resource being queried.    |
| Example: `aws_instance`                 | Example: `aws_ami`                                       |

**Key takeaway:** Data sources make configurations more dynamic by retrieving information instead of relying on hardcoded IDs.

---

## 5. Locals and Dynamic Tags

Locals allow you to define reusable expressions and values inside a Terraform configuration.

### `locals.tf`

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

For example, if:

```hcl
project_name = "terraweek"
environment  = "dev"
```

Then:

```text
local.name_prefix = terraweek-dev
```

### Applying Consistent Tags

**VPC:**

```hcl
tags = merge(local.common_tags, {
  Name = "${local.name_prefix}-vpc"
})
```

**Subnet:**

```hcl
tags = merge(local.common_tags, {
  Name = "${local.name_prefix}-subnet"
})
```

**EC2 instance:**

```hcl
tags = merge(local.common_tags, {
  Name = "${local.name_prefix}-server"
})
```

### Understanding `merge()`

The `merge()` function combines maps into a single map.

For example:

```hcl
tags = merge(local.common_tags, {
  Name = "${local.name_prefix}-server"
})
```

Produces the equivalent of:

```hcl
tags = {
  Project     = "terraweek"
  Environment = "dev"
  ManagedBy   = "Terraform"
  Name        = "terraweek-dev-server"
}
```

If the same key exists in multiple maps, the value from the later map takes precedence.

**Key takeaway:** Locals and `merge()` reduce duplication and make resource naming and tagging consistent.

---

## 6. Built-in Functions and Conditional Expressions

Terraform provides built-in functions to manipulate strings, collections, numbers, and network addresses.

Open the Terraform console:

```bash
terraform console
```

### String Functions

**1. `upper()` — Converts text to uppercase**

```hcl
upper("terraweek")
```

Result:

```text
"TERRAWEEK"
```

**2. `join()` — Joins strings using a separator**

```hcl
join("-", ["terra", "week", "2026"])
```

Result:

```text
"terra-week-2026"
```

**3. `format()` — Formats a string**

```hcl
format("arn:aws:s3:::%s", "my-bucket")
```

Result:

```text
"arn:aws:s3:::my-bucket"
```

This constructs a string resembling an S3 bucket ARN. It does not verify that the bucket exists.

### Collection Functions

**4. `length()` — Returns the number of elements**

```hcl
length(["a", "b", "c"])
```

Result:

```text
3
```

**5. `lookup()` — Retrieves a value from a map**

```hcl
lookup({ dev = "t2.micro", prod = "t3.small" }, "dev")
```

Result:

```text
"t2.micro"
```

A third argument can be provided as a default when the requested key does not exist.

### Additional Collection Function

`toset()` converts a collection into a set, removing duplicates:

```hcl
toset(["a", "b", "a"])
```

Result:

```text
toset([
  "a",
  "b",
])
```

Sets are unordered collections.

### Networking Function

`cidrsubnet()` calculates a subnet CIDR from a parent CIDR block.

```hcl
cidrsubnet("10.0.0.0/16", 8, 1)
```

Result:

```text
"10.0.1.0/24"
```

Here:

* `10.0.0.0/16` is the parent network.
* `8` adds eight network bits, producing a `/24`.
* `1` selects the second subnet, using zero-based numbering.

### Conditional Expressions

Conditional expressions select a value based on a condition.

```hcl
instance_type = var.environment == "prod" ? "t3.small" : "t2.micro"
```

This means:

* If `environment` equals `"prod"`, use `t3.small`.
* Otherwise, use `t2.micro`.

**Important:** Terraform allows only one assignment to a given argument within a resource block. Replace the existing `instance_type` assignment rather than adding a second one.

Also, if the goal is to test environment-specific sizing, remove or account for the existing `instance_type` variable value so the intended conditional expression is actually used.

### Five Useful Functions

| Function       | Purpose                                                    |
| -------------- | ---------------------------------------------------------- |
| `merge()`      | Combines maps, especially resource tags.                   |
| `lookup()`     | Retrieves a value from a map.                              |
| `length()`     | Counts elements in a collection or characters in a string. |
| `cidrsubnet()` | Calculates subnet CIDR blocks.                             |
| `format()`     | Creates formatted strings from values.                     |

---

## 7. Understanding Variable, Local, Output, and Data

| Concept    | Purpose                                                      | Example                                     |
| ---------- | ------------------------------------------------------------ | ------------------------------------------- |
| `variable` | Accepts input from the user or configuration.                | `var.region`                                |
| `local`    | Defines reusable values or expressions.                      | `local.name_prefix`                         |
| `output`   | Exposes useful information after Terraform operations.       | `aws_instance.terra-iac-instance.public_ip` |
| `data`     | Retrieves information about existing or available resources. | `data.aws_ami.amazon_linux.id`              |

### Example Workflow

1. A variable defines the AWS region.
2. A data source retrieves an AMI in that region.
3. A resource creates an EC2 instance using the AMI.
4. A local constructs the instance's Name tag.
5. An output displays the instance's public IP.

Together, these features help create modular, reusable Terraform configurations.

---

## 8. Validation and Deployment

Format the configuration:

```bash
terraform fmt
```

Validate the configuration:

```bash
terraform validate
```

Review the development plan:

```bash
terraform plan
```

Review the production plan:

```bash
terraform plan -var-file="prod.tfvars"
```

Apply the development configuration:

```bash
terraform apply
```

Apply the production configuration:

```bash
terraform apply -var-file="prod.tfvars"
```

Check the outputs:

```bash
terraform output
```

**Caution:** Changing CIDR blocks or other resource arguments may cause Terraform to replace existing infrastructure. Always review the plan before applying, particularly in production.

---

## 9. Screenshots and Verification

Capture a screenshot of the successful `terraform apply` output.

The screenshot should show the final outputs, including:

* VPC ID
* Public subnet ID
* EC2 instance ID
* EC2 public IP
* EC2 public DNS
* Security group ID

Also verify the resource tags in the AWS console.

> **Screenshot placeholder:** Add your actual terminal screenshot here after a successful apply.

---

## 10. Key Learnings

* Input variables eliminate hardcoded configuration values.
* Variable files make it easier to manage development and production environments.
* Variable precedence determines which value Terraform uses when multiple values are provided.
* Outputs expose useful resource attributes after deployment.
* Data sources retrieve information about existing or available resources.
* Locals provide reusable expressions and consistent naming.
* `merge()` combines common tags with resource-specific tags.
* Built-in functions simplify string manipulation, collection processing, and CIDR calculations.
* Conditional expressions allow environment-specific configuration.
* `terraform fmt`, `terraform validate`, and `terraform plan` help verify changes before deployment.

## Conclusion

Day 63 focused on making Terraform configurations reusable, dynamic, and environment-aware.

By combining variables, `.tfvars` files, outputs, data sources, locals, built-in functions, and conditional expressions, the same configuration can support multiple environments with fewer hardcoded values and more consistent infrastructure management.
