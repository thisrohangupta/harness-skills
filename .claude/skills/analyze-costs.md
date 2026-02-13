---
name: analyze-costs
description: Analyze cloud costs and optimization opportunities using Harness CCM MCP tools. Generate cost reports, identify savings, and create action items.
triggers:
  - analyze costs
  - cloud costs
  - cost optimization
  - cost savings
  - ccm report
  - cloud spend
  - reduce costs
  - cost recommendations
---

# Analyze Costs Skill

Analyze cloud costs and identify optimization opportunities using Harness Cloud Cost Management (CCM) MCP tools.

## Overview

This skill helps teams:
- Understand cloud spending patterns
- Identify cost optimization opportunities
- Get actionable recommendations
- Track cost anomalies
- Create tickets for cost-saving initiatives

## Required MCP Tools

This skill requires the Harness MCP Server with the `ccm` toolset enabled:
- `get_ccm_overview` - Cost summary
- `get_ccm_metadata` - CCM configuration
- `list_ccm_perspectives` - Custom cost views
- `ccm_perspective_grid` - Detailed cost breakdown
- `ccm_perspective_time_series` - Cost trends
- `list_ccm_recommendations` - Optimization suggestions
- `get_ccm_anomalies_summary` - Unusual spending
- `list_ccm_anomalies` - Anomaly details
- `create_jira_ticket_for_ccm_recommendation` - Action item creation

## Workflow

### Step 1: Get Cost Overview

Start with high-level cost summary:

```
Use MCP tool: get_ccm_overview
Parameters:
  - time_period: "LAST_30_DAYS" (or LAST_7_DAYS, LAST_MONTH, etc.)
```

This provides:
- Total cloud spend
- Spend by cloud provider (AWS, GCP, Azure)
- Cost trend (increasing/decreasing)
- Comparison to previous period

### Step 2: Get Cost Metadata

Understand what's being tracked:

```
Use MCP tool: get_ccm_metadata
```

Returns:
- Connected cloud accounts
- Enabled features
- Data freshness
- Cost categories configured

### Step 3: List Perspectives

Get available cost views:

```
Use MCP tool: list_ccm_perspectives_detail
```

Common perspectives:
- By Service/Application
- By Environment (prod, staging, dev)
- By Team/Cost Center
- By Cloud Provider
- By Region

### Step 4: Analyze Specific Perspective

Drill into cost details:

```
Use MCP tool: ccm_perspective_grid
Parameters:
  - perspective_id: <perspective identifier>
  - time_period: "LAST_30_DAYS"
  - group_by: ["service", "environment"] (optional)
```

For trend analysis:

```
Use MCP tool: ccm_perspective_time_series
Parameters:
  - perspective_id: <perspective identifier>
  - time_period: "LAST_30_DAYS"
  - granularity: "DAILY"
```

### Step 5: Get Recommendations

Fetch optimization opportunities:

```
Use MCP tool: list_ccm_recommendations
Parameters:
  - resource_type: "all" (or ec2, azure_vm, ecs, workload, nodepool)
  - page_size: 20
```

For specific resource types:

```
Use MCP tool: list_ccm_recommendations_by_resource_type
Parameters:
  - resource_type: "ec2" (or azure_vm, ecs_service, workload, nodepool)
```

Get recommendation statistics:

```
Use MCP tool: get_ccm_recommendations_stats
```

### Step 6: Get Recommendation Details

For specific recommendations:

```
Use MCP tool: get_ec2_recommendation_detail
Parameters:
  - recommendation_id: <id>

# Or for other types:
# get_azure_vm_recommendation_detail
# get_ecs_service_recommendation_detail
# get_node_pool_recommendation_detail
# get_workload_recommendation_detail
```

### Step 7: Check for Anomalies

Identify unusual spending:

```
Use MCP tool: get_ccm_anomalies_summary
Parameters:
  - time_period: "LAST_30_DAYS"
```

Get anomaly details:

```
Use MCP tool: list_ccm_anomalies
Parameters:
  - time_period: "LAST_30_DAYS"
  - page_size: 10
```

### Step 8: Create Action Items

For actionable recommendations, create tickets:

```
Use MCP tool: create_jira_ticket_for_ccm_recommendation
Parameters:
  - recommendation_id: <recommendation id>
  - jira_connector_id: <jira connector>
  - project_key: <jira project>
  - issue_type: "Task"
  - summary: <ticket title>
  - description: <recommendation details>
```

Or for ServiceNow:

```
Use MCP tool: create_service_now_ticket_for_ccm_recommendation
Parameters:
  - recommendation_id: <recommendation id>
  - servicenow_connector_id: <connector>
  - table_name: "incident" (or change_request)
```

## Response Format

### Cost Overview Report

```markdown
## Cloud Cost Analysis Report

**Period:** Last 30 Days
**Generated:** <date>

### Summary

| Metric | Value | vs Previous Period |
|--------|-------|-------------------|
| Total Spend | $45,230 | +5.2% |
| AWS | $32,100 | +3.1% |
| GCP | $10,430 | +12.4% |
| Azure | $2,700 | -2.0% |

### Trend

[Brief description of spending trend]

📈 **Cost is trending UP** - primarily driven by GCP compute increases.

### Top Cost Drivers

| Service | Cost | % of Total | Change |
|---------|------|------------|--------|
| EC2 | $18,500 | 40.9% | +2.3% |
| RDS | $8,200 | 18.1% | +8.5% |
| S3 | $4,100 | 9.1% | -1.2% |
| GKE | $6,800 | 15.0% | +15.2% |
| CloudSQL | $3,200 | 7.1% | +10.1% |
```

