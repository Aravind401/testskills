# Day 10 — Terraform Fundamentals & Infrastructure as Code

> **30-Day DevOps → MLOps Study Plan**  
> **Focus:** Infrastructure as Code, Terraform architecture, providers, resources, data sources, variables, outputs, locals, state, remote state, state locking, modules, lifecycle, dependency graph, `count` vs `for_each`, import, drift, secrets, Azure/AWS Terraform, CI/CD, practical labs, troubleshooting, and senior-level interview preparation  
> **Recommended time:** 4–5 hours  
> **Target:** ~6 years DevOps experience

---

# 1. Day 10 Objectives

By the end of Day 10, you should be able to:

- Explain Infrastructure as Code.
- Explain why Terraform is used.
- Understand Terraform's architecture.
- Understand providers.
- Understand resources.
- Understand data sources.
- Understand variables.
- Understand outputs.
- Understand locals.
- Understand Terraform state.
- Understand remote state.
- Understand state locking.
- Understand Terraform modules.
- Understand dependency graphs.
- Understand `count` vs `for_each`.
- Understand `lifecycle`.
- Understand `terraform import`.
- Understand drift.
- Understand Terraform secrets management.
- Understand Azure and AWS Terraform concepts.
- Understand Terraform in CI/CD.
- Build a practical Terraform project.
- Troubleshoot common Terraform failures.
- Answer senior-level Terraform interview questions.

---

# 2. Recommended Schedule

| Time | Topic |
|---|---|
| 00:00–00:30 | Infrastructure as Code + Terraform architecture |
| 00:30–01:10 | Providers, Resources, Data Sources |
| 01:10–01:50 | Variables, Locals, Outputs |
| 01:50–02:30 | Terraform State + Remote State |
| 02:30–03:10 | Modules + dependency graph |
| 03:10–03:45 | `count`, `for_each`, lifecycle, import |
| 03:45–04:30 | Hands-on Terraform lab |
| 04:30–05:00 | Troubleshooting + interview Q&A |

---

# 3. What Is Infrastructure as Code?

Infrastructure as Code (IaC) means defining infrastructure using code/configuration instead of manually creating everything through a GUI.

Without IaC:

```text
Engineer
   ↓
Cloud Portal
   ↓
Click resources
   ↓
Configure manually
```

With IaC:

```text
Terraform Code
      ↓
Terraform
      ↓
Cloud Provider
      ↓
Infrastructure
```

Benefits:

- Repeatability
- Version control
- Review
- Automation
- Consistency
- Faster provisioning
- Easier disaster recovery
- Reduced configuration drift

---

# 4. Why Terraform?

Terraform is a declarative Infrastructure as Code tool.

You describe:

```text
What infrastructure should exist?
```

Terraform determines how to create/update it through providers.

Example:

```text
I need:
- Resource Group
- Virtual Network
- Subnet
- VM
```

Terraform builds a dependency graph and executes the required operations.

---

# 5. Declarative vs Imperative

### Imperative

You tell the system:

```text
1. Create VNet
2. Create subnet
3. Create VM
4. Attach NIC
5. Start VM
```

### Declarative

You describe the desired state:

```text
VNet exists
Subnet exists
VM exists
```

Terraform determines the required operations.

Easy memory:

```text
Imperative = How
Declarative = What
```

---

# 6. Terraform Architecture

High-level:

```text
Terraform Configuration
        ↓
Terraform Core
        ↓
Provider Plugin
        ↓
Cloud/API
        ↓
Infrastructure
```

Terraform Core handles:

- Configuration evaluation
- Dependency graph
- State
- Planning
- Change calculation

Provider plugins handle communication with external APIs.

Examples:

```text
AzureRM
AWS
Kubernetes
GitHub
```

---

# 7. Terraform Files

Typical project:

```text
terraform-project/
│
├── main.tf
├── variables.tf
├── outputs.tf
├── locals.tf
├── providers.tf
├── versions.tf
├── terraform.tfvars
└── .gitignore
```

You do not have to split files this way. Terraform loads all `.tf` files in the working directory together.

---

# 8. Terraform Configuration Example

```hcl
terraform {
  required_version = ">= 1.6.0"
}

provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "example" {
  name     = "rg-devops-day10"
  location = "East US"
}
```

Concept:

```text
terraform block
     ↓
Provider
     ↓
Resource
```

---

# 9. Terraform Provider

A provider is a plugin that allows Terraform to interact with an API.

Examples:

```hcl
provider "azurerm" {
  features {}
}
```

AWS:

```hcl
provider "aws" {
  region = "ap-south-1"
}
```

Kubernetes:

```hcl
provider "kubernetes" {
}
```

---

# 10. Provider Version Constraints

Production Terraform should control provider versions.

Example:

```hcl
terraform {
  required_providers {

    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.0"
    }

  }
}
```

Why?

Without version control:

```text
Today
 ↓
Provider version A

Later
 ↓
Provider version B
 ↓
Unexpected behavior
```

Use controlled upgrade processes.

---

# 11. Resource

A resource represents infrastructure Terraform manages.

Example:

