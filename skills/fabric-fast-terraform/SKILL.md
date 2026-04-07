---
name: fabric-fast-terraform
description: "Architect, provision, and operate production-grade GCP organizations using Terraform and the Google Cloud Foundation Fabric FAST framework. Covers the full FAST stage lifecycle (org setup, resource hierarchy, networking, project factory, workloads), factory-driven YAML patterns, secure CI/CD via Cloud Build + Workload Identity Federation, and GCS-backed remote state. Enforces Korvenx cloud policy: all terraform apply runs must go through Cloud Build — never executed locally."
version: "1.0.0"
---

# Fabric FAST Terraform Skill

Expert guidance for managing GCP infrastructure at scale using [Google Cloud Foundation Fabric FAST](https://github.com/GoogleCloudPlatform/cloud-foundation-fabric/tree/master/fast).

## 🎯 When to Use This Skill

* Setting up or extending a GCP organization using the FAST framework
* Working in the `korvenix-gcp-iac` repository (any stage directory)
* Creating or modifying Terraform for: org policies, resource hierarchy, networking, projects, Cloud Run, Artifact Registry, DNS, HTTPS LB, Cloud CDN, or projects.
* Designing or debugging Cloud Build CI/CD pipelines for IaC
* Troubleshooting stage bootstrap or output contract failures

***

## 🏗️ Fabric FAST Stage Architecture

FAST organizes GCP org setup into sequential stages. Each stage is an independent Terraform root module. Outputs of each stage are consumed as inputs by downstream stages via output files or remote state.

### Stage Map

| Stage                 | Directory                   | Purpose                                                                |
| --------------------- | --------------------------- | ---------------------------------------------------------------------- |
| **0**                 | `stages/0-org-setup`        | Bootstrap: automation SAs, state buckets, org-level IAM, WIF for CI/CD |
| **1-vpcsc**           | `stages/1-vpcsc`            | (Optional) VPC Service Controls perimeters                             |
| **2-security**        | `stages/2-security`         | Centralized KMS, VPC SC perimeters, security sinks                     |
| **2-networking**      | `stages/2-networking`       | Hub-and-spoke VPC, NCC, VPN, NVA                                       |
| **2-project-factory** | `stages/2-project-factory`  | YAML-driven project creation with Shared VPC and CMEK                  |
| **3-\***              | `stages/3-{workload}-{env}` | Application workloads (GKE, data platform, SecOps, etc.)               |

### Stage Contracts

Each stage produces a `terraform-outputs.json` (or equivalent `outputs.tf`) consumed by the next stage. **Never bypass this contract** by hardcoding values from one stage in another. Always wire outputs through the remote state data source or output files.

Standard remote state reference pattern:

```hcl
data "terraform_remote_state" "bootstrap" {
  backend = "gcs"
  config = {
    bucket = "korvenix-tf-state-bootstrap"
    prefix = "bootstrap"
  }
}

# Use: data.terraform_remote_state.bootstrap.outputs.service_accounts.resman
```

***

## 🛠️ Execution Protocol — Cloud Build Only

> ⛔ **CRITICAL: `terraform apply` MUST NEVER be run locally.** All applies go through Cloud Build. No exceptions. This is Korvenx policy.

The only permitted local operations are:

* `terraform init` (for IDE/linting support)
* `terraform validate` (syntax check)
* `terraform plan` (read-only, for review)

### CI/CD Flow

```
push to main (korvenix/korvenix-gcp-iac)
    └─> Cloud Build trigger fires (cloudbuild/iac-apply.yaml)
        └─> SA impersonation via Workload Identity Federation
            └─> terraform init → validate → plan → apply
```

For PRs, the `cloudbuild/iac-pr.yaml` pipeline runs `plan` only (no apply).

### Cloud Build Trigger Configuration (Terraform)

Use Developer Connect (not legacy GitHub App v1):

```hcl
resource "google_developer_connect_connection" "main" {
  location      = var.region
  connection_id = "korvenix-gcp-iac"
  project       = var.bootstrap_project_id

  github_config {
    github_app = "DEVELOPER_CONNECT"  # MUST be DEVELOPER_CONNECT, not FIREBASE
    authorizer_credential {
      oauth_token_secret_version = "" # populated after manual GitHub OAuth
    }
  }

  depends_on = [google_project_service.main["developerconnect.googleapis.com"]]

  lifecycle {
    ignore_changes = [github_config[0].authorizer_credential]
  }
}

resource "google_cloudbuild_trigger" "iac_apply" {
  developer_connect_event_config {
    git_repository_link = google_developer_connect_git_repository_link.main.id
    push {
      branch = "^main$"
    }
  }
  filename = "cloudbuild/iac-apply.yaml"
  project  = var.bootstrap_project_id
}
```

### Workload Identity Federation

Stage 0 provisions WIF automatically. Cloud Build uses the bootstrap SA (`sa-bootstrap@{project}.iam.gserviceaccount.com`) via WIF — no service account key files.

***

## 📦 GCS Backend & State Layout

All stages use GCS backends. The state bucket for each stage is provisioned by Stage 0:

| Stage       | Bucket                        | Prefix      |
| ----------- | ----------------------------- | ----------- |
| 0-bootstrap | `korvenix-tf-state-bootstrap` | `bootstrap` |
| resman      | `korvenix-tf-state-resman`    | `resman`    |
| factory     | `korvenix-tf-state-factory`   | `factory`   |
| security    | `korvenix-tf-state-security`  | `security`  |

Standard backend block (adjust bucket/prefix per stage):

```hcl
terraform {
  backend "gcs" {
    bucket = "korvenix-tf-state-bootstrap"
    prefix = "bootstrap"
  }
}
```

Enable Object Versioning on all state buckets for corruption recovery.

***

## 🏭 Factory Pattern (Stage 2-project-factory)

Projects are defined as YAML files, not raw Terraform. Each file \= one GCP project.

### YAML Project Definition

```yaml
# stages/2-project-factory/data/projects/korvenix-golinks-prod.yaml
billing_account: 0132B8-03DAEA-CF17C0
parent: folders/XXXXXXXXX       # from resman outputs
services:
  - run.googleapis.com
  - artifactregistry.googleapis.com
  - cloudbuild.googleapis.com
labels:
  env: prod
  product: golinks
iam:
  roles/run.admin:
    - serviceAccount:deployer@korvenix-bootstrap.iam.gserviceaccount.com
```

The factory module (`modules/project-factory`) reads all YAML files and creates projects idempotently. To create a new GCP project, add a YAML file and push to main — Cloud Build does the rest.

### Korvenx Project Naming Convention

| Project ID               | Purpose                            |
| ------------------------ | ---------------------------------- |
| `korvenix-bootstrap`     | Stage 0 automation project         |
| `korvenix-platform-prod` | Main marketing site (korvenix.com) |
| `korvenix-golinks-prod`  | Go Links app — production          |
| `korvenix-golinks-test`  | Go Links app — staging/test        |
| `korvenix-golinks-dev`   | Go Links app — development         |

***

## 🌐 Platform Layer (DNS, LB, Cloud Run)

After FAST stages establish projects and hierarchy, the `platform/` layer provisions application-level resources.

### DNS

* Zones: `korvenix.com`, `korvenix.io`, `korvenix.dev`
* Wildcard A record `*.korvenix.com` → LB IP
* File: `platform/dns.tf`, `stages/dns/main.tf`

### HTTPS Load Balancer

* Global HTTPS LB with serverless NEGs (one per Cloud Run service)
* HTTP → HTTPS redirect
* GCP-managed wildcard cert (`*.korvenix.com` + `korvenix.com`)
* Cloud CDN with `CACHE_ALL_STATIC`
* File: `platform/lb.tf`, `platform/cert.tf`

### Cloud Run Services

* All services: min 0, max 2 instances (board policy; increase requires load test + board approval)
* Authenticate Cloud Run deploys via WIF — no SA keys
* Deploy target: `korvenix-platform-prod` (marketing), `korvenix-golinks-{env}` (app)

***

## 🛡️ IAM Patterns

Follow least-privilege. Never use `google_project_iam_policy` (authoritative, causes lockouts).

```hcl
# Additive — prefer this
resource "google_project_iam_member" "run_invoker" {
  project = var.project_id
  role    = "roles/run.invoker"
  member  = "allUsers"  # only for public endpoints
}

# Authoritative for a single role — use carefully
resource "google_project_iam_binding" "deployer" {
  project = var.project_id
  role    = "roles/run.admin"
  members = [
    "serviceAccount:${var.deploy_sa}",
  ]
}
```

Key org-level roles needed by FAST SAs:

* `sa-bootstrap`: `roles/owner` on bootstrap project + `roles/resourcemanager.organizationAdmin`
* `sa-resman`: `roles/resourcemanager.folderAdmin` + `roles/resourcemanager.projectCreator`
* `sa-factory`: `roles/billing.projectManager` + folder-level `roles/resourcemanager.projectCreator`

***

## 📂 Repository Structure

```
korvenix-gcp-iac/
├── cloudbuild/
│   ├── iac-apply.yaml         # Cloud Build: merge-to-main apply
│   └── iac-pr.yaml            # Cloud Build: PR plan-only
├── platform/
│   ├── dns.tf                 # Cloud DNS zones + records
│   ├── lb.tf                  # Global HTTPS LB + Cloud CDN
│   ├── cert.tf                # GCP-managed wildcard cert
│   └── run.tf                 # Cloud Run service definitions
├── stages/
│   ├── 0-org-setup/           # Fabric FAST Stage 0
│   ├── dns/                   # DNS stage (applied after Stage 0)
│   ├── 1-resman/              # Resource Manager hierarchy
│   ├── 2-project-factory/     # YAML-driven project factory
│   │   └── data/projects/     # One YAML file per GCP project
│   └── 2-networking/          # VPC, subnets, firewall
└── modules/                   # Shared Terraform modules
```

***

## ⚠️ Anti-Patterns — Never Do These

| Anti-pattern                                             | Why                                                                    |
| -------------------------------------------------------- | ---------------------------------------------------------------------- |
| `terraform apply` locally                                | Policy violation. All applies must go through Cloud Build              |
| `google_project_iam_policy`                              | Authoritative — overwrites all IAM, causes lockouts                    |
| Hardcoded project IDs                                    | Use `var.project_id` or `data "google_project"`                        |
| SA key files (`.json`)                                   | Use WIF or metadata server identity                                    |
| `github_app = "FIREBASE"` on Developer Connect           | Causes trigger failures; always use `"DEVELOPER_CONNECT"`              |
| `repository_event_config` for Developer Connect          | Use `developer_connect_event_config` instead                           |
| Skipping `terraform plan` review                         | Always summarize plan, highlight any `destroy` actions before applying |
| Modifying downstream stage values without stage contract | Wire through remote state outputs — no hardcoding across stages        |
| Cloud Run max instances > 2 without load test            | Board policy: requires load test results and board approval            |

***

## 🔍 Debugging & Validation

### Schema Inspection

```bash
terraform providers schema -json | jq '.provider_schemas["registry.terraform.io/hashicorp/google"].resource_schemas | keys[]'
```

### Validate Before Pushing

```bash
terraform init -backend=false   # skip backend for lint
terraform validate
tflint --enable-plugin=google
```

### Check Stage Outputs

```bash
# What did Stage 0 produce?
gsutil cat gs://korvenix-tf-state-bootstrap/bootstrap/default.tfstate | \
  jq '.outputs | to_entries[] | {key: .key, value: .value.value}'
```

### Static Security Analysis

```bash
checkov -d stages/0-org-setup/ --framework terraform
trivy config stages/2-project-factory/
```

***

## 🧪 Testing Strategy

* **PR**: Cloud Build runs `terraform plan` only — review output before merge
* **Post-merge**: Cloud Build runs `terraform apply` — watch for errors in Cloud Build logs
* **Integration test**: Use `terraform test` (v1.6+) to assert labels, network tags, and project structure
* **No staging apply**: All changes go to prod via CI/CD; use PR plan output as the "staging" gate

***

## Required Provider Versions

```hcl
terraform {
  required_version = ">= 1.6.0"
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = ">= 7.20.0"
    }
    google-beta = {
      source  = "hashicorp/google-beta"
      version = ">= 7.20.0"
    }
  }
}
```