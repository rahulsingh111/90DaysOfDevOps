# Day 66 — Provisioning an AWS EKS Cluster with Terraform Modules

**Challenge:** 90 Days of DevOps — TerraWeek  
**Topic:** Infrastructure as Code | AWS | Terraform | Kubernetes | EKS  
**Project:** `terraweek-eks`

---

## 1. Objective

The objective of this exercise was to provision a fully functional Amazon EKS cluster using Terraform Registry modules instead of manually creating infrastructure through the AWS Console.

The infrastructure includes:

- AWS VPC with public and private subnets.
- NAT Gateway for private subnet internet access.
- Amazon EKS control plane.
- EKS managed node group.
- IAM roles and security groups.
- Kubernetes access through `kubectl`.
- NGINX deployment exposed through an AWS Load Balancer.

The final objective is to destroy all resources through Terraform to avoid unnecessary AWS charges.

## 2. Architecture

The infrastructure follows this architecture:

```text
                    Internet
                       |
                 Internet Gateway
                       |
              +--------------------+
              |     AWS VPC        |
              |    10.0.0.0/16     |
              |                    |
              |  Public Subnets    |
              |  +--------------+  |
              |  | Load Balancer|  |
              |  +--------------+  |
              |         |          |
              |  +--------------+  |
              |  | NAT Gateway  |  |
              |  +--------------+  |
              |         |          |
              |--------------------|
              |                    |
              |  Private Subnets   |
              |  +--------------+  |
              |  | EKS Worker   |  |
              |  | Nodes        |  |
              |  +--------------+  |
              |         |          |
              |  +--------------+  |
              |  | NGINX Pods   |  |
              |  +--------------+  |
              +--------------------+
                       |
                EKS Control Plane
```

**Traffic flow:**

1. External users access the public AWS Load Balancer.
2. The Load Balancer forwards traffic to the Kubernetes Service.
3. Kubernetes routes traffic to the NGINX pods.
4. Worker nodes reside in private subnets.
5. NAT Gateway provides outbound internet connectivity for private resources.

## 3. Project Structure

```text
terraform-eks/
│
├── providers.tf
├── vpc.tf
├── eks.tf
├── variables.tf
├── outputs.tf
├── terraform.tfvars
├── .terraform.lock.hcl
│
├── k8s/
│   └── nginx-deployment.yaml
│
└── day-66-eks-terraform.md
```

The `.terraform` directory is generated automatically and should not be committed to Git.

Recommended `.gitignore`:

```gitignore
.terraform/
*.tfstate
*.tfstate.*
*.tfplan
crash.log
```

Do not commit AWS credentials or sensitive Terraform variable files.

## 4. Terraform Configuration

### 4.1 providers.tf

Configure the required Terraform providers.

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }

    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.0"
    }
  }
}

provider "aws" {
  region = var.region
}

provider "kubernetes" {}
```

**Important learning:**

Provider versions and module versions are different things.

- AWS provider `~> 5.0` permits AWS provider 5.x.
- Kubernetes provider `~> 2.0` permits Kubernetes provider 2.x.
- Registry modules should also be version-pinned.

### 4.2 variables.tf

```hcl
variable "region" {
  description = "AWS deployment region"
  type        = string
}

variable "cluster_name" {
  description = "EKS cluster name"
  type        = string
  default     = "terraweek-eks"
}

variable "cluster_version" {
  description = "Kubernetes version"
  type        = string
  default     = "1.31"
}

variable "node_instance_type" {
  description = "Worker node instance type"
  type        = string
  default     = "t3.medium"
}

variable "node_desired_count" {
  description = "Desired worker node count"
  type        = number
  default     = 2
}

