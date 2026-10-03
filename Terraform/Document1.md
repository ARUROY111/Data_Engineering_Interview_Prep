# Terraform Interview Q&A: Complete Consolidated Set

Questions 1-35 consolidate **every topic** from `terra1.pdf` (100 Q&As) and `terra2.pdf` (50 Q&As) into one numbered set, with answers current as of mid-2026. Questions 36-40 are bonus topics the PDFs do not cover. A coverage map at the end shows where each original question landed.

Confirm version-specific details on developer.hashicorp.com before an exam or interview.

**Contents**
- A. Fundamentals and Language (1-8)
- B. State and Backends (9-13)
- C. Lifecycle, Import and Refactoring (14-18)
- D. Modules and Composition (19-22)
- E. Versions, Security and Quality (23-27)
- F. Delivery, Scale and Real-World (28-35)
- G. Bonus Topics (36-40)

---

## A. Fundamentals and Language

### 1. What is Terraform, what is IaC, and how does Terraform compare with other tools?

**Infrastructure as Code** manages infrastructure through machine-readable files instead of consoles or ad-hoc scripts. Benefits: version control and audit trail, repeatability, faster provisioning, fewer human errors, code review, and CI/CD for infrastructure.

**Terraform** is a declarative IaC tool from HashiCorp. You describe the *desired end state* in HCL (or JSON). Terraform builds a dependency graph, compares it with recorded state, and computes the create/update/destroy actions.

