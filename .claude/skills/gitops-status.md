---
name: gitops-status
description: Check GitOps application status and health using Harness GitOps MCP tools. Monitor ArgoCD applications, sync status, and troubleshoot issues.
triggers:
  - gitops status
  - argocd status
  - gitops apps
  - sync status
  - application health
  - gitops dashboard
  - argo applications
---

# GitOps Status Skill

Check GitOps application status and health using Harness GitOps MCP tools.

## Overview

This skill helps platform and DevOps teams:
- Monitor ArgoCD application health
- Check sync status across environments
- View resource trees and pod status
- Troubleshoot deployment issues
- Get pod logs for debugging

## Required MCP Tools

This skill requires the Harness MCP Server with the `gitops` toolset enabled:

### Application Tools
- `gitops_list_applications` - List all applications
- `gitops_get_application` - Get application details
- `gitops_get_app_resource_tree` - View resource hierarchy
- `gitops_list_app_events` - Application events

### Resource Tools
- `gitops_get_pod_logs` - Fetch pod logs
- `gitops_get_managed_resources` - List managed resources
- `gitops_list_resource_actions` - Available resource actions

### ApplicationSet Tools
- `gitops_list_applicationsets` - List application sets
- `gitops_get_applicationset` - Get application set details

### Cluster Tools
- `gitops_list_clusters` - List registered clusters
- `gitops_get_cluster` - Get cluster details

### Repository Tools
- `gitops_list_repositories` - List Git repositories
- `gitops_get_repository` - Get repository details
- `gitops_list_repo_credentials` - List repo credentials

### Agent Tools
- `gitops_list_agents` - List GitOps agents
- `gitops_get_agent` - Get agent details

### Dashboard
- `gitops_get_dashboard_overview` - Overall GitOps dashboard

## Workflow

### Step 1: Get Dashboard Overview

Start with the high-level view:

```
Use MCP tool: gitops_get_dashboard_overview
Parameters:
  - org_id: <organization>
  - project_id: <project>
```

This provides:
- Total applications
- Healthy vs unhealthy count
- Sync status summary
- Recent activity

### Step 2: List Applications

Get all GitOps applications:

```
Use MCP tool: gitops_list_applications
Parameters:
  - agent_id: <agent identifier> (optional)
  - org_id: <organization>
  - project_id: <project>
  - page_size: 50
```

Filter options:
- By agent
- By cluster
- By health status
- By sync status

### Step 3: Get Application Details

For specific application status:

```
Use MCP tool: gitops_get_application
Parameters:
  - agent_id: <agent identifier>
  - app_name: <application name>
  - org_id: <organization>
  - project_id: <project>
```

Returns:
- Health status
- Sync status
- Source (repo, path, revision)
- Destination (cluster, namespace)
- Conditions and messages

### Step 4: View Resource Tree

See the Kubernetes resource hierarchy:

```
Use MCP tool: gitops_get_app_resource_tree
Parameters:
  - agent_id: <agent identifier>
  - app_name: <application name>
  - org_id: <organization>
  - project_id: <project>
```

Shows:
- Deployments, Services, ConfigMaps
- Pods and their status
- ReplicaSets
- Health of each resource

### Step 5: Get Application Events

Check recent events:

```
Use MCP tool: gitops_list_app_events
Parameters:
  - agent_id: <agent identifier>
  - app_name: <application name>
  - org_id: <organization>
  - project_id: <project>
```

### Step 6: Get Pod Logs

For debugging failing pods:

```
Use MCP tool: gitops_get_pod_logs
Parameters:
  - agent_id: <agent identifier>
  - app_name: <application name>
  - pod_name: <pod name>
  - namespace: <namespace>
  - container: <container name> (optional)
  - tail_lines: 100
  - org_id: <organization>
  - project_id: <project>
```

### Step 7: Check Clusters

View cluster status:

```
Use MCP tool: gitops_list_clusters
Parameters:
  - agent_id: <agent identifier>
  - org_id: <organization>
  - project_id: <project>
```

Get cluster details:

```
Use MCP tool: gitops_get_cluster
Parameters:
  - agent_id: <agent identifier>
  - cluster_id: <cluster identifier>
  - org_id: <organization>
  - project_id: <project>
```

### Step 8: Check Agents

List GitOps agents:

```
Use MCP tool: gitops_list_agents
Parameters:
  - org_id: <organization>
  - project_id: <project>
```

## Response Format

### Dashboard Summary

```markdown
## GitOps Dashboard

**Generated:** <date>
**Scope:** <project>

### Overall Health

| Status | Count |
|--------|-------|
| Healthy | 24 |
| Degraded | 2 |
| Progressing | 1 |
| Missing | 0 |
| Unknown | 0 |

### Sync Status

| Status | Count |
|--------|-------|
| Synced | 23 |
| OutOfSync | 3 |
| Unknown | 1 |

### Agents

| Agent | Status | Applications |
|-------|--------|--------------|
| prod-agent | Connected | 15 |
| staging-agent | Connected | 10 |
| dev-agent | Connected | 2 |

### Attention Required

⚠️ **2 applications need attention:**

1. **payment-service** (prod-cluster)
   - Status: Degraded
   - Issue: Pod CrashLoopBackOff

2. **notification-service** (staging-cluster)
   - Status: OutOfSync
   - Issue: Pending sync for 2 hours
```

### Application Status Report

