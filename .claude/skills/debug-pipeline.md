---
name: debug-pipeline
description: Analyze pipeline execution failures using Harness MCP Server tools. Fetches execution logs, identifies error patterns, and suggests fixes.
triggers:
  - debug pipeline
  - pipeline failure
  - execution failed
  - why did pipeline fail
  - analyze failure
  - pipeline error
  - fix pipeline
---

# Debug Pipeline Skill

Analyze pipeline execution failures and suggest fixes using Harness MCP Server tools.

## Overview

This skill helps developers quickly diagnose and fix pipeline failures by:
- Fetching recent execution history
- Downloading and analyzing execution logs
- Identifying error patterns and root causes
- Suggesting specific fixes based on the failure type

## Required MCP Tools

This skill requires the Harness MCP Server with the following toolsets enabled:
- `pipelines` - For pipeline and execution queries
- `logs` - For downloading execution logs

## Workflow

### Step 1: Identify the Pipeline and Execution

First, gather information about the failed execution:

```
Use MCP tool: list_executions
Parameters:
  - pipeline_id: <pipeline identifier>
  - org_id: <organization> (optional if default set)
  - project_id: <project> (optional if default set)
  - status: "Failed" (to filter failed executions)
  - page_size: 5 (recent failures)
```

If no pipeline_id provided, list pipelines first:

```
Use MCP tool: list_pipelines
Parameters:
  - org_id: <organization>
  - project_id: <project>
  - search_term: <optional search>
```

### Step 2: Get Execution Details

Fetch detailed information about the failed execution:

```
Use MCP tool: get_execution
Parameters:
  - pipeline_id: <pipeline identifier>
  - execution_id: <execution identifier>
  - org_id: <organization>
  - project_id: <project>
```

Key information to extract:
- Which stage failed
- Which step failed
- Error message summary
- Execution timing
- Input values used

### Step 3: Download Execution Logs

Get the full logs for detailed analysis:

```
Use MCP tool: download_execution_logs
Parameters:
  - pipeline_id: <pipeline identifier>
  - execution_id: <execution identifier>
  - org_id: <organization>
  - project_id: <project>
```

### Step 4: Get Pipeline Definition

Understand the pipeline configuration:

```
Use MCP tool: get_pipeline
Parameters:
  - pipeline_id: <pipeline identifier>
  - org_id: <organization>
  - project_id: <project>
```

### Step 5: Analyze and Diagnose

Analyze the collected information to identify:

1. **Error Category:**
   - Build errors (compilation, dependencies)
   - Test failures (unit, integration, e2e)
   - Infrastructure errors (connectivity, resources)
   - Configuration errors (missing variables, secrets)
   - Deployment errors (K8s, cloud provider issues)
   - Timeout errors

2. **Root Cause Patterns:**

   **Build Failures:**
   - Missing dependencies → Check package.json/requirements.txt
   - Compilation errors → Review code changes in commit
   - Docker build failures → Check Dockerfile and base image

   **Test Failures:**
   - Flaky tests → Check test isolation and timing
   - Environment issues → Verify test environment setup
   - Data dependencies → Check test data availability

   **Infrastructure Errors:**
   - Delegate offline → Check delegate status
   - Connector failures → Verify credentials and connectivity
   - Resource limits → Check cloud quotas

   **Configuration Errors:**
   - Missing secrets → Verify secret references
   - Invalid variables → Check variable expressions
   - Wrong environment → Verify environment selection

   **Deployment Errors:**
   - K8s manifest errors → Validate YAML syntax
   - Image pull failures → Check registry credentials
   - Resource conflicts → Review existing deployments

### Step 6: Generate Fix Recommendations

Based on the diagnosis, provide:

1. **Immediate Fix:** What to change right now
2. **Root Cause:** Why this happened
3. **Prevention:** How to avoid this in the future

## Response Format

```markdown
## Pipeline Failure Analysis

**Pipeline:** <pipeline_name>
**Execution:** <execution_id>
**Failed At:** <timestamp>
**Duration:** <duration before failure>

### Failure Summary

**Stage:** <failed_stage_name>
**Step:** <failed_step_name>
**Error Type:** <category>

### Error Details

```
<relevant error message from logs>
```

### Root Cause Analysis

<explanation of what went wrong and why>

### Recommended Fix

**Immediate Action:**
<specific steps to fix the issue>

**Code/Config Changes:**
```yaml
# If applicable, show the fix
```

**Prevention:**
<how to prevent this in the future>

### Related Executions

<if pattern detected across multiple executions>
```

## Common Error Patterns

### 1. Delegate Not Available

**Symptoms:**
- "No delegate available to execute task"
- Long queue times before failure

**Fix:**
- Check delegate status in Harness UI
- Verify delegate tags match pipeline requirements
- Scale up delegate if under load

### 2. Secret Not Found

**Symptoms:**
- "Secret not found: <secret_name>"
- "Could not resolve expression"

