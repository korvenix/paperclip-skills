---
name: deploy
description: >
  Release engineer specializing in GCP Cloud Run and GKE deployments, rollbacks,
  and environment promotion (staging -> prod). Uses Terraform for IaC and
  Cloud Build for CI/CD pipelines. Follows Korvenix standards for environment
  segregation and secret management.
license: MIT
metadata:
  author: korvenix
  version: "1.0.0"
  domain: language
  triggers: deploy, release, rollback, Cloud Run, GKE, Cloud Build, Terraform, environment promotion, staging, production
  role: specialist
  scope: implementation
  output-format: text
  related-skills: devops-engineer, be-engineer, fe-engineer, paperclip
---

# Deploy Skill

Expertise in managing the release lifecycle of SaaS applications on Google Cloud Platform.
Ensures reliable, repeatable deployments and promotions across dev, test, and prod environments.

## Tech Stack (Non-Negotiable)

| Layer | Technology |
|-------|-----------|
| Compute | GCP Cloud Run, GKE Autopilot |
| CI/CD | Cloud Build |
| IaC | Terraform |
| Secrets | Secret Manager |
| Auth | Workload Identity Federation |
| Registry | Artifact Registry |

## When to Use This Skill

- Deploying new versions of services to Cloud Run or GKE
- Promoting builds from `staging` (test) to `production`
- Performing emergency rollbacks to a previous stable revision
- Configuring Cloud Build triggers and pipelines
- Managing environment-specific configurations via Terraform and Secret Manager

## Core Workflow

1. **Verify Artifacts** — Ensure the container image is present in Artifact Registry and has passed CI tests.
2. **Infrastructure Update** — Apply Terraform changes for the target environment if necessary.
3. **Deployment** — Update the Cloud Run service or GKE deployment to point to the new image tag.
4. **Validation** — Run health checks and smoke tests in the target environment.
5. **Promotion** — Once verified in `test`, tag the image for `prod` and trigger the production pipeline.
6. **Rollback (if needed)** — Revert the service revision to the previous known-good state.

## Cloud Build Patterns

### Simple Cloud Run Deploy (cloudbuild.yaml)
```yaml
steps:
  # Deploy container image to Cloud Run
  - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
    entrypoint: gcloud
    args:
      - 'run'
      - 'deploy'
      - '${_SERVICE_NAME}'
      - '--image'
      - '${_REGION}-docker.pkg.dev/${PROJECT_ID}/${_REPOSITORY}/${_IMAGE}:${SHORT_SHA}'
      - '--region'
      - '${_REGION}'
      - '--platform'
      - 'managed'
      - '--quiet'

images:
  - '${_REGION}-docker.pkg.dev/${PROJECT_ID}/${_REPOSITORY}/${_IMAGE}:${SHORT_SHA}'
```

### Promotion Trigger
```hcl
resource "google_cloudbuild_trigger" "prod_deploy" {
  name        = "prod-deploy-trigger"
  location    = var.region
  project     = var.project_id

  github {
    owner = var.github_owner
    name  = var.github_repo
    push {
      tag = "^v[0-9]+\\.[0-9]+\\.[0-9]+$"
    }
  }

  filename = "cloudbuild-prod.yaml"
}
```

## Rollback Playbook

1. **Identify Revision** — List revisions: `gcloud run revisions list --service [SERVICE] --region [REGION]`
2. **Execute Rollback** — `gcloud run services update-traffic [SERVICE] --to-revisions [REVISION]=100 --region [REGION]`
3. **Verify** — Check logs and health checks for the reverted revision.

## Constraints

### MUST DO
- Use Terraform for all infrastructure changes accompanying a deploy.
- Store environment-specific configurations in Secret Manager or Terraform variables.
- Tag all images with `SHORT_SHA` and semantic version tags for production.
- Require successful `test` environment validation before `prod` promotion.
- Maintain a clear audit trail of all deployments.

### MUST NOT DO
- Perform manual deployments via `gcloud` from a local machine (use Cloud Build).
- Deploy images that haven't passed automated security scans.
- Use static service account keys (use Workload Identity Federation).
- Hardcode environment variables in `cloudbuild.yaml`.

## Reference Guide

| Topic | Load When |
|-------|-----------|
| Terraform Cloud Run | Scaling, VPC connector, custom domains |
| GKE Autopilot Deploy | Helm charts, K8s manifests, workload identity |
| Cloud Build Advanced | Parallel steps, custom workers, secret volumes |
| Artifact Registry | Cleanup policies, remote repositories |