variable "vpc_cidr" {
  description = "VPC CIDR block"
  type        = string
  default     = "10.0.0.0/16"
}
```

**Version note:** Kubernetes 1.31 was specified in the original exercise. Before creating a new cluster, check AWS EKS version support and select a currently supported Kubernetes version. Also verify that the selected node AMI type is compatible with that version.

### 4.3 terraform.tfvars

```hcl
region             = "us-east-1"
cluster_name       = "terraweek-eks"
cluster_version    = "1.31"
node_instance_type = "t3.medium"
node_desired_count = 2
vpc_cidr           = "10.0.0.0/16"
```

Update the Kubernetes version before applying if it is no longer supported.

### 4.4 vpc.tf

Use the Terraform Registry VPC module.

```hcl
data "aws_availability_zones" "available" {
  state = "available"
}

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.21.0"

  name = "eks-vpc"
  cidr = var.vpc_cidr

  azs = slice(
    data.aws_availability_zones.available.names,
    0,
    2
  )

  public_subnets = [
    cidrsubnet(var.vpc_cidr, 8, 1),
    cidrsubnet(var.vpc_cidr, 8, 2)
  ]

  private_subnets = [
    cidrsubnet(var.vpc_cidr, 8, 3),
    cidrsubnet(var.vpc_cidr, 8, 4)
  ]

  enable_nat_gateway = true
  single_nat_gateway = true

  enable_dns_hostnames = true
  enable_dns_support   = true

  public_subnet_tags = {
    "kubernetes.io/role/elb" = 1
  }

  private_subnet_tags = {
    "kubernetes.io/role/internal-elb" = 1
  }

  tags = {
    Environment = "dev"
    Terraform   = "true"
  }
}
```

#### Why does EKS need public and private subnets?

**Public subnets:**

- Host internet-facing load balancers.
- Provide access through the Internet Gateway.
- Allow external users to access applications exposed through public services.

**Private subnets:**

- Host Kubernetes worker nodes.
- Avoid exposing worker nodes directly to the internet.
- Use NAT Gateway for outbound internet connectivity.

#### What do the Kubernetes subnet tags do?

| Tag | Purpose |
|---|---|
| `kubernetes.io/role/elb` | Identifies public subnets for internet-facing load balancers. |
| `kubernetes.io/role/internal-elb` | Identifies private subnets for internal load balancers. |

These tags help Kubernetes/AWS load-balancer integrations discover suitable subnets.

### 4.5 eks.tf

Use the official EKS Terraform Registry module.

```hcl
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "20.37.2"

  cluster_name    = var.cluster_name
  cluster_version = var.cluster_version

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  cluster_endpoint_public_access = true

  authentication_mode = "API_AND_CONFIG_MAP"

  enable_cluster_creator_admin_permissions = true

  eks_managed_node_groups = {
    terraweek_nodes = {
      ami_type       = "AL2_x86_64"
      instance_types = [var.node_instance_type]

      min_size     = 1
      max_size     = 3
      desired_size = var.node_desired_count
    }
  }

  tags = {
    Environment = "dev"
    Project     = "TerraWeek"
    ManagedBy   = "Terraform"
  }
}
```

**Note:** Verify the selected AMI type and Kubernetes version are compatible before applying.

### 4.6 outputs.tf

```hcl
output "cluster_name" {
  value = module.eks.cluster_name
}

output "cluster_endpoint" {
  value = module.eks.cluster_endpoint
}

output "cluster_region" {
  value = var.region
}
```

## 5. Terraform Initialization and Deployment

### Step 1: Initialize Terraform

```bash
terraform init
```

This downloads the required providers and modules.

### Step 2: Format configuration

```bash
terraform fmt
```

### Step 3: Validate configuration

```bash
terraform validate
```

### Step 4: Review the execution plan

```bash
terraform plan
```

Review the resources Terraform intends to create.

### Step 5: Apply infrastructure

```bash
terraform apply
```

Confirm the operation when prompted.

EKS provisioning can take approximately 10–15 minutes or longer.

**Resources created include:**

- VPC and subnets.
- Internet Gateway.
- NAT Gateway and Elastic IP.
- Route tables.
- EKS cluster.
- IAM roles and policies.
- Security groups.
- Managed node group.
- Supporting networking resources.

**Actual resource count:** [Fill in the number from your Terraform apply output.]

## 6. Connect kubectl to EKS

After successful provisioning:

```bash
aws eks update-kubeconfig \
  --name terraweek-eks \
  --region us-east-1