- **Declarative vs imperative:** declarative says *what* the end state is and the tool works out *how*. Imperative says *how*, step by step. Terraform is declarative.
- **vs Ansible/Puppet/Chef:** those are configuration-management tools for installing and configuring software on existing hosts, often procedural or agent-based. Terraform provisions and manages the lifecycle of infrastructure objects. Teams commonly use both.
- **vs CloudFormation:** CloudFormation is AWS-only and AWS manages its state. Terraform is multi-cloud through providers and keeps its own state.
- **vs Pulumi:** Pulumi uses general-purpose languages (Python, TypeScript, Go). Terraform uses HCL: more standardised and predictable, with a larger ecosystem. Pulumi is more flexible for developer-centric teams.
- **Licence:** since August 2023 Terraform is source-available under the **BSL 1.1**, not open source. **OpenTofu** is the open-source (MPL 2.0) fork under the Linux Foundation. HCL, providers and state are largely compatible, but the projects are diverging (for example OpenTofu's client-side state encryption). A neutral answer: it is a licensing and governance decision; check the terms against your use, test a switch with `plan`, and pin the tool version in CI.

### 2. Describe the core workflow and the essential CLI commands.

**Write, Plan, Apply** (and Destroy).

| Command | Purpose |
|---|---|
| `terraform init` | Downloads providers and modules, configures the backend, creates `.terraform/` and `.terraform.lock.hcl`. Safe to re-run. `-upgrade` updates providers, `-reconfigure` changes backend without migrating, `-migrate-state` migrates state. |
| `terraform fmt` | Rewrites files to canonical style. `-recursive`, `-check` (non-zero exit if changes needed, good for CI). |
| `terraform validate` | Checks syntax and internal consistency (after `init`). Does not contact providers or read state. |
| `terraform plan` | Dry run. Refreshes real state, compares with configuration, shows create/update/destroy. `-out=tfplan` saves it. `-detailed-exitcode` returns 2 if changes exist. |
| `terraform apply` | Executes a plan. Prompts for approval unless `-auto-approve`. Can apply a saved plan (`apply tfplan`). |
| `terraform destroy` | Plans and executes destruction of everything in state (or `-target` items). Equivalent to `apply -destroy`. |
| `terraform console` | Interactive expression evaluator. |
| `terraform graph` | Dependency graph in DOT format (`terraform graph \| dot -Tpng > graph.png`). |
| `terraform output / show / providers / version` | Inspect outputs, state/plan, provider requirements, versions. |

`-auto-approve` is for well-tested pipelines only; on a laptop it removes the safety net.

`validate` vs `fmt`: `validate` checks correctness, `fmt` checks style. Run both in CI.

### 3. What are providers, resources, data sources and the Registry?

- **Provider:** a plugin that turns Terraform resources and data sources into API calls for a platform (AWS, Azure, GCP, Kubernetes, SaaS). Declared in `required_providers`, configured with `provider` blocks, downloaded at `init`.
- **Resource:** an infrastructure object Terraform creates and manages. Address: `type.name` (for example `aws_instance.web`). Tracked in state.
- **Data source:** a read-only lookup of existing information (an AMI, VPC, secret). Uses a `data` block. Creates or changes nothing.
- **Registry (registry.terraform.io):** public index of providers and modules, with versioning and docs. HCP Terraform offers a private registry.

```hcl
terraform {
  required_providers {
    aws = { source = "hashicorp/aws", version = "~> 5.40" }
  }
}

provider "aws" { region = "ap-south-1" }

resource "aws_s3_bucket" "example" { bucket = "my-unique-bucket-name" }

data "aws_ami" "al2023" {
  most_recent = true
  owners      = ["amazon"]
  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }
}
```

Data source filters make configuration dynamic instead of hard-coding IDs.

### 4. Explain input variables, outputs, locals and complex types.

- **Variables** parameterise configuration. A `variable` block can set `type`, `default`, `description`, `validation`, `sensitive`, and `nullable`.
- **Outputs** expose values after apply (IPs, endpoints, IDs). They feed other modules, `terraform_remote_state`, and CI. Mark `sensitive = true` to redact them from CLI output.
- **Locals** name an expression for reuse inside a module (computed prefixes, merged tags). They are evaluated once and are module-scoped.
- **Types:** primitives (`string`, `number`, `bool`), collections (`list`, `set`, `map`), structural (`object`, `tuple`), and `any`. `optional()` with defaults keeps object inputs ergonomic.

```hcl
variable "glue_job" {
  type = object({
    name        = string
    worker_type = optional(string, "G.1X")
    workers     = optional(number, 2)
  })
}

variable "environment" {
  type = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Must be dev, staging, or prod."
  }
}

locals {
  name_prefix = "${var.project}-${var.environment}"
  common_tags = { Project = var.project, ManagedBy = "terraform" }
}

output "bucket_arn" { value = aws_s3_bucket.example.arn }
```

### 5. How do you pass different values per environment, and what is the variable precedence?

Ways to supply values: `-var`, `-var-file`, `terraform.tfvars`, `*.auto.tfvars`, `TF_VAR_name` environment variables, interactive prompts, and in HCP Terraform workspace variables and variable sets.

**Precedence (later overrides earlier):**
1. `default` in the variable block
2. `TF_VAR_*` environment variables
3. `terraform.tfvars`, then `terraform.tfvars.json`
4. `*.auto.tfvars` / `*.auto.tfvars.json` (alphabetical order)
5. `-var` and `-var-file` on the command line (in the order given)

`terraform.tfvars` and `*.auto.tfvars` load automatically; other files need `-var-file`. Typical pattern: `environments/prod/prod.tfvars` per environment, with separate state per environment (see Q13 and Q35).

### 6. How does Terraform order operations? (dependencies and the graph)

Terraform builds a directed acyclic graph.

- **Implicit dependencies:** created automatically when one block references another's attribute (`aws_instance.web.id`). Preferred.
- **Explicit dependencies:** `depends_on = [...]` for hidden dependencies Terraform cannot infer (for example an IAM policy attachment that must exist before a service uses the role). Overuse makes code harder to maintain.
- The graph is walked in topological order. Independent nodes run in parallel (`-parallelism`, default 10). Destroy runs in reverse order.
- **Cycles** (`Error: Cycle`) mean a design problem. Fix by removing one direction of dependency, adding an intermediate resource, or splitting into separate applies. Use `terraform graph` to visualise.

### 7. `count` vs `for_each`: how do they differ, and what goes wrong with each?

- **`count`** creates N instances addressed by index (`aws_instance.web[0]`). Removing an item from the middle of a list shifts indices, so later resources are destroyed and recreated.
- **`for_each`** takes a map or set and addresses instances by key (`aws_instance.web["web1"]`). Adding or removing a key only affects that instance. Prefer it for dynamic collections.
- Use `count` for simple repetition or an on/off toggle (`count = var.enabled ? 1 : 0`).

**Common error:** `The "for_each" map includes keys derived from resource attributes that cannot be determined until apply.` Keys must be known at plan time. Fixes: key on static values and use computed values only as map *values*; split into two applies or stacks; read existing objects through a data source; as a last resort `-target` for bootstrapping.

```hcl
# Fails: keys depend on unknown IDs
for_each = toset(aws_subnet.private[*].id)

# Works: static keys, computed values
for_each = { for k, s in aws_subnet.private : k => s.id }
```

### 8. What are dynamic blocks, expressions and built-in functions?

- **Dynamic blocks** generate repeated nested blocks (ingress rules, EBS volumes) from a collection, using `for_each` and a `content` block.
- **Expressions:** conditionals (`cond ? a : b`), `for` expressions (`[for s in var.list : upper(s)]`), splat (`aws_instance.web[*].id`), string templates.
- **Functions** are built in and pure: string (`join`, `split`, `format`, `replace`), collection (`merge`, `flatten`, `lookup`, `keys`, `toset`), type conversion (`tostring`, `tonumber`), encoding (`jsonencode`, `yamlencode`, `base64encode`), filesystem (`file`, `templatefile`). Providers can also ship **provider-defined functions** (1.8+): `provider::aws::arn_parse(...)`.
- **Path references:** `path.module` (this module's directory, most common), `path.root`, `path.cwd`.

```hcl
resource "aws_security_group" "example" {
  name = "example"
  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = [ingress.value.cidr]
    }
  }
}
```

Test expressions interactively with `terraform console`.

---

## B. State and Backends

### 9. What is Terraform state, what does it contain, and why is local state risky?

State (`terraform.tfstate`, JSON) maps configuration to real-world objects. It stores resource IDs, attributes, dependencies and metadata, so Terraform knows what exists, can compute plans, and can order updates and destroys. Top-level fields include `version`, `terraform_version`, `serial`, `lineage`, `outputs` and `resources` (mode, type, name, provider, instances).

**Risks of local state in a team:** it falls out of sync between people, can be lost or corrupted, has no locking (race conditions), and has no encryption, versioning or access control. State often contains **plaintext secrets**, so treat it as sensitive: never commit `*.tfstate*`, encrypt it, and restrict access.

Deleting the state file does not delete infrastructure. Terraform just loses track of it, leaving orphaned, billable resources. Use `terraform destroy` for clean teardown.

### 10. How do backends and state locking work? (S3, HCP Terraform, concurrent applies, lock errors, bootstrapping)

A **backend** decides where state is stored. The default is local. Remote options: S3, Azure Blob, GCS, HCP Terraform, Consul and others. Remote backends enable collaboration, encryption, versioning and (usually) locking.

**S3 with locking:** from Terraform **1.10**, the S3 backend supports **native locking** with `use_lockfile = true` (a `.tflock` object next to state). The `dynamodb_table` approach is **deprecated**, but you will still see it in existing code, so know both.

```hcl
terraform {
  backend "s3" {
    bucket       = "company-terraform-state"
    key          = "network/terraform.tfstate"
    region       = "ap-south-1"
    encrypt      = true
    use_lockfile = true      # legacy alternative: dynamodb_table = "terraform-state-lock"
  }
}
```

Enable bucket **versioning** (your recovery path). Partial configuration with `-backend-config=file` keeps environment values out of code.

**Two people apply at once:** with locking, the second run **fails immediately** with `Error acquiring the state lock`. It waits only if you pass `-lock-timeout=5m`. HCP Terraform **queues** runs per workspace. Without locking, both runs can corrupt state.

**Handling a lock error:** read the error (lock ID, holder, time). Check whether a job is genuinely still running. Only if the process died, run `terraform force-unlock <LOCK_ID>`, then `plan` to check for a half-applied state. Prevent with CI concurrency groups.

**Bootstrapping (chicken-and-egg):** the bucket holding state cannot be created by the config that stores state in it. Either create it with a small local-state bootstrap config and then `init -migrate-state`, create it once with CloudFormation or a script, or use HCP Terraform. Give the bucket versioning, KMS encryption, a public access block, and `prevent_destroy`.

### 11. Which `terraform state` commands matter, and how do you migrate or recover state?

**Commands (they change state only, never infrastructure):**
- `state list`, `state show ADDR`: inspect
- `state mv OLD NEW`: rename or move an address
- `state rm ADDR`: stop tracking without destroying
- `state pull` / `state push`: raw access (dangerous)
- `state replace-provider`: change provider source

**Migrating between backends** (local to S3, S3 to HCP Terraform, and so on): change the `backend` or `cloud` block, run `terraform init -migrate-state`, confirm the copy, run `plan` (should be clean), commit, and have teammates re-`init`. `-reconfigure` switches backend *without* migrating.

**Recovering lost or corrupted state:**
1. Restore a previous object version from backend versioning or backups (primary path).
2. Minor corruption: `state pull`, repair JSON, `state push` (last resort).
3. No backup: re-import every resource (`import` blocks help) and `plan` until clean.
4. Afterwards, add versioning, backups, restricted access, and write an incident note.

### 12. What is drift and how do you detect, handle and prevent it?

Drift is real infrastructure diverging from state or configuration, usually through console changes.

- **Detect:** `terraform plan` refreshes and shows differences. `plan -refresh-only` shows *only* state-vs-reality differences. The standalone `terraform refresh` command is deprecated.
- **Accept into state:** `terraform apply -refresh-only` (shows the proposed state update, asks approval).
- **Revert:** run a normal `apply` so infrastructure matches configuration.
- **Codify:** update configuration to match reality, then apply.
- **Legitimately external attributes** (autoscaling desired capacity, tags added by another tool): `lifecycle { ignore_changes = [...] }`.

**Examples:** someone edits a security group rule in the console, and the next `plan` proposes reverting it. Someone deletes a resource manually, and `plan` proposes recreating it. To stop managing it instead, see Q16.

**Continuous detection:**
```bash
terraform plan -detailed-exitcode -input=false
# 0 = no changes, 1 = error, 2 = changes present (drift)
```
Schedule it in CI, or use HCP Terraform health assessments. Prevent drift with IAM/SCP limits on console writes in production and CloudTrail alerts on out-of-band changes.

### 13. Workspaces vs separate state per environment: which do you use?

- **CLI workspaces:** multiple state files for one configuration in one backend (`terraform workspace new/select/list/show`, `terraform.workspace` in code). Quick and share code, but they share one backend and credentials, so isolation is weak and cross-environment mistakes are easy.
- **Separate directories or repos with separate backends/state:** better isolation, access control and blast radius. Preferred for production.
- **HCP Terraform workspaces:** a richer concept (own state, variables, permissions, run history, policies), usually one per environment or component.

```hcl
locals {
  env_config = {
    dev  = { instance_type = "t3.micro" }
    prod = { instance_type = "t3.large" }
  }
  config = local.env_config[terraform.workspace]
}
```

---

## C. Lifecycle, Import and Refactoring

### 14. Explain the `lifecycle` block and how to replace resources safely (including zero-downtime ASG replacement).

| Argument | Purpose |
|---|---|
| `create_before_destroy` | Create the replacement first, then destroy the old one. Needs resources that can coexist (no unique-name clash). The graph is reordered, and dependents are updated to point at the new object between create and destroy. |
| `prevent_destroy` | Plans that would destroy the resource error out. A guardrail for critical resources (does not stop removal from config followed by `state rm`). |
| `ignore_changes` | Ignore drift on listed attributes. |
| `replace_triggered_by` | Force replacement when another resource or attribute changes. |
| `precondition` / `postcondition` | Assertions (see Q25). |

**Forcing replacement:** `terraform apply -replace=ADDR` (replaces the deprecated `terraform taint`), `replace_triggered_by`, or changing a force-new argument.

**Zero-downtime ASG replacement:** keep the ASG behind a load balancer; create a new launch template version or new ASG with `create_before_destroy`; attach to the same target group; wait for health checks (use `instance_refresh` or a name derived from the launch template so a new ASG is created); then the old one drains and is destroyed. For true progressive delivery, pair Terraform with traffic-shifting tools.

```hcl
resource "aws_instance" "web" {
  # ...
  lifecycle {
    create_before_destroy = true
    prevent_destroy       = true
    ignore_changes        = [tags]
    replace_triggered_by  = [aws_security_group.web.id]
  }
}
```

### 15. How do you import existing infrastructure?

Use **`import` blocks** (1.5+). They run in the normal plan/apply flow and are reviewable in a PR.

```hcl
import {
  to = aws_s3_bucket.logs
  id = "my-existing-logs-bucket"
}
```
```bash
terraform plan -generate-config-out=generated.tf   # drafts the resource block
terraform apply
```

- `-generate-config-out` writes a draft. Clean it up (remove computed/default attributes, parameterise, move into modules) before committing.
- Since 1.7, `import` blocks support `for_each` to import many resources at once.
- The classic CLI form still works (`terraform import aws_instance.web i-0abc...`), but you must hand-write the resource block first.
- Run `plan` afterwards to confirm no unexpected changes. The `import` blocks can then be removed.

### 16. How do you refactor safely: rename, move between modules, or stop managing a resource?

- **Rename or move (including across modules):** `moved` block (1.1+). Declarative, shared with the team, and confirmed by a `plan` that shows only an address change.
- **Stop managing without destroying:** `removed` block (1.7+) with `destroy = false`. Without it, deleting a resource block **destroys** the real object on the next apply.
- **Imperative alternatives:** `terraform state mv` and `terraform state rm`.

```hcl
moved {
  from = module.legacy.aws_instance.web
  to   = module.platform.aws_instance.web
}

removed {
  from = aws_s3_bucket.legacy
  lifecycle { destroy = false }
}
```

### 17. What are provisioners, and what are the alternatives?

Provisioners (`local-exec`, `remote-exec`, `file`) run scripts at create or destroy time. HashiCorp treats them as a **last resort**: they are imperative, not modelled in plan, and failures can leave resources half-configured. Prefer `user_data`/cloud-init, Packer images, or configuration management.

When you need an escape hatch, use **`terraform_data`** (1.4+, built in, no extra provider), which replaces `null_resource`.

```hcl
resource "terraform_data" "notify" {
  triggers_replace = [aws_glue_job.etl.id]
  provisioner "local-exec" { command = "echo Glue job updated" }
}
```

### 18. How does Terraform handle partial failures, eventual consistency and `-target`? How do you recover from an interrupted destroy?

- **Partial apply failure:** resources already created stay in state; later ones are not attempted. There is **no automatic rollback**. Fix the cause and re-apply.
- **Eventual consistency:** providers retry and poll with timeouts. Tune with a `timeouts {}` block when applies are slow or flaky.
- **`-target`:** limits a run to a resource or module **and its dependencies**. Everything else stays un-applied, which can hide drift, and Terraform warns about this. Use only for recovery or bootstrapping.
- **Interrupted `destroy` in production:** do not panic. Run `plan` to compare state with configuration. Re-`apply` to recreate what was destroyed. For half-deleted objects, `state rm` and re-import or recreate. Check provider recycle bins and backups. Then add guardrails: `prevent_destroy`, policy as code, approval gates, least-privilege IAM.

---

## D. Modules and Composition

### 19. What are modules, and how do you source, version and compose them?

A module is a directory of `.tf` files. The directory where you run Terraform is the **root module**; modules it calls are **child modules**. Modules give reuse, encapsulation and consistency. Data flows only through **input variables** (parent to child) and **outputs** (child to parent, read as `module.name.output`).

**Sources:** local path, Terraform Registry, Git (`git::https://...//subdir?ref=v1.2.0`), HTTP archives, private registry. Always pin versions of remote modules.

```hcl
module "network" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"
  name    = "my-vpc"
  cidr    = "10.0.0.0/16"
}

module "compute" {
  source    = "./modules/compute"
  vpc_id    = module.network.vpc_id
  subnet_id = module.network.private_subnets[0]
}
```

**Good practices:** typed `object` variables for clean interfaces; keep nesting shallow; do not put `provider` blocks inside reusable modules (declare `required_providers` with `configuration_aliases` and let callers pass providers); use `for_each`/`count` on modules where useful.

**When not to use a module:** small or one-off infrastructure, early prototypes, or when abstraction costs more maintenance than the reuse saves.

**Staying DRY:** modules, locals, variables, `for_each`, dynamic blocks, shared tfvars.

### 20. How do you design, release, upgrade and test modules?

- **Release:** one repo per module (or a monorepo with path filters), semver Git tags (`v1.2.3`), README, `examples/`, CHANGELOG, and CI tests on every tag. Publish to the public Registry or a private registry.
- **Consuming:** pin with `~> 1.2` (flexible) or exact versions (strict).
- **Breaking major upgrade:** read the upgrade guide, bump in a feature branch, fix renamed variables/outputs, run `plan` in non-prod and review every change, apply, test, promote through environments. Dependabot/Renovate can open PRs, but a human reviews the plan.
- **Moving resources between module versions:** use `moved` blocks to avoid recreation.
- **Monorepo compatibility:** versioned modules, contract tests on variable/output schemas, example-apply CI, pinned versions in consumers, CI path filtering so only affected stacks are planned.

### 21. How do you manage multiple regions, accounts or clouds?

Use **provider aliases**, each with its own region or assumed role, and select them with the `provider` meta-argument or pass them into modules via `providers = {}`.

```hcl
provider "aws" { region = "ap-south-1" }

provider "aws" {
  alias  = "workload"
  region = "ap-south-1"
  assume_role {
    role_arn = "arn:aws:iam::${var.workload_account_id}:role/TerraformExecution"
  }
}

resource "aws_s3_bucket" "x" {
  provider = aws.workload
  bucket   = "workload-bucket"
}
```

**Multi-account pattern:** a deployment account assumes least-privilege roles into workload accounts; state lives in a dedicated account. **Multi-cloud:** resource types are provider-specific, so avoid a single "cloud-agnostic" module. Build one module per cloud with the **same input/output contract** and compose them from the root module.

### 22. How do you share data between separate Terraform configurations?

- **`terraform_remote_state`:** reads the *root outputs* of another state. Simple but creates tight coupling, and the reader needs read access to the **entire state file**, which may contain secrets.
- **Better:** look things up with **data sources** (by tags or names), publish values to SSM Parameter Store or Secrets Manager, or use `tfe_outputs` in HCP Terraform.
- **Within one config:** use module outputs and variables.

```hcl
data "terraform_remote_state" "network" {
  backend = "s3"
  config = {
    bucket = "company-terraform-state"
    key    = "network/terraform.tfstate"
    region = "ap-south-1"
  }
}

resource "aws_instance" "app" {
  subnet_id = data.terraform_remote_state.network.outputs.public_subnet_ids[0]
}
```

---

## E. Versions, Security and Quality

### 23. How do you manage Terraform, provider and module versions?

| Constraint | Meaning |
|---|---|
| `= 5.40.0` | exactly that version |
| `>= 5.0` | that or newer, including the next major (risky) |
| `~> 5.40` | `>= 5.40, < 6.0` |
| `~> 5.40.0` | `>= 5.40.0, < 5.41.0` |

- `required_version` in the `terraform` block constrains the CLI version (for example `>= 1.5, < 2.0`).
- `required_providers` holds provider **constraints**. The **`.terraform.lock.hcl`** file pins the **exact provider versions and hashes** chosen at `init`. It covers **providers only, not modules**. Commit it.
- **Upgrading a provider:** read the changelog, bump the constraint, `terraform init -upgrade`, review the `plan`, apply in non-prod first. Major bumps often break.
- **Multiple platforms:** a lock file generated on a Mac lacks Linux hashes. Run `terraform providers lock -platform=linux_amd64 -platform=linux_arm64 -platform=darwin_arm64`.
- **`.terraform/`** is a disposable local cache (providers, modules, backend config). Never commit it; `init` recreates it.
- Pin module versions with `version =` or Git `?ref=`.

### 24. How do you handle secrets in Terraform?

`sensitive = true` only **redacts CLI output**. The value is still stored in **state and plan files in plaintext**.

- **Ephemeral resources, variables and outputs (1.10+):** exist only during the run and are never persisted.
- **Write-only arguments (1.11+):** sent to the provider but never stored in state (typically named `*_wo`; needs provider support).

```hcl
ephemeral "aws_secretsmanager_secret_version" "db" {
  secret_id = aws_secretsmanager_secret.db.id
}

resource "aws_db_instance" "main" {
  # ...
  password_wo         = ephemeral.aws_secretsmanager_secret_version.db.secret_string
  password_wo_version = 1    # bump to rotate
}
```

**Layered approach:**
1. Keep secrets out of state where possible (ephemeral, write-only).
2. Encrypt state with KMS; restrict access with IAM; enable access logging.
3. Mark variables and outputs sensitive.
4. Never commit `*.tfstate` or secret `.tfvars`.
5. Source secrets at runtime from Secrets Manager, SSM or Vault, or generate with the `random` provider and store them in a secrets manager.
6. Use OIDC or IAM roles instead of static cloud credentials.

### 25. `validation`, `precondition`, `postcondition` and `check`: what is the difference?

| Feature | Where | Checks | Failure |
|---|---|---|---|
| `validation` | `variable` block | The input value | Error |
| `precondition` | lifecycle of `resource`, `data`, `output`, `module` | Assumptions before use/creation | Error |
| `postcondition` | same | Guarantees after the object exists | Error |
| `check` | Top-level block (may hold its own scoped data source) | Any assertion | **Warning only** |

```hcl
resource "aws_instance" "app" {
  ami           = data.aws_ami.al2023.id
  instance_type = var.instance_type
  lifecycle {
    precondition {
      condition     = data.aws_ami.al2023.architecture == "x86_64"
      error_message = "AMI architecture must be x86_64."
    }
    postcondition {
      condition     = self.public_ip != ""
      error_message = "Instance must get a public IP."
    }
  }
}

check "health" {
  data "http" "app" { url = "https://${aws_lb.main.dns_name}/health" }
  assert {
    condition     = data.http.app.status_code == 200
    error_message = "Health endpoint is not returning 200."
  }
}
```

Rule of thumb: bad input is `validation`; a wrong assumption is `precondition`; a bad result is `postcondition`; an ongoing signal that should not block is `check`.

### 26. How do you test Terraform code?

A testing pyramid:
1. **Static:** `fmt -check`, `validate`, TFLint, Trivy/Checkov.
2. **Unit-style:** native `terraform test` (1.6+) with **mock providers** (1.7+), no cloud credentials needed. Tests are `*.tftest.hcl`.
3. **Integration:** `terraform test` with real applies in a sandbox account, or Terratest (Go).
4. **Policy tests:** OPA/Conftest or Sentinel against the plan.
5. **Plan-only assertions** in CI (resource counts, attributes).

```hcl
# tests/s3.tftest.hcl
mock_provider "aws" {}

variables { bucket_name = "demo-bucket" }

run "bucket_is_encrypted" {
  command = plan
  assert {
    condition     = aws_s3_bucket_server_side_encryption_configuration.this.rule[0].apply_server_side_encryption_by_default[0].sse_algorithm == "aws:kms"
    error_message = "Bucket must use KMS encryption."
  }
}
```

Always test changes in a sandbox or staging account before production.

### 27. How do you enforce policy, linting, tagging and cost control?

- **Linters and scanners:** TFLint (static analysis), Trivy (absorbed tfsec) and Checkov (misconfigurations).
- **Policy as code:** Sentinel (HCP Terraform/Enterprise) and OPA/Conftest evaluate the plan. Sentinel enforcement levels: advisory, soft-mandatory (overridable), hard-mandatory.
- **Tagging:** provider `default_tags`, a required `tags` variable in module interfaces, plus a policy that rejects untagged resources.
- **Cost:** Infracost or HCP Terraform cost estimation on every PR.
- **IAM:** separate plan (read-mostly) and apply (write) roles; restrict powerful permissions such as `iam:*`.

```hcl
provider "aws" {
  region = "ap-south-1"
  default_tags {
    tags = { ManagedBy = "terraform", Environment = var.environment }
  }
}
```

Defence in depth: no single control is enough.

---

## F. Delivery, Scale and Real-World

### 28. Describe a CI/CD pipeline for Terraform (GitOps, approvals, and secure cloud authentication).

**Flow:** PR opened, CI runs `fmt -check`, `validate`, TFLint, a security scanner and `plan` (posted as a PR comment). After review and approval, merge to `main`. A second job (or the HCP Terraform VCS workflow) runs `apply`, with a manual approval gate for production. State is remote and locked; everything is auditable through Git.

**Approval gates:** PR review of the plan, GitHub **environment protection rules** (or GitLab manual jobs), HCP Terraform manual confirm and Sentinel soft-mandatory overrides. (HCP Run Tasks call external services such as scanners or cost tools; they are not approval gates themselves.)

**Authentication without static keys (GitHub OIDC to AWS):**

```yaml
permissions:
  id-token: write
  contents: read
  pull-requests: write

jobs:
  plan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: hashicorp/setup-terraform@v3
        with: { terraform_version: 1.10.5 }
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::111122223333:role/gha-terraform-plan
          aws-region: ap-south-1
      - run: terraform fmt -check -recursive
      - run: terraform init -input=false
      - run: terraform validate
      - run: terraform plan -input=false -out=tfplan

  apply:
    needs: plan
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production           # approval gate
    steps:
      - uses: actions/checkout@v4
      - uses: hashicorp/setup-terraform@v3
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::111122223333:role/gha-terraform-apply
          aws-region: ap-south-1
      - run: terraform init -input=false
      - run: terraform apply -input=false -auto-approve
```

**Security checklist:** trust policy conditioned on repo and branch (`token.actions.githubusercontent.com:sub`); separate plan and apply roles; ideally apply the saved plan artifact rather than re-planning; concurrency group per state; locked-down runners; scanners in CI; short-lived credentials; encrypted state; audit logging. Jenkins and GitLab CI follow the same stages (`init`, `plan`, approval, `apply`).

### 29. What is HCP Terraform, and how does it differ from the CLI and open-source workflows?

HCP Terraform (formerly Terraform Cloud; self-hosted is Terraform Enterprise) provides:
- Remote state with locking, encryption and versioning
- **Remote runs** (plan/apply on managed workers, queued per workspace) vs **local runs** (CLI on your machine, state in HCP)
- VCS-, CLI- and API-driven workflows
- Policy as code (Sentinel, OPA), run tasks, cost estimation
- Teams and RBAC, variable sets, private module/provider registry, audit logs
- Health assessments (drift detection)

HCP workspaces are first-class objects with their own state, variables, run history, permissions and policy sets, far richer than CLI workspaces (Q13). Remote runs give a consistent environment and better secret handling than developer laptops.

### 30. How do you manage Terraform at organisational scale (many teams, hundreds of states) and keep it fast?

- **Small, focused states** by component and environment. Never one giant state.
- Remote, encrypted, locked backends with strict IAM/RBAC; clear ownership per workspace/state.
- HCP Terraform or Terraform Enterprise for collaboration, policy and run governance; automated workspace creation (API or the `tfe` provider).
- A private module registry, semver, CI templates reused across teams, plan on PR and review before apply.
- **Terragrunt:** a thin wrapper adding DRY backend/provider generation, cross-stack `dependency` blocks, and `run-all`. Use it when many environments multiplied by many components force repeated backend and provider code. Plain Terraform plus modules is enough for a few stacks. Alternatives: HCP Terraform run triggers, HCP Terraform Stacks, Atmos. Every extra tool is another failure mode and onboarding cost.
- **Performance:** split state; `-refresh=false` only for quick iteration (never for the final apply); `-parallelism` carefully (API rate limits); avoid data sources that list huge sets; cache providers in CI with `TF_PLUGIN_CACHE_DIR`; keep Terraform and providers current.

### 31. How do you debug a failing Terraform run?

1. Read the error closely; many are configuration or permission issues.
2. `terraform validate` and `fmt` for syntax; `terraform console` to test expressions.
3. `export TF_LOG=TRACE` (or `DEBUG`) and `TF_LOG_PATH=terraform.log` for verbose logs.
4. Inspect state: `terraform state list`, `terraform state show ADDR`, `terraform show`.
5. `terraform plan -refresh-only` to separate drift from configuration issues.
6. Lock problems: see Q10. Unknown `for_each` keys: see Q7. Cycles: see Q6.
7. Check provider docs, changelog and open issues; try a minimal reproduction.

### 32. How would you implement blue/green or canary deployments with Terraform?

Terraform provisions the infrastructure; traffic shifting is usually done by a load balancer, DNS or service mesh.

1. Two parallel stacks (blue and green) driven by a variable or workspace, with weighted routing (ALB weighted target groups, Route 53 weighted records) to shift traffic gradually.
2. `create_before_destroy` on launch templates or ASGs behind a load balancer.
3. Application deployments handled by a separate CI/CD or progressive-delivery tool, with Terraform owning only the underlying infrastructure.

Terraform on its own is not a progressive-delivery tool.

### 33. How would you manage a data platform (Glue, Redshift, S3, Airflow) with Terraform?

**In Terraform:** S3 buckets per layer (landing, raw, curated), KMS keys, IAM roles and policies, Glue databases/jobs/crawlers/connections, Redshift cluster or Serverless namespace and workgroup, SNS/SQS, MWAA environment, networking.

**Usually not in Terraform:** DAG code, Glue script contents and frequently changing table schemas. Deploy those through CI/CD (S3 sync, dbt, migration tools).

```hcl
resource "aws_glue_job" "etl" {
  name              = "${local.prefix}-orders-etl"
  role_arn          = aws_iam_role.glue.arn
  glue_version      = "4.0"
  worker_type       = "G.1X"
  number_of_workers = var.glue_workers

  command {
    name            = "glueetl"
    script_location = "s3://${aws_s3_bucket.scripts.bucket}/orders_etl.py"
  }

  default_arguments = {
    "--enable-metrics"      = "true"
    "--TempDir"             = "s3://${aws_s3_bucket.tmp.bucket}/glue/"
    "--job-bookmark-option" = "job-bookmark-enable"
  }

  lifecycle { ignore_changes = [number_of_workers] }   # if operators tune it
}
```

Talking points: one module per component, separate state per layer (network, storage, compute, orchestration), `ignore_changes` for values operators legitimately tune, per-environment tfvars, drift checks in CI.

### 34. What are the common production pitfalls, how do you run a break-glass procedure, and how do you tell a failure story?

**Pitfalls and fixes:**
1. Monolithic state: split by component.
2. No state locking: use a locking backend.
3. Secrets in state or VCS: ephemeral/write-only values, external stores, encryption.
4. Unpinned providers/modules: constraints plus the lock file.
5. Manual changes and chronic drift: process, restricted console access, drift detection.
6. Over-use of provisioners: prefer declarative alternatives.
7. No plan review: plan on PR, approvals before apply.
8. No testing: at least validate, plan and scanners in CI.
9. Wrong workspace or backend: separate states per environment, approval gates for destroy, `prevent_destroy`.

**Break-glass:** a documented, audited, time-boxed process that elevates privileges or bypasses normal approvals (a dedicated emergency role/workspace), logs every action, and is followed by mandatory review and state reconciliation. It should be rare.

**Failure story (STAR):**
- *Situation:* environment, blast radius, stakes (one sentence).
- *Task:* your responsibility.
- *Action:* concrete steps (stopped further applies, read plan and state, restored a state version, `state rm` or re-import, re-applied).
- *Result:* outcome in time and impact, then the systemic guardrails added.

Avoid blame and the generic "ran destroy in the wrong workspace" story. Use a real incident of your own.

### 35. How do you structure a production-grade Terraform repository and multi-environment setup?

```
repo/
├── modules/                 # reusable, versioned building blocks
│   ├── network/
│   ├── compute/
│   └── database/
├── environments/            # thin root modules, one backend + tfvars each
│   ├── dev/
│   ├── stage/
│   └── prod/
├── tests/                   # *.tftest.hcl and integration tests
├── .terraform.lock.hcl      # committed
└── .github/workflows/       # fmt, validate, scan, plan on PR; apply on merge
```

Principles: separate state per environment (and per component), thin root modules calling shared modules with different variables, pinned versions, remote locked encrypted state, CI-driven plan and apply, README and examples, clear ownership, and `.gitignore` for `.terraform/` and `*.tfstate*`.

---

## G. Bonus Topics

### 36. How do you read a plan, and what decides update-in-place vs replace?

| Symbol | Meaning |
|---|---|
| `+` | create |
| `-` | destroy |
| `~` | update in place |
| `-/+` | destroy then create (replace) |
| `+/-` | create then destroy (replace with `create_before_destroy`) |
| `<=` | read (data source) |

Look for `# forces replacement` next to an argument: that argument cannot be changed in place. Also note `(known after apply)` values and redacted `(sensitive value)`. Review replacements with extra care because they can cause downtime or data loss. Save plans with `-out` and inspect them with `terraform show tfplan` or `terraform show -json tfplan` (useful for policy checks).

### 37. What if the provider does not support a resource or feature yet?

In order of preference:
1. Check for a newer provider version or an open upstream issue; contribute if feasible.
2. Use the **`awscc`** provider (AWS Cloud Control API) for newer AWS resources.
3. Wrap it in an **`aws_cloudformation_stack`** resource.
4. Use a generic provider such as `restapi` or `http` for API-driven objects.
5. Build a custom provider with the Terraform Plugin Framework.
6. As a last resort, `terraform_data` with `local-exec` (imperative, no real state tracking).

### 38. How do you secure access to the state backend itself?

State contains secrets and the full map of your infrastructure, so treat the backend as sensitive.
- Dedicated state bucket (often in a separate account), KMS encryption, versioning, public access block, and deny `s3:DeleteObject` or require MFA delete where appropriate.
- Least-privilege IAM, with per-prefix policies so a team can only reach its own state keys.
- Separate roles for plan (read) and apply (write).
- Access logging and CloudTrail alerts on state reads and writes.
- Remember `terraform_remote_state` requires read access to the whole state, so prefer published outputs (SSM parameters) for cross-team sharing.

### 39. How do you run Terraform in restricted or air-gapped environments?

- **Provider mirror:** `terraform providers mirror /path` downloads providers, and the CLI config's `provider_installation` block points to a `filesystem_mirror` or `network_mirror`.
- **Private registry** for modules and providers (HCP Terraform or a self-hosted registry), or vendored modules from internal Git.
- **Lock file hashes** for every platform you run on (`terraform providers lock -platform=...`).
- **Self-hosted runners/agents** (for example HCP Terraform agents) so runs execute inside the private network without opening inbound access.
- Plugin cache (`TF_PLUGIN_CACHE_DIR`) to avoid repeated downloads.

### 40. What is immutable infrastructure, and how does Terraform fit?

Instead of patching running servers, build a new image (Packer/AMI) and have Terraform **replace** the instance or launch template. Benefits: no configuration drift on hosts, simple rollback (previous AMI), consistent environments. Terraform's replacement semantics (`create_before_destroy`, `replace_triggered_by`, `-replace`) match that model. The contrast is mutable configuration management (Ansible), which changes machines in place.

---