### Recommendations Report

```markdown
## Cost Optimization Recommendations

**Potential Monthly Savings:** $8,450

### High Priority (>$1,000/month savings)

#### 1. Rightsize EC2 Instances
**Savings:** $3,200/month
**Resource:** prod-api-server (i3.2xlarge → i3.xlarge)
**Confidence:** 95%
**Utilization:** CPU avg 23%, Memory avg 45%

**Action:** Downgrade instance type during next maintenance window

---

#### 2. Convert to Reserved Instances
**Savings:** $2,100/month
**Resources:** 5 EC2 instances running 24/7
**Commitment:** 1-year reserved

**Action:** Purchase reserved capacity for stable workloads

---

### Medium Priority ($100-$1,000/month)

#### 3. Delete Unused EBS Volumes
**Savings:** $450/month
**Resources:** 12 unattached volumes (2.4 TB)

**Action:** Review and delete unused volumes

---

#### 4. Optimize S3 Storage Classes
**Savings:** $320/month
**Resources:** 500GB infrequently accessed data

**Action:** Move to S3 Infrequent Access tier

---

### Quick Wins (<$100/month but easy)

- Delete 3 unused Elastic IPs ($10/month)
- Remove old snapshots >90 days ($85/month)
- Stop dev instances on weekends ($120/month)
```

### Anomaly Report

```markdown
## Cost Anomalies Detected

**Period:** Last 30 Days
**Anomalies Found:** 3

### Critical Anomalies

#### 1. GKE Cluster Spike
**Date:** 2024-01-15
**Service:** GKE
**Expected:** $180/day
**Actual:** $520/day
**Excess:** $340 (+189%)

**Possible Causes:**
- Auto-scaling event
- New deployment with higher resource requests
- Runaway container

**Status:** Investigating

---

#### 2. Data Transfer Surge
**Date:** 2024-01-18
**Service:** AWS Data Transfer
**Expected:** $50/day
**Actual:** $180/day
**Excess:** $130 (+260%)

**Possible Causes:**
- Cross-region replication
- Large data export
- Misconfigured CDN

**Status:** Resolved - one-time data migration
```

## Common Analysis Scenarios

### 1. Monthly Cost Review

```
/analyze-costs

Give me a summary of cloud costs for the last month
compared to the previous month
```

### 2. Environment Breakdown

```
/analyze-costs

Break down costs by environment (prod, staging, dev)
for the last 30 days
```

### 3. Team Attribution

```
/analyze-costs

Show costs by team/cost center for Q4
```

### 4. Optimization Opportunities

```
/analyze-costs

What are the top 10 cost optimization recommendations
sorted by potential savings?
```

### 5. Anomaly Investigation

```
/analyze-costs

Are there any unusual spending patterns in the last week?
```

### 6. Specific Service Analysis

```
/analyze-costs

Analyze our Kubernetes costs and suggest optimizations
```

## Cost Categories

Harness CCM organizes costs into categories:

### By Resource Type
- **Compute:** EC2, VMs, GKE/EKS nodes
- **Database:** RDS, CloudSQL, CosmosDB
- **Storage:** S3, GCS, Blob Storage
- **Network:** Data transfer, Load balancers, VPN
- **Containers:** ECS, EKS, GKE, AKS

### By Optimization Type
- **Rightsizing:** Oversized instances
- **Reserved/Committed:** On-demand to reserved
- **Spot/Preemptible:** Use spot instances
- **Cleanup:** Unused/orphaned resources
- **Storage Tiering:** Move to cheaper tiers

## Creating Action Items

When recommendations warrant action:

### For Jira

```
Would you like me to create a Jira ticket for this recommendation?

**Recommendation:** Rightsize prod-api-server
**Savings:** $3,200/month
**Effort:** Low (instance type change)

I can create a ticket in your project with:
- Detailed recommendation
- Step-by-step implementation guide
- Expected savings
- Risk assessment
```

### For ServiceNow

```
Should I create a ServiceNow change request for this optimization?

This would include:
- Change description
- Impact assessment
- Implementation steps
- Rollback plan
```

## Budget Tracking

If budgets are configured:

```
Use MCP tool: ccm_perspective_summary_with_budget
Parameters:
  - perspective_id: <perspective>
```

Response includes:
- Budget amount
- Current spend
- Forecasted spend
- Budget utilization %
- Alerts if over budget

## Example Usage

### Quick Overview

```
/analyze-costs

How much are we spending on cloud this month?
```

### Deep Dive

```
/analyze-costs

Analyze our AWS costs by service and region
Show me the top 5 cost drivers and any recommendations
```

### Find Savings

```
/analyze-costs

Find me $5,000 in monthly savings from our cloud infrastructure
```

### Investigate Spike

```
/analyze-costs

Our cloud bill jumped 30% last week - what happened?
```

## Instructions

When analyzing costs:

1. **Start with overview:**
   - Get high-level cost summary
   - Understand the time period requested
   - Note any specific focus areas (service, team, environment)

2. **Gather detailed data:**
   - Use perspectives for organized views
   - Get time series for trends
   - Check for anomalies

3. **Identify opportunities:**
   - Fetch recommendations
   - Prioritize by savings potential
   - Assess implementation effort

4. **Present findings:**
   - Lead with key numbers
   - Highlight significant changes
   - Provide actionable recommendations

5. **Offer next steps:**
   - Create tickets for action items
   - Suggest monitoring improvements
   - Recommend governance policies