```

This updates the local Kubernetes configuration file.

Verify the active context:

```bash
kubectl config current-context
```

Check the cluster:

```bash
kubectl cluster-info
```

Check worker nodes:

```bash
kubectl get nodes
```

Check all namespaces:

```bash
kubectl get pods -A
```

Expected result:

- Two worker nodes.
- Both nodes in `Ready` state.
- Kubernetes system pods running.

## 7. Deploy NGINX

Create `k8s/nginx-deployment.yaml`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-terraweek
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
```

### Apply the manifest

```bash
kubectl apply -f k8s/nginx-deployment.yaml
```

### Verify deployment

```bash
kubectl get deployments
kubectl get pods
kubectl get svc
```

### Wait for the Load Balancer

```bash
kubectl get svc nginx-service -w
```

AWS provisions a load balancer and assigns an external DNS hostname.

Access the application using:

```text
http://<EXTERNAL-LOAD-BALANCER-DNS>
```

Use HTTP because the example service exposes port 80 without TLS configuration.

### Actual deployment result

During this exercise, the service returned:

```text
NAME            TYPE           CLUSTER-IP
nginx-service   LoadBalancer   172.20.129.104

EXTERNAL-IP:
a7309174b92964cf995d766b1f25cd0c-755082164.us-east-1.elb.amazonaws.com

PORT(S):
80:31346/TCP
```

The NGINX application was exposed through the AWS Load Balancer.

## 8. Troubleshooting — Real Issues Encountered

This section documents the problems encountered during the exercise and their solutions.

### Issue 1: Terraform module source error

**Error:**

```text
Error: Variables not allowed

source = terraform-aws-modules/vpc/aws
```

**Cause:**

The module source was not enclosed in double quotes.

**Solution:**

Incorrect:

```hcl
source = terraform-aws-modules/vpc/aws
```

Correct:

```hcl
source = "terraform-aws-modules/vpc/aws"
```

Terraform requires module sources to be string values.

### Issue 2: AWS provider version conflict

**Error:**

```text
Failed to query available provider packages

no available releases match the given constraints
```

**Investigation:**

Executed:

```bash
terraform providers
```

This revealed conflicting provider requirements:

| Component | AWS provider requirement |
|---|---|
| EKS module 20.37.2 | `>= 5.95.0, < 6.0.0` |
| VPC module 6.x | `>= 6.28.0` |

Both requirements could not be satisfied by one AWS provider version.

**Solution:**

Pinned the VPC module to version 5.21.0 and kept the EKS module at 20.37.2.

The root AWS provider constraint was kept compatible with AWS 5.x.

```hcl
aws = {
  source  = "hashicorp/aws"
  version = "~> 5.0"
}
```

Then executed:

```bash
terraform init -upgrade
```

**Learning:**

Always inspect provider requirements when combining Terraform modules. A module upgrade can introduce a provider-version conflict.

### Issue 3: kubectl authentication failure

**Error:**

```text
The server has asked the client to provide credentials

error: You must be logged in to the server
```

**Initial investigation:**

```bash
aws sts get-caller-identity
```

AWS authentication was successful.

The active IAM identity was:

```text
arn:aws:iam::384043731926:user/terra-user
```

Checked EKS authentication mode:

```bash
aws eks describe-cluster \
  --name terraweek-eks \
  --region us-east-1 \
  --query "cluster.accessConfig"
```

Result:

```json
{
  "authenticationMode": "API_AND_CONFIG_MAP"
}
```

**Root cause:**

The IAM identity was valid in AWS but did not have the required Kubernetes cluster access.

Updating kubeconfig alone does not grant cluster permissions.

**Solution:**

Created an EKS access entry:

```bash
aws eks create-access-entry \
  --cluster-name terraweek-eks \
  --region us-east-1 \
  --principal-arn arn:aws:iam::384043731926:user/terra-user \
  --type STANDARD
```

