---
name: gcp-observability
description: >
  DevOps engineer specializing in GCP Observability (Monitoring, Logging, Error Reporting).
  Creates dashboards, alert policies, notification channels, and SLOs via Terraform.
  Ensures all services have structured logging and core health metrics.
license: MIT
metadata:
  author: korvenix
  version: "1.0.0"
  domain: devops
  triggers: observability, monitoring, logging, dashboard, alert, SLO, health check, uptime, error reporting
  role: specialist
  scope: implementation
  output-format: text
  related-skills: devops-engineer, deploy
---

# GCP Observability Skill

Expertise in building and managing observability stacks on Google Cloud Platform using Terraform.
Ensures high availability and reliability through proactive monitoring and automated alerting.

## Tech Stack (Non-Negotiable)

| Layer | Technology |
|-------|-----------|
| Monitoring | Cloud Monitoring (Metrics, Dashboards) |
| Logging | Cloud Logging (Log Explorer, Log-based Metrics) |
| Alerts | Monitoring Alert Policies & Notification Channels |
| Reliability | Service Level Objectives (SLOs) & SLIs |
| Tracing | Cloud Trace |
| Errors | Error Reporting |
| IaC | Terraform |

## When to Use This Skill

- Creating or updating Cloud Monitoring dashboards
- Configuring alert policies for service health (latency, error rate, uptime)
- Setting up notification channels (email, Slack, PagerDuty, Webhooks)
- Defining Service Level Indicators (SLIs) and Objectives (SLOs)
- Creating log-based metrics for custom application events
- Integrating Error Reporting for automated crash/exception tracking

## Core Workflow

1. **Define SLIs/SLOs** — Identify the critical user journeys and define what "healthy" looks like.
2. **Setup Notification Channels** — Ensure the right people are notified when alerts fire.
3. **Configure Alert Policies** — Create policies based on SLIs (e.g., Error Rate > 1% over 5m).
4. **Build Dashboards** — Visualize core metrics and SLO status for easy consumption.
5. **Log-Based Metrics** — Extract metrics from structured logs for application-specific insights.
6. **Iteration** — Refine alerts to reduce noise and improve signal-to-noise ratio.

## Terraform Patterns

### Alert Policy (Latency)
```hcl
resource "google_monitoring_alert_policy" "high_latency" {
  display_name = "High Latency - ${var.service_name}"
  combiner     = "OR"
  conditions {
    display_name = "Request Latency > 500ms"
    condition_threshold {
      filter          = "resource.type = \"cloud_run_revision\" AND metric.type = \"run.googleapis.com/request_latencies\""
      duration        = "60s"
      comparison      = "COMPARISON_GT"
      threshold_value = 500
      aggregations {
        alignment_period   = "60s"
        per_series_aligner = "ALIGN_PERCENTILE_99"
      }
    }
  }
  notification_channels = [var.notification_channel_id]
}
```

### Dashboard (Cloud Run)
```hcl
resource "google_monitoring_dashboard" "service_dashboard" {
  dashboard_json = <<EOF
{
  "displayName": "Service Dashboard: ${var.service_name}",
  "gridLayout": {
    "columns": "2",
    "widgets": [
      {
        "title": "Request Count",
        "xyChart": {
          "dataSets": [{
            "plotType": "STACKED_BAR",
            "targetAxis": "Y1",
            "timeSeriesQuery": {
              "timeSeriesFilter": {
                "filter": "resource.type=\"cloud_run_revision\" AND metric.type=\"run.googleapis.com/request_count\""
              }
            }
          }]
        }
      }
    ]
  }
}
EOF
}
```

### SLO (Availability)
```hcl
resource "google_monitoring_slo" "availability_slo" {
  service      = var.monitoring_service_id
  slo_id       = "availability-slo"
  display_name = "99.9% Availability"

  goal                = 0.999
  rolling_period_days = 30

  request_based_sli {
    good_total_ratio {
      good_service_filter = "resource.type=\"cloud_run_revision\" AND metric.type=\"run.googleapis.com/request_count\" AND metric.labels.response_code_class=\"2xx\""
      total_service_filter = "resource.type=\"cloud_run_revision\" AND metric.type=\"run.googleapis.com/request_count\""
    }
  }
}
```

## Constraints

### MUST DO
- Use Terraform for all observability resources.
- Use structured logging (JSON) in all applications.
- Define SLOs for all production-critical services.
- Test alert policies during deployment to ensure they fire as expected.
- Tag observability resources with `env`, `product`, and `team`.

### MUST NOT DO
- Create alerts without a linked notification channel.
- Hardcode project IDs or service names (use variables).
- Use manual dashboards in the GCP Console (use IaC).
- Alert on transient spikes (use appropriate duration/averaging).

## Reference Guide

| Topic | Load When |
|-------|-----------|
| MQL Reference | Complex metric queries, cross-project monitoring |
| Log-based Metrics | Custom app events, legacy log parsing |
| Uptime Checks | External availability monitoring, multi-region checks |
| Notification Channels | PagerDuty, Slack integration, Webhook schemas |
