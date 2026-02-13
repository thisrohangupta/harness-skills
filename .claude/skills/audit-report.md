---
name: audit-report
description: Generate audit reports and compliance trails using Harness MCP tools. Track user actions, resource changes, and access patterns.
triggers:
  - audit report
  - audit trail
  - audit log
  - compliance audit
  - user activity
  - change log
  - access audit
---

# Audit Report Skill

Generate audit reports and compliance trails using Harness MCP tools.

## Overview

This skill helps security, compliance, and platform teams:
- Track user actions and changes
- Generate compliance reports
- Investigate security incidents
- Monitor access patterns
- Audit resource modifications

## Required MCP Tools

This skill requires the Harness MCP Server with the `audit` toolset enabled:

### Audit Tools
- `list_user_audits` - List audit events for users

### Supporting Tools (from other toolsets)
- `list_secrets` - List secrets (for access auditing)
- `list_connectors` - List connectors
- `list_pipelines` - List pipelines
- `list_executions` - List pipeline executions

## Workflow

### Step 1: List User Audits

Get audit events:

```
Use MCP tool: list_user_audits
Parameters:
  - start_time: <start timestamp>
  - end_time: <end timestamp>
  - user_id: <optional user filter>
  - action: <optional action filter>
  - resource_type: <optional resource filter>
  - org_id: <organization>
  - project_id: <project>
  - page_size: 100
```

### Step 2: Filter by Action Type

Common action types:
- `CREATE` - Resource creation
- `UPDATE` - Resource modification
- `DELETE` - Resource deletion
- `LOGIN` - User login
- `LOGOUT` - User logout
- `ACCESS` - Resource access
- `EXECUTE` - Pipeline execution

### Step 3: Filter by Resource Type

Common resource types:
- `PIPELINE` - Pipeline changes
- `SECRET` - Secret access/changes
- `CONNECTOR` - Connector modifications
- `SERVICE` - Service definitions
- `ENVIRONMENT` - Environment changes
- `USER` - User management
- `ROLE` - Role assignments
- `USER_GROUP` - Group changes

### Step 4: Correlate Events

For investigation, correlate:
- User activity across resources
- Changes leading to incidents
- Access patterns over time

### Step 5: Generate Report

Format findings based on use case:
- Compliance audit
- Security investigation
- Change management

## Response Format

### General Audit Report

```markdown
## Audit Report

**Period:** January 1-31, 2024
**Scope:** Production Project
**Generated:** February 1, 2024

### Summary

| Metric | Count |
|--------|-------|
| Total Events | 1,245 |
| Unique Users | 28 |
| Resources Modified | 156 |
| Pipelines Executed | 892 |

### Activity by Category

| Category | Events | % of Total |
|----------|--------|------------|
| Pipeline Executions | 892 | 72% |
| Resource Updates | 198 | 16% |
| Resource Creates | 87 | 7% |
| Resource Deletes | 12 | 1% |
| Access Events | 56 | 4% |

### Top Users by Activity

| User | Events | Primary Activity |
|------|--------|------------------|
| deploy-bot | 456 | Pipeline Execution |
| john.doe | 234 | Pipeline Management |
| jane.smith | 189 | Resource Updates |
| ops-team | 145 | Deployments |
| security-scan | 98 | Secret Access |

### Critical Changes

| Date | User | Action | Resource | Details |
|------|------|--------|----------|---------|
| Jan 28 | admin | DELETE | Secret | prod-db-password |
| Jan 25 | john.doe | UPDATE | Pipeline | Added prod stage |
| Jan 20 | jane.smith | CREATE | Connector | new-aws-connector |
| Jan 15 | ops-team | UPDATE | Environment | Modified prod vars |
```

### Security Audit Report

