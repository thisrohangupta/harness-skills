---
name: dora-metrics
description: Generate DORA metrics and engineering performance reports using Harness SEI MCP tools. Track deployment frequency, lead time, change failure rate, and MTTR.
triggers:
  - dora metrics
  - dora report
  - deployment frequency
  - lead time
  - change failure rate
  - mttr
  - engineering metrics
  - team performance
  - sei metrics
  - developer productivity
---

# DORA Metrics Skill

Generate DORA metrics and engineering performance reports using Harness Software Engineering Insights (SEI) MCP tools.

## Overview

This skill helps engineering leaders:
- Track the four DORA metrics
- Analyze team and organization performance
- Identify improvement opportunities
- Compare against industry benchmarks
- Generate executive reports

## Required MCP Tools

This skill requires the Harness MCP Server with the `sei` toolset enabled:

### DORA Metrics Tools
- `sei_deployment_frequency` - Deployment frequency metric
- `sei_efficiency_lead_time` - Lead time for changes
- `sei_change_failure_rate` - Change failure rate
- `sei_mttr` - Mean time to recovery
- `sei_deployment_frequency_drilldown` - Detailed deployment data
- `sei_change_failure_rate_drilldown` - Detailed failure data

### Team & Organization Tools
- `sei_get_team` - Team details
- `sei_get_teams_list` - List all teams
- `sei_get_team_integrations` - Team tool integrations
- `sei_get_team_developers` - Team members
- `sei_get_org_trees` - Organization structure
- `sei_get_org_tree_efficiency_profile` - Org efficiency metrics
- `sei_get_org_tree_productivity_profile` - Org productivity metrics

### Business Alignment Tools
- `sei_get_ba_all_profiles` - Business alignment profiles
- `sei_get_ba_insight_metrics` - Alignment metrics
- `sei_get_ba_insight_summary` - Alignment summary

## The Four DORA Metrics

### 1. Deployment Frequency
**Question:** How often do you deploy to production?

| Performance | Frequency |
|-------------|-----------|
| Elite | Multiple times per day |
| High | Between once per day and once per week |
| Medium | Between once per week and once per month |
| Low | Less than once per month |

### 2. Lead Time for Changes
**Question:** How long does it take from code commit to production?

| Performance | Lead Time |
|-------------|-----------|
| Elite | Less than one hour |
| High | Between one day and one week |
| Medium | Between one week and one month |
| Low | More than one month |

### 3. Change Failure Rate
**Question:** What percentage of deployments cause failures?

| Performance | Failure Rate |
|-------------|--------------|
| Elite | 0-15% |
| High | 16-30% |
| Medium | 31-45% |
| Low | 46-60% |

### 4. Mean Time to Recovery (MTTR)
**Question:** How long does it take to recover from failures?

| Performance | MTTR |
|-------------|------|
| Elite | Less than one hour |
| High | Less than one day |
| Medium | Between one day and one week |
| Low | More than one week |

## Workflow

### Step 1: Get Deployment Frequency

```
Use MCP tool: sei_deployment_frequency
Parameters:
  - team_id: <team identifier> (optional)
  - time_period: "LAST_30_DAYS"
```

For detailed breakdown:

```
Use MCP tool: sei_deployment_frequency_drilldown
Parameters:
  - team_id: <team identifier>
  - time_period: "LAST_30_DAYS"
  - group_by: "week" (or day, month)
```

### Step 2: Get Lead Time

```
Use MCP tool: sei_efficiency_lead_time
Parameters:
  - team_id: <team identifier> (optional)
  - time_period: "LAST_30_DAYS"
```

Lead time breakdown:
- Coding time (first commit to PR open)
- Pickup time (PR open to first review)
- Review time (first review to approval)
- Deploy time (merge to production)

### Step 3: Get Change Failure Rate

```
Use MCP tool: sei_change_failure_rate
Parameters:
  - team_id: <team identifier> (optional)
  - time_period: "LAST_30_DAYS"
```

For failure analysis:

```
Use MCP tool: sei_change_failure_rate_drilldown
Parameters:
  - team_id: <team identifier>
  - time_period: "LAST_30_DAYS"
```

### Step 4: Get MTTR

```
Use MCP tool: sei_mttr
Parameters:
  - team_id: <team identifier> (optional)
  - time_period: "LAST_30_DAYS"
```

### Step 5: Get Team Information

List teams:

```
Use MCP tool: sei_get_teams_list
```

