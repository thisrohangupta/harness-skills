---
name: scorecard-review
description: Review IDP scorecards and service maturity using Harness Internal Developer Portal MCP tools. Check service compliance and improvement areas.
triggers:
  - scorecard review
  - service scorecard
  - idp scorecard
  - maturity check
  - compliance check
  - service health
  - developer portal
---

# Scorecard Review Skill

Review IDP scorecards and service maturity using Harness Internal Developer Portal MCP tools.

## Overview

This skill helps platform and engineering teams:
- View service scorecards and scores
- Check compliance with engineering standards
- Identify improvement areas
- Track maturity progress over time
- Review check details and remediation

## Required MCP Tools

This skill requires the Harness MCP Server with the `idp` toolset enabled:

### Scorecard Tools
- `get_scorecard` - Get scorecard definition
- `list_scorecards` - List all scorecards
- `get_score_summary` - Get score summary for entity
- `get_scores` - Get detailed scores
- `get_scorecard_stats` - Get scorecard statistics

### Check Tools
- `get_scorecard_check` - Get check definition
- `list_scorecard_checks` - List all checks
- `get_scorecard_check_stats` - Get check statistics

### Entity Tools
- `get_entity` - Get catalog entity details
- `list_entities` - List catalog entities

### Other Tools
- `execute_workflow` - Execute self-service workflow
- `search_tech_docs` - Search technical documentation

## Workflow

### Step 1: List Scorecards

See available scorecards:

```
Use MCP tool: list_scorecards
Parameters:
  - org_id: <organization>
  - project_id: <project>
```

Common scorecards:
- Production Readiness
- Security Compliance
- Documentation Quality
- Operational Excellence
- Developer Experience

### Step 2: Get Scorecard Details

View scorecard definition:

```
Use MCP tool: get_scorecard
Parameters:
  - scorecard_id: <scorecard identifier>
  - org_id: <organization>
  - project_id: <project>
```

Returns:
- Name and description
- Checks included
- Passing threshold
- Weight distribution

### Step 3: List Entities

Find services/components to review:

```
Use MCP tool: list_entities
Parameters:
  - kind: "Component" (or Service, API, etc.)
  - org_id: <organization>
  - project_id: <project>
```

### Step 4: Get Score Summary

View scores for an entity:

```
Use MCP tool: get_score_summary
Parameters:
  - entity_ref: <entity reference>
  - org_id: <organization>
  - project_id: <project>
```

### Step 5: Get Detailed Scores

Get check-by-check breakdown:

```
Use MCP tool: get_scores
Parameters:
  - entity_ref: <entity reference>
  - scorecard_id: <scorecard identifier>
  - org_id: <organization>
  - project_id: <project>
```

### Step 6: Get Check Details

Understand a specific check:

```
Use MCP tool: get_scorecard_check
Parameters:
  - check_id: <check identifier>
  - org_id: <organization>
  - project_id: <project>
```

### Step 7: Get Statistics

View overall compliance stats:

```
Use MCP tool: get_scorecard_stats
Parameters:
  - scorecard_id: <scorecard identifier>
  - org_id: <organization>
  - project_id: <project>
```

## Response Format

### Scorecard Overview

```markdown
## Scorecard Overview

**Organization:** my-org
**Total Scorecards:** 4
**Total Services:** 45

### Scorecards Summary

| Scorecard | Passing | Failing | Avg Score |
|-----------|---------|---------|-----------|
| Production Readiness | 38 | 7 | 82% |
| Security Compliance | 42 | 3 | 91% |
| Documentation | 30 | 15 | 68% |
| Operational Excellence | 35 | 10 | 76% |

### Services Needing Attention

| Service | Lowest Score | Issue |
|---------|--------------|-------|
| legacy-api | 45% | Production Readiness |
| admin-tool | 52% | Documentation |
| batch-processor | 58% | Operational Excellence |

### Top Performing Services

| Service | Avg Score | Badges |
|---------|-----------|--------|
| api-gateway | 98% | 🏆 Production Ready |
| user-service | 96% | 🛡️ Security Certified |
| payment-service | 94% | 📚 Well Documented |
```