```hcl
resource "azurerm_resource_group" "app" {

  name     = "rg-app"
  location = "East US"

}
```

Terraform address:

```text
azurerm_resource_group.app
```

---

# 12. Resource Address

For:

```hcl
resource "aws_s3_bucket" "logs" {
}
```

Address:

```text
aws_s3_bucket.logs
```

With a module:

```text
module.network.aws_vpc.main
```

Resource addresses are extremely important for:

- State
- Import
- Targeted inspection
- Troubleshooting

---

# 13. Data Sources

A data source reads existing information.

Example:

```hcl
data "azurerm_resource_group" "existing" {

  name = "rg-existing"

}
```

Then:

```hcl
location = data.azurerm_resource_group.existing.location
```

Difference:

```text
resource
→ Manage/create infrastructure

data
→ Read existing information
```

---

# 14. Variables

Variables make Terraform reusable.

`variables.tf`:

```hcl
variable "location" {

  description = "Azure region"

  type        = string

  default     = "East US"

}
```

Use:

```hcl
location = var.location
```

---

# 15. Variable Types

Common Terraform types:

```text
string
number
bool
list
set
map
object
tuple
```

Example:

```hcl
variable "environment" {
  type = string
}
```

Map:

```hcl
variable "tags" {

  type = map(string)

}
```

Object:

```hcl
variable "vm_config" {

  type = object({
    size  = string
    admin = string
  })

}
```

---

# 16. Input Variables

Possible sources include:

```text
Default value
 ↓
terraform.tfvars
 ↓
*.auto.tfvars
 ↓
-var
 ↓
Environment variables
```

For CI/CD, pass environment-specific values through secure pipeline mechanisms rather than committing sensitive values.

---

# 17. `terraform.tfvars`

Example:

```hcl
location    = "East US"
environment = "dev"

tags = {
  owner = "platform"
  env   = "dev"
}
```

Do not commit sensitive `.tfvars` files containing secrets.

Add appropriate files to `.gitignore`.

---

# 18. Outputs

Outputs expose useful values after Terraform runs.

Example:

```hcl
output "resource_group_name" {

  value = azurerm_resource_group.app.name

}
```

Run:

```bash
terraform output
```

Output:

```text
resource_group_name = "rg-app"
```

Outputs are also useful when passing information between modules.

---

# 19. Locals

Locals reduce repetition.

Example:

```hcl
locals {

  common_tags = {
    environment = "dev"
    managed_by  = "terraform"
  }

}
```

Use:

```hcl
tags = local.common_tags
```

Locals are useful for:

- Naming
- Tags
- Derived values
- Reusable expressions

---

# 20. Resource Naming

Avoid hard-coded names everywhere.

Bad:

```hcl
name = "my-vm"
```

Better:

```hcl
name = "${var.project}-${var.environment}-vm"
```

Example:

```text
payments-dev-vm
payments-prod-vm
```

Naming standards become especially important when managing many environments.

---

# 21. Terraform State

Terraform needs to know:

```text
What exists?
What does Terraform manage?
What changed?
```

Terraform stores this information in the state.

Common file:

```text
terraform.tfstate
```

Concept:

```text
Configuration
     ↓
Desired State

State
     ↓
Known Managed Infrastructure

Cloud
     ↓
Actual Infrastructure
```

Terraform compares these to determine changes.

---

# 22. State Is Critical

Do not treat:

```text
terraform.tfstate
```

as a normal temporary file.

It may contain:

- Resource metadata
- IDs
- Attributes
- Sensitive values depending on configuration/provider behavior

Protect it.

Never casually commit state files to Git.

---

# 23. Terraform Plan

Run:

```bash
terraform plan
```

Terraform evaluates:

```text
Configuration
+
State
+
Provider APIs
      ↓
Proposed Changes
```

Typical output:

```text
+ create
~ update
- destroy
```

---

# 24. Terraform Apply

```bash
terraform apply
```

Terraform:

```text
Configuration
      ↓
Plan
      ↓
Approval
      ↓
Provider API
      ↓
Infrastructure
      ↓
State Update
```

Use:

```bash
terraform apply
```

to apply changes interactively.

In CI/CD, use an approved plan workflow rather than blindly applying arbitrary changes.

---

# 25. Terraform Init

Run:

```bash
terraform init
```

It initializes:

- Provider plugins
- Module dependencies
- Backend configuration

Typical first command in a new Terraform project:

```bash
terraform init
```

---

# 26. Terraform Validate

```bash
terraform validate
```

Checks whether the configuration is syntactically and structurally valid.

Useful in CI:

```text
terraform fmt
terraform validate
terraform plan
```

---

# 27. Terraform Format

```bash
terraform fmt -recursive
```

This standardizes formatting.

Use it before committing code.

---

# 28. Terraform Destroy

```bash
terraform destroy
```

This attempts to remove resources managed by the configuration/state.

Be extremely careful in production.

A production pipeline should have strong controls around destructive operations.

---

# 29. Remote State

Local state:

```text
Developer Laptop
      ↓
terraform.tfstate
```

Problem:

```text
Developer A
Developer B
CI/CD
```

Everyone needs consistent state.

Remote state:

```text
Terraform
   ↓
Remote Backend
   ↓
Shared State
```

---

# 30. Remote Backend

A backend determines where Terraform stores state.

Common enterprise choices:

### Azure

```text
Azure Storage Account
 ↓
Blob Container
 ↓
Terraform State
```

### AWS

A common design uses:

```text
Amazon S3
 ↓
Terraform State
```

with an appropriate locking/concurrency mechanism supported by the current Terraform/backend architecture.

---

# 31. State Locking

Imagine:

```text
Engineer A
   ↓
terraform apply
   ↓
State locked

Engineer B
   ↓
terraform apply
   ↓
Blocked
```

Without proper concurrency control:

```text
State corruption
+
Conflicting changes
```

Use a backend and locking/concurrency mechanism appropriate for your Terraform version and cloud platform.

---

# 32. State Security

Remote state should have:

- Encryption
- Access control
- Versioning where appropriate
- Audit logging
- Backup/recovery
- Restricted network access where appropriate

Example architecture:

```text
CI/CD
  ↓
Identity
  ↓
Terraform Backend
  ↓
Encrypted State
```

---

# 33. Terraform Modules

A module is a reusable collection of Terraform configuration.

Example:

```text
modules/
  network/
    main.tf
    variables.tf
    outputs.tf
```

Root module:

```text
environment/
  main.tf
  variables.tf
  outputs.tf
```

Use:

```hcl
module "network" {

  source = "./modules/network"

}
```

---

# 34. Why Use Modules?

Without modules:

```text
Project A → 500 lines
Project B → 500 lines
Project C → 500 lines
```

With modules:

```text
Network Module
      ↓
Dev
Prod
QA
```

Benefits:

- Reuse
- Standardization
- Governance
- Easier maintenance
- Consistent architecture

---

# 35. Good Module Design

A good module should expose:

```text
Inputs
   ↓
Module
   ↓
Outputs
```

Example:

```text
network module

Inputs:
- address_space
- environment
- tags

Outputs:
- vnet_id
- subnet_ids
```

Avoid modules with hundreds of unrelated variables.

---

# 36. Module Versioning

For production, version reusable modules.

Example:

```hcl
module "network" {

  source  = "..."
  version = "1.2.0"

}
```

For internal Git modules, use controlled Git references/tags rather than floating branches where appropriate.

---

# 37. Dependency Graph

Terraform builds a dependency graph.

Example:

```text
Resource Group
      ↓
Virtual Network
      ↓
Subnet
      ↓
NIC
      ↓
VM
```

Terraform can determine creation order automatically from references.

---

# 38. Implicit Dependency

Example:

```hcl
resource "azurerm_subnet" "app" {

  virtual_network_name = azurerm_virtual_network.main.name

}
```

Terraform sees:

```text
Subnet
 ↓
VNet
```

This is an implicit dependency.

Prefer references over unnecessary manual dependencies.

---

# 39. Explicit Dependency

Use:

```hcl
depends_on = [
  azurerm_resource_group.app
]
```

when Terraform cannot infer the required dependency from expressions.

Do not use `depends_on` everywhere.

Bad:

```text
depends_on everywhere
```

Better:

```text
Only where dependency cannot be expressed naturally
```

---

# 40. `count`

Example:

```hcl
resource "aws_instance" "app" {

  count = 3

}
```

Creates:

```text
app[0]
app[1]
app[2]
```

Useful for simple repetition.

---

# 41. `for_each`

Example:

```hcl
variable "instances" {

  type = set(string)

  default = [
    "web",
    "api",
    "worker"
  ]

}

resource "aws_instance" "app" {

  for_each = var.instances

}
```

Addresses:

```text
app["web"]
app["api"]
app["worker"]
```

---

# 42. `count` vs `for_each`

| count | for_each |
|---|---|
| Numeric indexes | Keys |
| Simple repetition | Named instances |
| Index changes can cause address changes | Stable keys can be easier to manage |
| Good for identical resources | Good for distinct logical resources |

Rule of thumb:

```text
Simple replicas → count
Named/different instances → for_each
```

---

# 43. Terraform Lifecycle

Terraform provides lifecycle meta-arguments.

Example:

```hcl
lifecycle {

  prevent_destroy = true

}
```

Useful for critical resources.

Other lifecycle features include:

```text
create_before_destroy
prevent_destroy
ignore_changes
replace_triggered_by
```

Use them carefully because they can change normal Terraform behavior.

---

# 44. `prevent_destroy`

Example:

```hcl
lifecycle {

  prevent_destroy = true

}
```

Useful for:

```text
Production database
Critical storage
Important networking resources
```

But understand:

> It is a Terraform safety mechanism, not a complete disaster-recovery solution.

---

# 45. `ignore_changes`

Example:

```hcl
lifecycle {

  ignore_changes = [
    tags
  ]

}
```

Use only when another trusted system is intentionally managing that attribute.

Danger:

```text
Ignore everything
```

can hide real drift.

---

# 46. Terraform Import

Suppose infrastructure already exists:

```text
Cloud Portal
 ↓
Existing VNet
```

but Terraform does not manage it.

Import associates an existing resource with Terraform state.