```markdown
## Security Audit Report

**Period:** Last 30 Days
**Focus:** Access Control & Secrets
**Classification:** Confidential

### Executive Summary

This report covers security-relevant audit events including
secret access, permission changes, and authentication events.

### Authentication Events

| Event Type | Count | Status |
|------------|-------|--------|
| Successful Logins | 342 | ✅ Normal |
| Failed Logins | 12 | ⚠️ Review |
| API Key Usage | 1,205 | ✅ Normal |
| Service Account Access | 456 | ✅ Normal |

### Failed Login Attempts

| User | Attempts | Last Attempt | Source IP |
|------|----------|--------------|-----------|
| unknown@company.com | 5 | Jan 28 10:30 | 203.0.113.42 |
| john.doe | 3 | Jan 25 14:22 | 10.0.0.15 |
| test.user | 4 | Jan 20 09:15 | 192.168.1.100 |

**Recommendation:** Review failed login patterns for
potential brute force attempts.

### Secret Access Audit

| Secret | Access Count | Users | Last Access |
|--------|--------------|-------|-------------|
| prod-db-password | 245 | 3 | 2h ago |
| api-keys | 189 | 5 | 1h ago |
| ssl-certificates | 45 | 2 | 1d ago |
| oauth-secrets | 78 | 4 | 3h ago |

### Privileged Actions

| Date | User | Action | Details |
|------|------|--------|---------|
| Jan 28 | admin | Role Change | Added prod-admin to john.doe |
| Jan 25 | admin | Secret Delete | Removed deprecated-key |
| Jan 22 | security | Connector Update | Rotated AWS credentials |
| Jan 18 | admin | User Create | Added new-contractor |

### Access Pattern Anomalies

⚠️ **Unusual Activity Detected:**

1. **User: john.doe**
   - 50% increase in secret access
   - New IP address: 203.0.113.50
   - Outside normal working hours

2. **Service: deploy-bot**
   - Accessed 3 new secrets
   - New pipeline triggers

**Recommendation:** Verify these changes are authorized.

### Compliance Checklist

| Control | Status | Evidence |
|---------|--------|----------|
| MFA Enabled | ✅ Pass | All admin users |
| Secret Rotation | ✅ Pass | 90-day rotation |
| Access Reviews | ✅ Pass | Quarterly review |
| Audit Logging | ✅ Pass | All events captured |
| Least Privilege | ⚠️ Review | 3 users with broad access |
```

### Change Management Report

```markdown
## Change Management Report

**Period:** Last 7 Days
**Focus:** Production Pipeline Changes
**Prepared for:** CAB Review

### Summary

| Change Type | Count | Approved | Emergency |
|-------------|-------|----------|-----------|
| Pipeline Updates | 12 | 10 | 2 |
| Infrastructure Changes | 5 | 5 | 0 |
| Configuration Updates | 8 | 7 | 1 |
| Secret Rotations | 3 | 3 | 0 |

### Detailed Change Log

#### Pipeline Changes

| Date | Pipeline | Change | User | Approved By |
|------|----------|--------|------|-------------|
| Jan 30 | deploy-prod | Added canary stage | john.doe | jane.smith |
| Jan 29 | ci-main | Updated Node version | dev-team | auto |
| Jan 28 | security-scan | New SAST step | security | john.doe |
| Jan 27 | deploy-prod | Increased replicas | ops-team | jane.smith |

#### Infrastructure Changes

| Date | Resource | Change | User | Impact |
|------|----------|--------|------|--------|
| Jan 29 | k8s-connector | Updated credentials | ops-team | None |
| Jan 28 | aws-connector | Added region | cloud-team | None |
| Jan 26 | docker-registry | New ECR repo | dev-team | None |

#### Emergency Changes

| Date | Change | Reason | User | Post-Approval |
|------|--------|--------|------|---------------|
| Jan 30 | Rollback deploy | Production incident | ops-team | Pending |
| Jan 27 | Hotfix pipeline | Critical bug | john.doe | Approved |

### Pending Approvals

| Change | Requested By | Date | Status |
|--------|--------------|------|--------|
| New prod environment | dev-team | Jan 30 | Pending CAB |
| Connector permissions | ops-team | Jan 29 | Pending security |

### Risk Assessment

| Change | Risk Level | Mitigation |
|--------|------------|------------|
| Canary deployment | Low | Gradual rollout |
| New SAST step | Low | Non-blocking |
| Increased replicas | Low | Auto-scaling backup |
```

### User Activity Report