Associated the administrator policy:

```bash
aws eks associate-access-policy \
  --cluster-name terraweek-eks \
  --region us-east-1 \
  --principal-arn arn:aws:iam::384043731926:user/terra-user \
  --policy-arn arn:aws:eks::aws:cluster-access-policy/AmazonEKSClusterAdminPolicy \
  --access-scope type=cluster
```

Then refreshed kubeconfig:

```bash
aws eks update-kubeconfig \
  --name terraweek-eks \
  --region us-east-1
```

Finally:

```bash
kubectl get nodes
```

Authentication worked.

**Learning:**

There are three separate concepts:

1. IAM authentication — Who are you?
2. EKS access entries — Which IAM identities can access the cluster?
3. Kubernetes authorization — What actions are those identities allowed to perform?

For production environments, use least-privilege access instead of granting cluster administrator permissions to every user.

## 9. Destroy Infrastructure

This is an important part of Infrastructure as Code.

Creating infrastructure is only half the job. We must also be able to remove it cleanly.

### Step 1: Delete Kubernetes resources

```bash
kubectl delete -f k8s/nginx-deployment.yaml
```

This removes the Deployment and LoadBalancer Service.

### Step 2: Wait for Load Balancer deletion

Check:

```bash
kubectl get svc
```

Wait until the LoadBalancer has been removed from AWS.

### Step 3: Destroy Terraform infrastructure

```bash
terraform destroy
```

Review the resources scheduled for deletion and confirm.

### Step 4: Verify cleanup

Check the AWS Console:

- EKS: No remaining cluster.
- EC2: No remaining EKS worker instances.
- VPC: Terraform-created VPC deleted.
- NAT Gateways: Deleted.
- Elastic IPs: Released.
- Load Balancers: Deleted.

**Important:** Verify all resources are removed before considering the exercise complete. Some independently created or externally managed resources may not be tracked by Terraform.

**Destroy verification:** [Fill in your actual result.]

## 10. Terraform vs kind vs Minikube

| Feature | kind | Minikube | AWS EKS + Terraform |
|---|---|---|---|
| Environment | Local containers | Local machine | AWS cloud |
| Control plane | Local | Local | AWS managed |
| Worker nodes | Containers | Local VMs/containers | EC2 instances |
| Networking | Local | Local | VPC and subnets |
| Load Balancer | Usually simulated | Add-ons/tunnels | AWS Load Balancer |
| IAM integration | No native AWS IAM | No native AWS IAM | AWS IAM and EKS access |
| Production relevance | Testing | Development | Production-capable |
| Cost | Mostly free | Mostly free | AWS charges apply |

### Key differences

**kind / Minikube:**

- Excellent for learning Kubernetes fundamentals.
- Quick to create and destroy.
- No need to provision cloud infrastructure.
- Limited representation of production cloud networking and IAM.

**EKS with Terraform:**

- Demonstrates real cloud infrastructure provisioning.
- Introduces IAM, VPC, networking and security groups.
- Provides managed Kubernetes control plane.
- Supports managed node groups and cloud load balancers.
- Requires cost management and infrastructure lifecycle planning.

The biggest takeaway is that Terraform allows infrastructure to be defined, reviewed, version-controlled, reproduced and destroyed consistently.

## 11. Cost and Security Considerations

EKS infrastructure can generate charges even when application workloads are idle.

Important cost components:

- EKS cluster control plane.
- EC2 worker nodes.
- NAT Gateway hourly and data processing charges.
- Elastic IP charges where applicable.
- Load Balancer hourly and usage charges.
- Data transfer.

For this exercise, a single NAT Gateway was used to reduce cost.

However, a single NAT Gateway is not highly available across Availability Zones. Production environments may require one NAT Gateway per AZ.

Security considerations:

- Keep worker nodes in private subnets.
- Restrict EKS API endpoint access where practical.
- Use least-privilege IAM policies.
- Avoid exposing sensitive applications publicly.
- Use HTTPS for production applications.
- Never commit AWS credentials or Terraform state files to GitHub.

## 12. Key Learnings

1. Terraform Registry modules simplify complex infrastructure provisioning.
2. Module versions and provider versions must be compatible.
3. `terraform providers` is useful for dependency troubleshooting.
4. EKS requires carefully designed VPC networking.
5. Public and private subnet tags help AWS load-balancer integrations discover subnets.
6. EKS IAM authentication and Kubernetes authorization are separate concerns.
7. Updating kubeconfig does not automatically grant cluster access.
8. Kubernetes LoadBalancer Services can provision AWS load balancers.
9. Terraform makes infrastructure repeatable and easier to manage.
10. Destroying infrastructure is equally important as provisioning it.

## 13. Evidence and Screenshots

Add the following screenshots before submitting the challenge.

| Screenshot | Evidence |
|---|---|
| `01-terraform-apply.png` | Successful Terraform apply |
| `02-terraform-nodes.png` | Two Kubernetes nodes in Ready state |
| `03-kube-system.png` | Kubernetes system pods |
| `04-nginx-pods.png` | Three NGINX replicas running |
| `05-nginx-service.png` | LoadBalancer service and external hostname |
| `06-nginx-browser.png` | NGINX welcome page in browser |
| `07-terraform-destroy.png` | Successful Terraform destroy |
| `08-aws-cleanup.png` | AWS console showing cleanup |

Store screenshots in:

```text
2026/day-66/screenshots/
```

**Actual Terraform resource count:** [Fill in]

**Terraform apply result:** [Attach screenshot]

**Node verification:** [Attach screenshot]

**NGINX verification:** [Attach screenshot]

**Terraform destroy result:** [Attach screenshot]

**AWS cleanup verification:** [Attach screenshot]

## 14. Final Reflection

This exercise demonstrated how infrastructure teams use Terraform to automate cloud infrastructure instead of relying on manual console operations.

Compared with kind and Minikube, AWS EKS introduces real-world infrastructure concepts such as VPC design, subnet routing, IAM authentication, managed node groups, NAT Gateways and cloud load balancing.

The troubleshooting process was equally valuable. I encountered Terraform module/provider version conflicts and EKS authentication issues, investigated them using Terraform and AWS CLI commands, and resolved them without recreating the cluster.

The biggest takeaway:

**Infrastructure as Code is not just about creating infrastructure. It is about making infrastructure predictable, reproducible, maintainable and disposable.**

---

## 15. GitHub Submission

Expected repository structure:

```text
2026/
└── day-66/
    ├── day-66-eks-terraform.md
    ├── providers.tf
    ├── vpc.tf
    ├── eks.tf
    ├── variables.tf
    ├── outputs.tf
    ├── terraform.tfvars
    ├── k8s/
    │   └── nginx-deployment.yaml
    └── screenshots/
        ├── 01-terraform-apply.png
        ├── 02-terraform-nodes.png
        ├── 03-kube-system.png
        ├── 04-nginx-pods.png
        ├── 05-nginx-service.png
        ├── 06-nginx-browser.png
        ├── 07-terraform-destroy.png
        └── 08-aws-cleanup.png
```

### Git commands

```bash
git add 2026/day-66/

git commit -m "Day 66: Provision AWS EKS with Terraform modules"

git push origin main
```

### Submission checklist

- [ ] Terraform configuration committed.
- [ ] VPC module and EKS module versions pinned.
- [ ] Terraform apply completed successfully.
- [ ] EKS cluster created.
- [ ] Two worker nodes in Ready state.
- [ ] kubectl connected successfully.
- [ ] NGINX deployment running with three replicas.
- [ ] Public LoadBalancer URL accessible.
- [ ] Screenshots added.
- [ ] Terraform destroy completed.
- [ ] AWS resources verified as deleted.
- [ ] Documentation committed and pushed.

**End of Day 66 — 90 Days of DevOps**