Modern Terraform supports import blocks as well as CLI workflows.

Concept:

```text
Existing Infrastructure
       ↓
Terraform Configuration
       ↓
Import
       ↓
Terraform State
```

---

# 47. Import Example

The exact import ID depends on the provider/resource.

Conceptually:

```bash
terraform import <resource-address> <provider-resource-id>
```

Example shape:

```bash
terraform import azurerm_resource_group.app /subscriptions/.../resourceGroups/rg-app
```

After importing, write matching Terraform configuration and run:

```bash
terraform plan
```

Your goal is to reach a predictable plan, ideally with no unexpected changes.

---

# 48. Import Is Not Full Configuration Generation

Important:

```text
terraform import
```

primarily associates an existing remote object with Terraform state.

It does not magically create a perfect production Terraform configuration for the resource.

After import:

```text
Import
 ↓
Inspect
 ↓
Write/adjust configuration
 ↓
terraform plan
 ↓
Reconcile
```

---

# 49. Drift

Drift occurs when infrastructure changes outside Terraform.

Example:

```text
Terraform
 ↓
VM size = Standard_B2s
```

Someone changes it manually:

```text
Portal
 ↓
VM size = Standard_D4s
```

Now:

```text
Terraform configuration
        ≠
Actual infrastructure
```

---

# 50. Detecting Drift

Run:

```bash
terraform plan
```

Terraform refreshes/reconciles its knowledge of infrastructure and compares it against configuration/state as part of planning.

Possible result:

```text
~ update
```

---

# 51. Drift Management

Production principle:

```text
Terraform
   ↓
Source of Truth
   ↓
Infrastructure
```

Avoid:

```text
Terraform
+
Portal changes
+
CLI changes
+
Scripts
```

unless those changes are intentionally integrated into the IaC workflow.

---

# 52. Terraform and Secrets

Avoid:

```hcl
password = "MyPassword123"
```

in source code.

Also remember:

> Marking an output as `sensitive` prevents normal display, but it does not automatically mean the value is absent from Terraform state.

Use appropriate secret-management patterns.

---

# 53. Better Secret Architecture

Instead of:

```text
Terraform code
 ↓
Hard-coded password
```

prefer:

```text
Secret Manager
 ↓
Secure identity
 ↓
Terraform / Workload
```

Examples:

```text
Azure Key Vault
AWS Secrets Manager
```

Where possible, design workloads so they retrieve secrets at runtime rather than placing long-lived secrets into Terraform configuration/state unnecessarily.

---

# 54. Terraform with Azure

Common provider:

```text
azurerm
```

Typical architecture:

```text
Terraform
   ↓
AzureRM Provider
   ↓
Azure Resource Manager
   ↓
Azure Resources
```

Resources may include:

```text
Resource Groups
VNets
Subnets
VMs
AKS
Storage Accounts
Key Vault
App Services
```

---

# 55. Terraform with AWS

Common provider:

```text
aws
```

Architecture:

```text
Terraform
   ↓
AWS Provider
   ↓
AWS APIs
   ↓
AWS Resources
```

Common resources:

```text
VPC
Subnets
EC2
EKS
S3
IAM
RDS
Load Balancers
```

---

# 56. Azure Authentication

Avoid storing permanent credentials in Terraform code.

Prefer identity-based authentication such as:

```text
Managed Identity
OIDC federation
Service Principal with controlled credentials
```

In CI/CD, OIDC/federated identity can avoid long-lived cloud secrets.

---

# 57. AWS Authentication

Prefer:

```text
IAM Role
OIDC
Federated identity
```

rather than:

```text
AWS access key embedded in Terraform code
```

For CI/CD:

```text
Pipeline
 ↓
OIDC
 ↓
AWS IAM Role
 ↓
Terraform
```

---

# 58. Terraform in CI/CD

A common workflow:

```text
Developer
   ↓
Git
   ↓
Pull Request
   ↓
terraform fmt
   ↓
terraform validate
   ↓
terraform plan
   ↓
Review / Approval
   ↓
Merge
   ↓
terraform apply
```

Production should separate:

```text
Plan
```

from:

```text
Apply
```

with appropriate approval and identity controls.

---

# 59. CI/CD Terraform Pipeline

Example stages:

```text
1. Checkout
2. Init
3. Format check
4. Validate
5. Security scan
6. Plan
7. Publish plan
8. Approval
9. Apply
10. Post-deployment verification
```

Security scanning may include:

- IaC static analysis
- Secret scanning
- Policy checks
- Provider/module checks

---

# 60. Terraform Plan Artifact

A safer CI/CD approach:

```bash
terraform plan -out=tfplan
```

Then:

```bash
terraform apply tfplan
```

This reduces the chance that the configuration changes between plan and apply.

Still protect the pipeline and plan artifact appropriately.

---

# 61. Practical Lab — Local Terraform

You can learn Terraform without spending cloud money.

Use:

```text
Terraform
+
local provider
```

This lab demonstrates:

- Variables
- Resources
- Outputs
- State
- Plan
- Apply
- Destroy
- Drift concepts

---

# 62. Step 1 — Create Directory