```markdown
## Application Status: api-gateway

**Agent:** prod-agent
**Cluster:** prod-cluster
**Namespace:** production

### Health & Sync

| Metric | Status |
|--------|--------|
| Health | ✅ Healthy |
| Sync | ✅ Synced |
| Operation | None in progress |

### Source

| Property | Value |
|----------|-------|
| Repository | github.com/org/k8s-manifests |
| Path | apps/api-gateway |
| Target Revision | main |
| Current Revision | abc123d |

### Resource Tree

```
Application: api-gateway
├── Service: api-gateway-svc ✅
├── Deployment: api-gateway ✅
│   └── ReplicaSet: api-gateway-7d9f8b ✅
│       ├── Pod: api-gateway-7d9f8b-abc12 ✅ Running
│       ├── Pod: api-gateway-7d9f8b-def34 ✅ Running
│       └── Pod: api-gateway-7d9f8b-ghi56 ✅ Running
├── ConfigMap: api-gateway-config ✅
├── Secret: api-gateway-secrets ✅
├── HorizontalPodAutoscaler: api-gateway-hpa ✅
└── Ingress: api-gateway-ingress ✅
```

### Recent Events

| Time | Type | Reason | Message |
|------|------|--------|---------|
| 10m ago | Normal | Synced | Successfully synced |
| 1h ago | Normal | OperationSucceeded | Sync completed |
| 2h ago | Normal | ResourceUpdated | Deployment updated |
```

### Troubleshooting Report

```markdown
## Troubleshooting: payment-service

**Status:** Degraded
**Issue:** Pod CrashLoopBackOff

### Problem Summary

The payment-service application has pods in CrashLoopBackOff state.

### Affected Resources

| Resource | Status | Issue |
|----------|--------|-------|
| Pod: payment-service-5f7d8-abc12 | CrashLoopBackOff | Exit code 1 |
| Pod: payment-service-5f7d8-def34 | CrashLoopBackOff | Exit code 1 |

### Pod Logs (payment-service-5f7d8-abc12)

```
2024-01-15T10:30:15Z ERROR: Failed to connect to database
2024-01-15T10:30:15Z ERROR: Connection refused: postgres:5432
2024-01-15T10:30:15Z FATAL: Cannot start application without database
```

### Root Cause Analysis

The pods are failing because they cannot connect to the PostgreSQL database.

**Possible causes:**
1. Database service not running
2. Network policy blocking connection
3. Wrong database credentials
4. Database DNS resolution failing

### Recommended Actions

1. **Check database status:**
   ```bash
   kubectl get pods -n production -l app=postgres
   ```

2. **Verify service exists:**
   ```bash
   kubectl get svc postgres -n production
   ```

3. **Check network policies:**
   ```bash
   kubectl get networkpolicies -n production
   ```

4. **Verify credentials:**
   - Check the `db-credentials` secret
   - Ensure it matches database configuration
```

## Common Scenarios

### 1. Overall Health Check

```
/gitops-status

Show me the status of all GitOps applications
```

### 2. Specific Application

```
/gitops-status

What's the status of the api-gateway application
in the production cluster?
```

### 3. Out of Sync Applications

```
/gitops-status

Which applications are out of sync and need attention?
```

### 4. Troubleshoot Failing App

```
/gitops-status

The payment-service is showing degraded - help me debug it
```

### 5. Cluster Overview

```
/gitops-status

Show me all applications deployed to the prod-cluster
```

### 6. Recent Changes

```
/gitops-status

What changed in the last hour across our GitOps apps?
```

## Health Status Reference

### Application Health

| Status | Meaning |
|--------|---------|
| Healthy | All resources are healthy |
| Progressing | Resources are being updated |
| Degraded | One or more resources are unhealthy |
| Suspended | Application is suspended |
| Missing | Resources are missing from cluster |
| Unknown | Health cannot be determined |

### Sync Status

| Status | Meaning |
|--------|---------|
| Synced | Live state matches desired state |
| OutOfSync | Live state differs from desired |
| Unknown | Sync status cannot be determined |

### Common Resource Issues

| Issue | Description | Common Fix |
|-------|-------------|------------|
| CrashLoopBackOff | Container keeps crashing | Check logs for errors |
| ImagePullBackOff | Cannot pull container image | Verify image exists and credentials |
| Pending | Pod cannot be scheduled | Check node resources |
| OOMKilled | Out of memory | Increase memory limits |
| CreateContainerError | Container creation failed | Check container config |

## Example Usage

### Quick Health Check

```
/gitops-status

Are all our GitOps applications healthy?
```

### Environment Status

```
/gitops-status

Show me the status of all staging applications
```

### Debug Mode

```
/gitops-status

Get pod logs for the failing checkout-service pods
```

### Sync Issues

```
/gitops-status

Why is the frontend application out of sync?
```

## Instructions

When checking GitOps status:

1. **Start with overview:**
   - Get dashboard summary
   - Identify any issues immediately
   - Note applications needing attention

2. **Drill down as needed:**
   - Get specific application details
   - View resource trees
   - Check events for context

3. **Troubleshoot issues:**
   - For degraded apps, get pod logs
   - Check resource status
   - Identify root causes

4. **Provide clear status:**
   - Use visual indicators (✅ ⚠️ ❌)
   - Show resource trees
   - Include timestamps

5. **Recommend actions:**
   - For out-of-sync: suggest sync
   - For failures: provide debugging steps
   - For issues: identify root cause