### Service Scorecard Report

```markdown
## Scorecard Report: api-gateway

**Entity:** component:api-gateway
**Owner:** platform-team
**Lifecycle:** production

### Overall Scores

| Scorecard | Score | Status | Trend |
|-----------|-------|--------|-------|
| Production Readiness | 95% | ✅ Pass | ⬆️ +5% |
| Security Compliance | 100% | ✅ Pass | → 0% |
| Documentation | 88% | ✅ Pass | ⬆️ +10% |
| Operational Excellence | 92% | ✅ Pass | ⬆️ +3% |

### Production Readiness Breakdown

**Score: 95%** (19/20 checks passing)

#### Passing Checks ✅

| Check | Weight | Status |
|-------|--------|--------|
| Has CI/CD Pipeline | 10% | ✅ |
| Automated Testing | 10% | ✅ |
| Monitoring Configured | 10% | ✅ |
| Alerting Setup | 10% | ✅ |
| Runbook Exists | 5% | ✅ |
| On-Call Rotation | 5% | ✅ |
| SLO Defined | 10% | ✅ |
| Disaster Recovery | 10% | ✅ |
| Security Scan Passing | 10% | ✅ |
| Load Testing Done | 5% | ✅ |
| Dependency Up-to-Date | 5% | ✅ |
| ... | | |

#### Failing Checks ❌

| Check | Weight | Issue | Remediation |
|-------|--------|-------|-------------|
| Chaos Testing | 5% | No chaos experiments | Create chaos experiment for pod failure |

### Improvement Recommendations

1. **Add Chaos Testing** (+5%)
   - Create pod-delete experiment
   - Run weekly resilience tests
   - Document results

### Badges Earned 🏆

- ✅ Production Ready
- ✅ Security Certified
- ✅ Well Documented
- ⏳ Chaos Champion (1 check remaining)
```

### Check Details Report

```markdown
## Check Details: has-runbook

**Check ID:** has-runbook
**Category:** Operational Excellence
**Weight:** 5%

### Description

Verifies that the service has an operational runbook documented
in the expected location.

### Evaluation Criteria

| Criteria | Requirement |
|----------|-------------|
| Type | Metadata Check |
| Field | `metadata.annotations.runbook-url` |
| Condition | Must be non-empty URL |

### Current Status Across Services

| Status | Count | Percentage |
|--------|-------|------------|
| Passing | 38 | 84% |
| Failing | 7 | 16% |

### Failing Services

| Service | Owner | Issue |
|---------|-------|-------|
| legacy-api | legacy-team | Missing annotation |
| batch-processor | data-team | Invalid URL |
| admin-tool | platform-team | Missing annotation |

### Remediation Steps

1. Create runbook in your documentation system
2. Add annotation to catalog-info.yaml:
   ```yaml
   metadata:
     annotations:
       runbook-url: https://docs.example.com/runbooks/my-service
   ```
3. Commit and push changes
4. Wait for catalog refresh (up to 5 minutes)

### Related Resources

- [Runbook Template](https://docs.example.com/templates/runbook)
- [Catalog Annotation Guide](https://docs.example.com/catalog/annotations)
```

### Compliance Dashboard