```bash
mkdir terraform-day10
cd terraform-day10
```

---

# 63. Step 2 — Create `main.tf`

```hcl
terraform {
  required_providers {

    local = {
      source  = "hashicorp/local"
      version = "~> 2.5"
    }

  }
}

provider "local" {}

resource "local_file" "devops" {

  filename = "${path.module}/devops.txt"

  content = <<-EOT
    Day 10 Terraform Lab

    Infrastructure as Code
    State
    Variables
    Outputs
  EOT
}
```

---

# 64. Step 3 — Initialize

```bash
terraform init
```

You should see provider installation.

Then:

```bash
terraform version
```

---

# 65. Step 4 — Format

```bash
terraform fmt
```

---

# 66. Step 5 — Validate

```bash
terraform validate
```

Expected:

```text
Success! The configuration is valid.
```

---

# 67. Step 6 — Plan

```bash
terraform plan
```

Expected concept:

```text
Plan: 1 to add, 0 to change, 0 to destroy.
```

---

# 68. Step 7 — Apply

```bash
terraform apply
```

Approve when prompted.

Check:

```bash
ls
```

You should see:

```text
devops.txt
```

Read:

```bash
cat devops.txt
```

---

# 69. Step 8 — Inspect State

```bash
terraform show
```

List state resources:

```bash
terraform state list
```

Expected:

```text
local_file.devops
```

---

# 70. Step 9 — Inspect Resource

```bash
terraform state show local_file.devops
```

This helps you understand how Terraform tracks managed objects.

---

# 71. Step 10 — Add Variables

Create `variables.tf`:

```hcl
variable "environment" {

  description = "Environment name"

  type    = string
  default = "dev"

}
```

Change `main.tf`:

```hcl
content = <<-EOT
  Terraform Day 10
  Environment: ${var.environment}
EOT
```

Run:

```bash
terraform plan
```

---

# 72. Step 11 — Pass Variable

```bash
terraform apply   -var="environment=production"
```

Or create:

`terraform.tfvars`

```hcl
environment = "dev"
```

Then:

```bash
terraform apply
```

---

# 73. Step 12 — Add Output

Create `outputs.tf`:

```hcl
output "file_path" {

  value = local_file.devops.filename

}
```

Run:

```bash
terraform apply
```

Then:

```bash
terraform output
```

---

# 74. Step 13 — Add Locals

Create:

```hcl
locals {

  project_name = "devops"

  file_name = "${local.project_name}-${var.environment}.txt"

}
```

Use:

```hcl
filename = "${path.module}/${local.file_name}"
```

Run:

```bash
terraform fmt
terraform validate
terraform plan
terraform apply
```

---

# 75. Step 14 — Observe State Changes

Run:

```bash
terraform state list
```

Then:

```bash
terraform show
```

Understand:

```text
Configuration
 ↓
State
 ↓
Resource
```

---

# 76. Step 15 — Create Multiple Resources with `count`

Example:

```hcl
resource "local_file" "server" {

  count = 3

  filename = "${path.module}/server-${count.index}.txt"

  content = "Server ${count.index}"

}
```

Run:

```bash
terraform plan
terraform apply
```

State:

```bash
terraform state list
```

You should see:

```text
local_file.server[0]
local_file.server[1]
local_file.server[2]
```

---

# 77. Step 16 — Try `for_each`

Replace the resource with:

```hcl
variable "servers" {

  type = set(string)

  default = [
    "web",
    "api",
    "worker"
  ]

}

resource "local_file" "server" {

  for_each = var.servers

  filename = "${path.module}/${each.key}.txt"

  content = "Server: ${each.key}"

}
```

Run:

```bash
terraform plan
terraform apply
```

Inspect:

```bash
terraform state list
```

You should see key-based addresses such as:

```text
local_file.server["api"]
local_file.server["web"]
local_file.server["worker"]
```

---

# 78. Step 17 — Test Drift Concept

Manually edit one generated file:

```bash
echo "manual change" >> web.txt
```

Run:

```bash
terraform plan
```

Observe whether the provider detects the external modification for this resource.

Important lesson:

> Drift detection depends on what the provider/resource can observe. Terraform cannot detect every possible external change for every resource.

---

# 79. Step 18 — Destroy

```bash
terraform destroy
```

Verify:

```bash
ls
```

The managed files should be removed.

---

# 80. Optional Azure Lab

If you have an Azure subscription, create a minimal Resource Group.

`main.tf`:

```hcl
terraform {
  required_providers {

    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.0"
    }

  }
}

provider "azurerm" {

  features {}

}

variable "location" {

  type    = string
  default = "East US"

}

resource "azurerm_resource_group" "day10" {

  name     = "rg-terraform-day10"
  location = var.location

}
```

Run:

```bash
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
```

Then verify:

```bash
terraform output
```

No output exists yet unless you define one.

Clean up:

```bash
terraform destroy
```

---

# 81. Optional AWS Lab

With appropriate AWS credentials configured:

```hcl
terraform {
  required_providers {

    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }

  }
}

provider "aws" {

  region = "ap-south-1"

}
```

Start with a low-risk resource and understand the costs before creating paid infrastructure.