**Fix:**
- Verify secret exists at correct scope (account/org/project)
- Check secret identifier spelling
- Ensure pipeline has access to secret scope

### 3. Connector Authentication Failed

**Symptoms:**
- "401 Unauthorized"
- "Authentication failed"
- "Invalid credentials"

**Fix:**
- Rotate connector credentials
- Verify connector configuration
- Check token/key expiration

### 4. Docker Image Pull Failed

**Symptoms:**
- "ImagePullBackOff"
- "unauthorized: authentication required"
- "manifest not found"

**Fix:**
- Verify image tag exists
- Check registry connector credentials
- Ensure image path is correct

### 5. Kubernetes Deployment Failed

**Symptoms:**
- "CrashLoopBackOff"
- "OOMKilled"
- "Readiness probe failed"

**Fix:**
- Check container logs for application errors
- Adjust resource limits
- Review probe configuration

### 6. Test Timeout

**Symptoms:**
- "Execution timed out"
- Tests running longer than expected

**Fix:**
- Increase step timeout
- Optimize slow tests
- Check for deadlocks or infinite loops

### 7. Artifact Not Found

**Symptoms:**
- "Artifact not found"
- "No artifacts matched the criteria"

**Fix:**
- Verify artifact path and filters
- Check upstream build completed
- Ensure artifact retention policy

### 8. Terraform/Infrastructure Errors

**Symptoms:**
- "Error acquiring state lock"
- "Resource already exists"
- "Provider configuration not present"

**Fix:**
- Check state backend connectivity
- Force unlock if safe
- Verify provider credentials

## Example Usage

### Basic Usage

```
/debug-pipeline

Analyze why the build-and-deploy pipeline failed
```

### With Specific Execution

```
/debug-pipeline

Debug execution abc123 of the ci-pipeline
```

### Recent Failures

```
/debug-pipeline

Show me the last 3 failures of the staging-deploy pipeline
and identify any patterns
```

## Error Handling

### Common Errors

| Error Code | Description | Solution |
|------------|-------------|----------|
| `EXECUTION_NOT_FOUND` | Execution ID doesn't exist | Verify execution ID and project scope |
| `PIPELINE_NOT_FOUND` | Pipeline doesn't exist | Check pipeline identifier spelling |
| `ACCESS_DENIED` | Insufficient permissions | Verify user has pipeline view access |
| `LOGS_UNAVAILABLE` | Logs expired or not generated | Check log retention policy |
| `MCP_CONNECTION_ERROR` | Cannot connect to MCP server | Verify MCP server is running |

### MCP Tool Errors

```
# Common MCP tool issues:

# Tool not available
Error: Tool 'download_execution_logs' not found
→ Ensure 'logs' toolset is enabled in MCP server config

# Invalid parameters
Error: Missing required parameter 'pipeline_id'
→ Provide all required parameters

# Authentication failed
Error: Invalid or expired API key
→ Check MCP server API key configuration
```

## Troubleshooting

### Logs Not Available

1. **Check execution status:**
   - Logs may not exist for aborted executions
   - Very recent executions may have delayed logs

2. **Verify log retention:**
   - Logs expire based on account settings
   - Check if execution is within retention window

3. **Check delegate logs:**
   - If delegate-side, logs may be on delegate host
   - Use delegate troubleshooting tools

### Cannot Find Execution

1. **Verify scope:**
   - Check org/project identifiers
   - Execution may be in different project

2. **Check filters:**
   - Remove status filter to see all executions
   - Try broader time range

3. **Permissions:**
   - User may not have access to all executions
   - Check RBAC settings

### MCP Tools Not Working

1. **Connection issues:**
   - Verify MCP server is running
   - Check network connectivity
   - Validate API credentials

2. **Toolset configuration:**
   - Ensure required toolsets are enabled
   - Check MCP server logs for errors

3. **Parameter validation:**
   - Verify all required parameters provided
   - Check parameter types (string vs number)

### Analysis Tips

1. **Start broad, then narrow:**
   - Get execution overview first
   - Then drill into specific stages/steps

2. **Check related executions:**
   - Is this a recurring failure?
   - Did previous executions pass?

3. **Review recent changes:**
   - Pipeline modifications
   - Infrastructure changes
   - Connector updates

## Instructions

When a user asks to debug a pipeline:

1. **Gather context:**
   - Which pipeline? (name or ID)
   - Which execution? (specific ID or "last", "recent")
   - Which project/org? (use defaults if not specified)

2. **Use MCP tools to fetch data:**
   - List executions to find failures
   - Get execution details
   - Download logs
   - Get pipeline definition if needed

3. **Analyze systematically:**
   - Identify the failure point (stage/step)
   - Extract relevant error messages
   - Categorize the error type
   - Determine root cause

4. **Provide actionable recommendations:**
   - Specific fix steps
   - Code/config changes if applicable
   - Prevention strategies

5. **Offer follow-up:**
   - Ask if they want to see more details
   - Offer to help implement the fix
   - Suggest related checks (e.g., connector status)