```markdown
## User Activity Report

**User:** john.doe
**Period:** January 2024
**Role:** Platform Engineer

### Activity Summary

| Metric | Value |
|--------|-------|
| Login Days | 22 |
| Total Actions | 234 |
| Pipelines Modified | 8 |
| Deployments Triggered | 45 |
| Secrets Accessed | 12 |

### Daily Activity Pattern

```
Activity by Hour (UTC):
00-06: ░░░░░░
06-12: ████████████ Peak
12-18: ██████████ High
18-24: ████ Moderate
```

### Actions by Type

| Action | Count | % |
|--------|-------|---|
| Execute Pipeline | 45 | 19% |
| Update Pipeline | 32 | 14% |
| View Resources | 98 | 42% |
| Create Resources | 15 | 6% |
| Access Secrets | 12 | 5% |
| Other | 32 | 14% |

### Resources Accessed

| Resource Type | Count | Actions |
|---------------|-------|---------|
| Pipelines | 8 | View, Update, Execute |
| Secrets | 12 | View |
| Connectors | 5 | View |
| Environments | 3 | View, Update |
| Services | 4 | View |

### Session Information

| Date | Duration | IP Address | Location |
|------|----------|------------|----------|
| Jan 30 | 8h 15m | 10.0.0.15 | Office |
| Jan 29 | 7h 45m | 10.0.0.15 | Office |
| Jan 28 | 6h 30m | 192.168.1.50 | VPN |
| Jan 27 | 8h 00m | 10.0.0.15 | Office |

### Notable Actions

| Date | Time | Action | Resource | Notes |
|------|------|--------|----------|-------|
| Jan 30 | 14:22 | Updated | deploy-prod | Added canary stage |
| Jan 28 | 10:15 | Deleted | old-pipeline | Cleanup |
| Jan 25 | 16:45 | Created | new-service | New microservice |
```

## Common Scenarios

### 1. General Audit

```
/audit-report

Generate an audit report for the last 30 days
```

### 2. Security Investigation

```
/audit-report

Show me all secret access events from last week
```

### 3. User Activity

```
/audit-report

What has john.doe been doing in the last 7 days?
```

### 4. Change Tracking

```
/audit-report

Show all production pipeline changes this month
```

### 5. Compliance Audit

```
/audit-report

Generate a SOC2-style audit report for Q4
```

### 6. Incident Investigation

```
/audit-report

What changes were made before the production incident
on January 28th?
```

## Compliance Frameworks

### SOC 2

| Control | Audit Events |
|---------|--------------|
| CC6.1 - Logical Access | Login events, permission changes |
| CC6.2 - Access Authorization | Role assignments, approvals |
| CC6.3 - Access Removal | User deletions, permission revocations |
| CC7.1 - Change Management | Pipeline/resource changes |
| CC7.2 - System Monitoring | Execution logs, alerts |

### GDPR

| Requirement | Audit Events |
|-------------|--------------|
| Access Logging | All data access events |
| Consent Tracking | Configuration changes |
| Data Processing | Pipeline executions |
| Right to Audit | Complete audit trail |

### HIPAA

| Control | Audit Events |
|---------|--------------|
| Access Controls | Authentication events |
| Audit Controls | All system events |
| Integrity Controls | Resource modifications |
| Transmission Security | Connector access |

## Example Usage

### Quick Overview

```
/audit-report

What happened in the platform today?
```

### User Investigation

```
/audit-report

Audit all activity by the ops-team service account
```

### Change Audit

```
/audit-report

List all resource deletions in the last month
```

### Compliance Report

```
/audit-report

Generate a compliance audit report for the security team
```

## Instructions

When generating audit reports:

1. **Understand the purpose:**
   - General audit vs investigation
   - Compliance framework if applicable
   - Time period and scope

2. **Gather relevant events:**
   - Filter by time, user, action, resource
   - Include related events for context
   - Get sufficient detail

3. **Analyze patterns:**
   - Identify anomalies
   - Note trends
   - Correlate related events

4. **Present clearly:**
   - Lead with summary
   - Highlight critical items
   - Include recommendations

5. **Support compliance:**
   - Map to control frameworks
   - Note any gaps
   - Suggest improvements