For production learning, focus on:

```text
VPC
Subnet
IAM
S3
EKS
```

but always verify current provider/resource behavior before applying examples.

---

# 82. Terraform Troubleshooting

## Error: Provider Not Found

Run:

```bash
terraform init
```

Check:

```bash
terraform providers
```

---

# 83. Error: Authentication Failed

Check:

```text
Cloud credentials
 ↓
Identity
 ↓
Permissions
 ↓
Provider configuration
```

Do not immediately grant administrator privileges.

---

# 84. Error: Resource Already Exists

Example:

```text
Resource already exists
```

Possible situation:

```text
Cloud resource exists
+
Terraform state does not know it
```

Options:

```text
Import
```

or intentionally change/remove the external resource.

Use import when Terraform should become the owner.

---

# 85. Error: State Lock

If state is locked:

```text
Another Terraform operation may be running
```

First verify whether another operation is genuinely active.

Do not blindly force-unlock a state.

A forced unlock can be dangerous if another Terraform process is still operating.

---

# 86. Error: Unexpected Destroy

Stop.

Run:

```bash
terraform plan
```

Investigate:

```text
Resource address changed?
count/for_each changed?
Configuration changed?
Provider behavior changed?
Lifecycle rules?
State mismatch?
```

Never blindly apply an unexpected destructive plan.

---

# 87. Terraform Debugging Commands

Format:

```bash
terraform fmt
```

Validate:

```bash
terraform validate
```

Inspect providers:

```bash
terraform providers
```

Inspect state:

```bash
terraform show
```

List resources:

```bash
terraform state list
```

Inspect resource:

```bash
terraform state show <address>
```

Graph:

```bash
terraform graph
```

Plan:

```bash
terraform plan
```

---

# 88. `terraform graph`

Run:

```bash
terraform graph
```

This shows the dependency graph in DOT format.

Concept:

```text
Resource Group
      ↓
VNet
      ↓
Subnet
      ↓
NIC
      ↓
VM
```

For large architectures, understanding the graph helps troubleshoot dependency behavior.

---

# 89. State Commands — Important Warning

Commands such as:

```bash
terraform state mv
terraform state rm
terraform state replace-provider
```

can change state without necessarily changing infrastructure.

Use them only when you understand the consequences.

State manipulation is powerful and can cause Terraform to lose track of resources if done incorrectly.

---

# 90. Terraform Workspaces

Terraform workspaces can separate state instances for the same configuration.

Concept:

```text
Configuration
   ↓
Workspace: dev
Workspace: test
Workspace: prod
```

However:

> Workspaces are not automatically the best environment-isolation strategy for every organization.

Many production teams prefer separate root modules/directories and separate state backends for stronger isolation.

Know both patterns.

---

# 91. Environment Architecture

A scalable approach:

```text
terraform/
│
├── modules/
│   ├── network/
│   ├── aks/
│   └── database/
│
└── environments/
    ├── dev/
    ├── staging/
    └── prod/
```

Concept:

```text
Reusable Modules
       ↓
Environment Root Modules
       ↓
Separate State
       ↓
Cloud Infrastructure
```

---

# 92. Production Terraform Repository

Example:

```text
infra/
│
├── modules/
│
│   ├── network/
│   ├── aks/
│   ├── database/
│   └── monitoring/
│
├── environments/
│
│   ├── dev/
│   ├── staging/
│   └── prod/
│
├── .gitignore
└── README.md
```

Avoid one giant:

```text
main.tf
```

for the entire company.

---

# 93. CI/CD Security

Terraform pipeline identity should have:

```text
Least privilege
+
OIDC/federation where possible
+
Separate environment permissions
+
Approval for production
```

Avoid:

```text
Pipeline
 ↓
Permanent admin access key
```

---

# 94. Terraform Security Checklist

- [ ] State is stored securely.
- [ ] State access is restricted.
- [ ] State locking/concurrency is configured appropriately.
- [ ] Provider versions are controlled.
- [ ] Modules are versioned.
- [ ] Secrets are not committed to Git.
- [ ] CI/CD uses short-lived/federated credentials where possible.
- [ ] Production apply requires appropriate approval.
- [ ] `terraform plan` is reviewed.
- [ ] Security/IaC scanning is part of CI.
- [ ] Destructive changes are reviewed.
- [ ] Remote state has appropriate backup/versioning.
- [ ] Terraform state is never casually edited manually.

---

# 95. Day 10 Interview Q&A

## Q1. What is Infrastructure as Code?

> Infrastructure as Code defines and manages infrastructure through version-controlled configuration instead of manual provisioning.

## Q2. Why Terraform?

> Terraform provides declarative infrastructure management, dependency handling, state tracking, planning, and provider-based integrations with many platforms.

## Q3. What is Terraform state?

> State records Terraform's knowledge of managed resources and is used to calculate changes between configuration and real infrastructure.

## Q4. Why is state important?

> Terraform uses state to map configuration resources to real infrastructure and determine what needs to change.

## Q5. Should terraform.tfstate be committed to Git?

> Normally no. State can contain sensitive information and needs controlled access and concurrency management.