Get team details:

```
Use MCP tool: sei_get_team
Parameters:
  - team_id: <team identifier>
```

Get team members:

```
Use MCP tool: sei_get_team_developers
Parameters:
  - team_id: <team identifier>
```

### Step 6: Organization View

Get org structure:

```
Use MCP tool: sei_get_org_trees
```

Get org-level metrics:

```
Use MCP tool: sei_get_org_tree_efficiency_profile
Parameters:
  - org_tree_id: <org tree identifier>
```

```
Use MCP tool: sei_get_org_tree_productivity_profile
Parameters:
  - org_tree_id: <org tree identifier>
```

### Step 7: Business Alignment

Get alignment profiles:

```
Use MCP tool: sei_get_ba_all_profiles
```

Get alignment metrics:

```
Use MCP tool: sei_get_ba_insight_metrics
Parameters:
  - profile_id: <profile identifier>
```

## Response Format

### Executive DORA Summary

```markdown
## DORA Metrics Report

**Team:** Platform Engineering
**Period:** Last 30 Days
**Generated:** <date>

### Performance Summary

| Metric | Value | Performance | Trend |
|--------|-------|-------------|-------|
| Deployment Frequency | 4.2/day | Elite | ⬆️ +15% |
| Lead Time | 2.3 hours | Elite | ⬇️ -20% |
| Change Failure Rate | 8.5% | Elite | ⬇️ -3% |
| MTTR | 45 min | Elite | ⬇️ -25% |

### Overall Assessment

🏆 **Elite Performer** - Your team is performing at the highest level across all DORA metrics.

### 30-Day Trend

```
Deployment Frequency:
Week 1: ████████░░ 3.8/day
Week 2: █████████░ 4.1/day
Week 3: █████████░ 4.3/day
Week 4: ██████████ 4.5/day
```

### Key Insights

1. **Deployment velocity increasing** - 15% more deployments than last month
2. **Lead time improved** - New CI/CD optimizations cutting deploy time
3. **Reliability maintained** - Failure rate stable despite increased velocity
4. **Faster recovery** - Improved observability reducing MTTR
```

### Detailed Metrics Report

```markdown
## Deployment Frequency Analysis

**Total Deployments:** 126
**Daily Average:** 4.2
**Performance Level:** Elite

### By Service

| Service | Deployments | Frequency | Trend |
|---------|-------------|-----------|-------|
| api-gateway | 45 | 1.5/day | ⬆️ |
| user-service | 38 | 1.3/day | → |
| payment-service | 28 | 0.9/day | ⬆️ |
| notification-service | 15 | 0.5/day | ⬇️ |

### By Day of Week

| Day | Deployments | Avg |
|-----|-------------|-----|
| Monday | 28 | 7.0 |
| Tuesday | 32 | 8.0 |
| Wednesday | 35 | 8.8 |
| Thursday | 22 | 5.5 |
| Friday | 9 | 2.3 |

### Deployment Patterns

- **Peak Time:** Tuesday-Wednesday, 10am-2pm
- **Low Activity:** Friday afternoon, weekends
- **Hotfix Ratio:** 8% of deployments

---

## Lead Time Breakdown

**Total Lead Time:** 2.3 hours
**Performance Level:** Elite

### Stage Breakdown

| Stage | Time | % of Total |
|-------|------|------------|
| Coding Time | 45 min | 33% |
| PR Pickup | 15 min | 11% |
| Review Time | 35 min | 25% |
| Merge to Deploy | 45 min | 31% |

### Bottleneck Analysis

🔍 **Observation:** Review time is the largest controllable factor.

**Recommendations:**
1. Enable auto-assignment for PR reviews
2. Set SLA for initial review (< 2 hours)
3. Use smaller PRs (< 200 lines)

---

## Change Failure Rate

**Failed Deployments:** 11 of 126
**Failure Rate:** 8.5%
**Performance Level:** Elite

### Failure Categories

| Category | Count | % |
|----------|-------|---|
| Test Failures | 5 | 45% |
| Config Errors | 3 | 27% |
| Dependency Issues | 2 | 18% |
| Infrastructure | 1 | 10% |

### Recent Failures

| Date | Service | Cause | Resolution |
|------|---------|-------|------------|
| Jan 15 | api-gateway | Config mismatch | Rollback, fix config |
| Jan 12 | payment-service | Test flake | Retry successful |
| Jan 10 | user-service | Missing env var | Added to config |

---

## Mean Time to Recovery

**Average MTTR:** 45 minutes
**Performance Level:** Elite

### Recovery Time Distribution

| Duration | Count | % |
|----------|-------|---|
| < 15 min | 4 | 36% |
| 15-30 min | 3 | 27% |
| 30-60 min | 3 | 27% |
| > 60 min | 1 | 10% |

### Incident Response

**Detection Method:**
- Automated alerts: 82%
- Manual detection: 18%

**Resolution Method:**
- Rollback: 64%
- Hotfix: 27%
- Config change: 9%
```