```markdown
## Compliance Dashboard

**Period:** Last 30 Days
**Services Evaluated:** 45

### Overall Compliance

```
Production Readiness  ████████░░ 84%
Security Compliance   █████████░ 93%
Documentation         ██████░░░░ 67%
Operational Excellence ███████░░░ 78%
```

### Trend Analysis

| Scorecard | 30d Ago | Today | Change |
|-----------|---------|-------|--------|
| Production Readiness | 79% | 84% | ⬆️ +5% |
| Security Compliance | 90% | 93% | ⬆️ +3% |
| Documentation | 60% | 67% | ⬆️ +7% |
| Operational Excellence | 75% | 78% | ⬆️ +3% |

### Most Improved Services

| Service | Score Change | Key Improvements |
|---------|--------------|------------------|
| checkout-service | +25% | Added monitoring, runbook |
| notification-service | +20% | Security scan, docs |
| inventory-service | +18% | CI/CD, testing |

### Most Common Failures

| Check | Failure Rate | Impact |
|-------|--------------|--------|
| Has Tech Docs | 35% | Documentation |
| Chaos Testing | 30% | Prod Readiness |
| API Documentation | 28% | Documentation |
| SLO Defined | 22% | Prod Readiness |

### Recommendations

1. **Documentation Initiative**
   - 16 services missing tech docs
   - Schedule documentation sprint

2. **Chaos Engineering Rollout**
   - 14 services without chaos tests
   - Provide templates and training

3. **SLO Workshop**
   - 10 services need SLO definition
   - Conduct SLO best practices session
```

## Common Scenarios

### 1. Service Status Check

```
/scorecard-review

How is the api-gateway doing on scorecards?
```

### 2. Find Failing Services

```
/scorecard-review

Which services are failing production readiness?
```

### 3. Check Compliance

```
/scorecard-review

Show me overall compliance with security scorecard
```

### 4. Get Remediation Steps

```
/scorecard-review

The user-service is failing the 'has-runbook' check.
How do I fix it?
```

### 5. Team Overview

```
/scorecard-review

Show scorecard status for all services owned by platform-team
```

### 6. Progress Tracking

```
/scorecard-review

How has our documentation score improved this month?
```

## Common Scorecard Checks

### Production Readiness

| Check | Description |
|-------|-------------|
| Has CI/CD Pipeline | Service has automated deployment |
| Automated Testing | Test coverage meets threshold |
| Monitoring Configured | Metrics being collected |
| Alerting Setup | Alerts defined for key metrics |
| Runbook Exists | Operational runbook documented |
| On-Call Rotation | Team has on-call schedule |
| SLO Defined | Service level objectives set |
| Disaster Recovery | DR plan documented |

### Security Compliance

| Check | Description |
|-------|-------------|
| Security Scan Passing | No critical vulnerabilities |
| Secrets Managed | No hardcoded secrets |
| Authentication Required | Endpoints protected |
| Dependency Up-to-Date | No known CVEs |
| Data Classification | Data handling documented |

### Documentation

| Check | Description |
|-------|-------------|
| Has Tech Docs | Technical documentation exists |
| API Documented | OpenAPI/GraphQL spec available |
| README Complete | README has required sections |
| Architecture Diagram | System diagram available |
| Changelog Maintained | Version history documented |

### Operational Excellence

| Check | Description |
|-------|-------------|
| Observability Stack | Logs, metrics, traces |
| Graceful Degradation | Handles dependency failures |
| Rate Limiting | Protects against overload |
| Health Endpoints | Liveness/readiness probes |
| Chaos Testing | Resilience testing done |

## Example Usage

### Quick Check

```
/scorecard-review

Check the scorecard for payment-service
```

### All Services

```
/scorecard-review

Show me the worst scoring services
```

### Specific Scorecard

```
/scorecard-review

How are we doing on security compliance?
```

### Fix Issues

```
/scorecard-review

Help me improve the checkout-service scorecard score
```

## Instructions

When reviewing scorecards:

1. **Understand context:**
   - Which service or team?
   - Which scorecard(s)?
   - Current vs historical?

2. **Gather data:**
   - Get score summaries
   - Get detailed check results
   - Get check definitions

3. **Present clearly:**
   - Lead with overall status
   - Show passing vs failing
   - Highlight trends

4. **Provide remediation:**
   - For failing checks, show fix steps
   - Include code/config examples
   - Link to documentation

5. **Prioritize improvements:**
   - Focus on high-weight checks
   - Consider effort vs impact
   - Suggest quick wins first