## Q6. What is a provider?

> A provider is a plugin that lets Terraform interact with an external API or platform.

## Q7. Resource vs data source?

> A resource manages infrastructure; a data source reads existing information.

## Q8. What is a variable?

> An input parameter that makes Terraform configuration reusable.

## Q9. What is an output?

> A value exposed by a Terraform module or root configuration for users, automation, or other modules.

## Q10. What is a local?

> A named expression used within a module to reduce repetition and derive values.

## Q11. What is `terraform init`?

> It initializes the working directory, downloads providers/modules, and configures the backend.

## Q12. What is `terraform plan`?

> It calculates and displays the changes Terraform proposes to make.

## Q13. What is `terraform apply`?

> It executes the planned changes against the target infrastructure.

## Q14. What is remote state?

> State stored in a shared remote backend rather than only on a developer's local machine.

## Q15. Why do we need state locking?

> To prevent concurrent Terraform operations from modifying shared state unsafely.

## Q16. What is a module?

> A reusable collection of Terraform configuration that encapsulates infrastructure patterns.

## Q17. `count` vs `for_each`?

> `count` uses numeric indexes and is good for simple repetition; `for_each` uses stable keys and is better for named/distinct instances.

## Q18. What is `depends_on`?

> It explicitly tells Terraform about a dependency that cannot be inferred from resource references.

## Q19. What is drift?

> Drift occurs when real infrastructure changes outside Terraform and no longer matches the desired configuration.

## Q20. How do you detect drift?

> Run `terraform plan` and investigate differences reported between configuration and the observed infrastructure.

## Q21. What does `terraform import` do?

> It associates an existing remote resource with a Terraform state address so Terraform can begin managing it. Configuration still needs to match the real resource.

## Q22. Does import generate Terraform code?

> Traditional import primarily updates state. It does not guarantee a complete production-ready configuration.

## Q23. What is `prevent_destroy`?

> A lifecycle setting that prevents Terraform from destroying a protected resource through normal Terraform operations.

## Q24. What is `ignore_changes`?

> A lifecycle setting telling Terraform to ignore changes to specified attributes when calculating differences.

## Q25. Why should provider versions be pinned?

> To make infrastructure behavior predictable and allow controlled upgrades.

## Q26. Why version Terraform modules?

> To prevent unexpected module changes from affecting production infrastructure.

## Q27. What is the dependency graph?

> Terraform's graph of resource relationships used to determine operation ordering and parallelism.

## Q28. How do you secure Terraform state?

> Use a secure remote backend, encryption, least-privilege access, locking/concurrency controls, audit logging, and appropriate backup/versioning.

## Q29. How do you authenticate Terraform in CI/CD?

> Prefer short-lived/federated identities such as OIDC where supported instead of storing long-lived cloud access keys.

## Q30. How would you structure Terraform for dev/staging/prod?

> Use reusable modules with separate environment root modules and appropriately isolated state/backends, with controlled CI/CD promotion.

---

# 96. Senior-Level Scenarios

## Scenario 1 — Two Engineers Run Apply at the Same Time

Problem:

```text
Engineer A
   ↓
terraform apply

Engineer B
   ↓
terraform apply
```

Solution:

```text
Remote State
+
Concurrency/Locking
```

Also establish CI/CD as the primary production execution path.

---

## Scenario 2 — Production Plan Wants to Destroy Database

Do not run:

```bash
terraform apply
```

Immediately.

Investigate:

```text
Why replacement?
 ↓
Attribute changed?
 ↓
Resource address changed?
 ↓
count/for_each change?
 ↓
Provider upgrade?
 ↓
State issue?
 ↓
Lifecycle?
```

Only apply after the destructive action is understood and approved.

---

## Scenario 3 — Resource Created Manually

Situation:

```text
Azure Portal
 ↓
VNet created manually
```

Terraform needs to manage it.

Approach:

```text
Write configuration
 ↓
terraform import
 ↓
terraform plan
 ↓
Reconcile configuration
 ↓
Plan should become predictable
```

---

## Scenario 4 — Developer Wants Portal Access to Production

Production model:

```text
Terraform
   ↓
Source of Truth
   ↓
CI/CD
   ↓
Production
```

Portal changes create drift.

Allow emergency/manual access only through controlled break-glass procedures where organizational policy requires it.

---

## Scenario 5 — Terraform State Contains Sensitive Values

Treat state as sensitive.

Use:

```text
Encrypted remote backend
+
RBAC
+
Restricted access
+
Audit logs
```

Remember:

```text
sensitive output
≠
value absent from state
```

---

## Scenario 6 — Module Upgrade Breaks Production

Do not automatically use:

```hcl
version = "latest"
```

Instead:

```text
Module version
 ↓
Test in dev
 ↓
Test in staging
 ↓
Review plan
 ↓
Production approval
 ↓
Upgrade
```

---

# 97. Production Terraform Workflow

A mature workflow:

```text
Developer
   ↓
Feature Branch
   ↓
Terraform fmt
   ↓
Validate
   ↓
Security Scan
   ↓
Pull Request
   ↓
Plan
   ↓
Peer Review
   ↓
Merge
   ↓
Production Pipeline
   ↓
Approval
   ↓
Apply Saved Plan
   ↓
Verification
```