### Team Comparison Report

```markdown
## Team Performance Comparison

**Period:** Last 30 Days
**Teams:** 5

### DORA Metrics by Team

| Team | Deploy Freq | Lead Time | CFR | MTTR | Level |
|------|-------------|-----------|-----|------|-------|
| Platform | 4.2/day | 2.3h | 8.5% | 45m | Elite |
| Payments | 2.1/day | 4.5h | 12% | 1.2h | Elite |
| Mobile | 0.8/day | 8.2h | 15% | 2.5h | High |
| Data | 0.5/day | 24h | 22% | 4h | Medium |
| Legacy | 0.1/day | 72h | 35% | 8h | Low |

### Improvement Opportunities

**Data Team:**
- Lead time too high → Automate more testing
- Deploy frequency low → Break down monolith

**Legacy Team:**
- All metrics need improvement
- Consider modernization initiative
- Start with automated testing

### Best Practices to Share

From **Platform Team:**
1. Feature flags for safe deployments
2. Automated rollback on error spike
3. Trunk-based development
4. 15-minute PR review SLA
```

## Common Scenarios

### 1. Monthly DORA Report

```
/dora-metrics

Generate the monthly DORA metrics report
for the engineering organization
```

### 2. Team Deep Dive

```
/dora-metrics

Show me detailed DORA metrics for the
platform team over the last quarter
```

### 3. Metric Investigation

```
/dora-metrics

Our change failure rate increased last week.
Help me understand why.
```

### 4. Team Comparison

```
/dora-metrics

Compare DORA metrics across all teams
and identify who needs support
```

### 5. Improvement Tracking

```
/dora-metrics

Are we improving our lead time?
Show me the trend over the last 6 months.
```

### 6. Executive Summary

```
/dora-metrics

Create an executive summary of engineering
performance for the board meeting
```

## Improving DORA Metrics

### Improving Deployment Frequency

**Barriers:**
- Manual testing
- Complex deployment process
- Fear of breaking production
- Large batch sizes

**Solutions:**
- Automate testing and deployments
- Implement feature flags
- Use canary/blue-green deployments
- Work in smaller batches

### Improving Lead Time

**Barriers:**
- Long review cycles
- Complex builds
- Manual processes
- Dependencies between teams

**Solutions:**
- Set PR review SLAs
- Parallelize CI/CD
- Automate approvals for low-risk changes
- Enable self-service deployments

### Improving Change Failure Rate

**Barriers:**
- Insufficient testing
- Environment differences
- Configuration drift
- Inadequate monitoring

**Solutions:**
- Increase test coverage
- Use infrastructure as code
- Implement GitOps
- Add pre-deploy validation

### Improving MTTR

**Barriers:**
- Slow detection
- Complex debugging
- Manual rollback
- Knowledge silos

**Solutions:**
- Implement observability
- Automate rollbacks
- Create runbooks
- Practice incident response

## Example Usage

### Quick Overview

```
/dora-metrics

How are we doing on DORA metrics?
```

### Specific Metric

```
/dora-metrics

What's our deployment frequency this month
compared to last month?
```

### Trend Analysis

```
/dora-metrics

Show me the MTTR trend over the last 6 months
and identify any patterns
```

### Benchmarking

```
/dora-metrics

How do we compare to industry elite performers?
```

## Instructions

When generating DORA metrics reports:

1. **Clarify scope:**
   - Which team(s)?
   - What time period?
   - Which metrics?
   - Comparison needed?

2. **Gather all metrics:**
   - Fetch each DORA metric
   - Get team information
   - Include historical data for trends

3. **Contextualize performance:**
   - Compare to benchmarks
   - Note trends (improving/declining)
   - Identify outliers

4. **Provide insights:**
   - Explain what the numbers mean
   - Identify root causes
   - Highlight successes

5. **Recommend actions:**
   - Specific improvements
   - Best practices to adopt
   - Resources or support needed