---

# 98. Day 10 Checklist

- [ ] I understand Infrastructure as Code.
- [ ] I understand declarative configuration.
- [ ] I understand Terraform Core.
- [ ] I understand providers.
- [ ] I understand resources.
- [ ] I understand data sources.
- [ ] I understand variables.
- [ ] I understand outputs.
- [ ] I understand locals.
- [ ] I understand state.
- [ ] I understand remote state.
- [ ] I understand state locking/concurrency.
- [ ] I understand modules.
- [ ] I understand module versioning.
- [ ] I understand dependency graphs.
- [ ] I understand implicit dependencies.
- [ ] I understand `depends_on`.
- [ ] I understand `count`.
- [ ] I understand `for_each`.
- [ ] I understand lifecycle rules.
- [ ] I understand import.
- [ ] I understand drift.
- [ ] I understand Terraform secrets risks.
- [ ] I understand Azure Terraform.
- [ ] I understand AWS Terraform.
- [ ] I understand Terraform CI/CD.
- [ ] I completed the local Terraform lab.
- [ ] I tested variables and outputs.
- [ ] I tested `count`.
- [ ] I tested `for_each`.
- [ ] I tested state inspection.
- [ ] I answered at least 25 interview questions aloud.

---

# 99. Homework

- [ ] Build the local Terraform lab from scratch without copying.
- [ ] Create a reusable local-file module.
- [ ] Use variables.
- [ ] Use locals.
- [ ] Use outputs.
- [ ] Use `for_each`.
- [ ] Inspect Terraform state.
- [ ] Run `terraform graph`.
- [ ] Intentionally create a configuration drift scenario.
- [ ] Read the resulting plan.
- [ ] Practice importing an existing resource in a safe lab.
- [ ] Create an Azure Resource Group with Terraform if you have Azure access.
- [ ] Create a small AWS resource if you have AWS access.
- [ ] Design a Terraform repository for dev/staging/prod.
- [ ] Explain how you would secure remote state.
- [ ] Explain how you would implement Terraform in Azure DevOps/GitHub Actions.

---

# 100. Final Day 10 Challenge

Without looking at your notes, explain this architecture:

```text
                 Git Repository
                       ↓
                Terraform Code
                       ↓
              CI/CD Validation
                       ↓
                terraform plan
                       ↓
                  Approval
                       ↓
                terraform apply
                       ↓
                 Cloud Provider
                       ↓
                Infrastructure
                       ↓
                Remote State
```

Then answer:

1. Why is Terraform declarative?
2. What does Terraform Core do?
3. What does a provider do?
4. Resource vs data source?
5. Variable vs local?
6. What is Terraform state?
7. Why should state not be stored in Git?
8. What is remote state?
9. Why do we need state locking/concurrency control?
10. What is a module?
11. How should modules be versioned?
12. `count` vs `for_each`?
13. What is an implicit dependency?
14. When would you use `depends_on`?
15. What does `prevent_destroy` do?
16. What does `ignore_changes` do?
17. What is drift?
18. How do you detect drift?
19. What does import do?
20. Why doesn't import automatically solve configuration management?
21. How do you protect Terraform state?
22. How do you authenticate Terraform in CI/CD?
23. How do you manage Terraform secrets?
24. How do you structure dev/staging/prod?
25. How do you prevent an accidental production destroy?
26. How would you handle a module upgrade?
27. How would you handle manually created infrastructure?
28. How would you troubleshoot a locked state?
29. How would you design Terraform for 100+ services?
30. How would you integrate Terraform with your DevOps CI/CD pipeline?

---

# 101. Success Criteria

You are ready for **Day 11 — Advanced Terraform, Modules, Remote State & CI/CD** when you can confidently explain:

```text
Terraform Configuration
        ↓
Terraform Core
        ↓
Dependency Graph
        ↓
Provider
        ↓
Cloud API
        ↓
Infrastructure
        ↓
Remote State
```

You should also be able to explain:

```text
Git
 ↓
Pull Request
 ↓
Validate
 ↓
Security Scan
 ↓
Plan
 ↓
Review
 ↓
Approval
 ↓
Apply
 ↓
Verification
```

Most importantly:

> **Terraform is not just a tool for creating resources. It is a system for managing infrastructure state safely and predictably.**

---

# Next Day

## Day 11 — Advanced Terraform, Modules, Remote State & CI/CD

Topics:

- Production Terraform architecture
- Advanced modules
- Module composition
- Remote state design
- State isolation
- State migration
- State refactoring
- `terraform state mv`
- `moved` blocks
- Import blocks
- Provider aliases
- Multiple subscriptions/accounts
- Terraform dependency patterns
- Advanced `for_each`
- Dynamic blocks
- `locals` and expressions
- CI/CD plan/apply architecture
- Azure DevOps Terraform pipeline
- GitHub Actions Terraform pipeline
- Policy as Code
- Terraform security scanning
- Cost controls
- Drift detection
- Production incident scenarios
- Advanced troubleshooting
- 30+ senior-level interview questions
