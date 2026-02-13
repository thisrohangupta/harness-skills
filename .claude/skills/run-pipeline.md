---
name: run-pipeline
description: Trigger and monitor Harness pipeline executions using MCP Server tools. Execute pipelines with custom inputs and track progress.
triggers:
  - run pipeline
  - execute pipeline
  - trigger pipeline
  - start pipeline
  - deploy
  - run deployment
  - trigger deployment
---

# Run Pipeline Skill

Trigger and monitor Harness pipeline executions using MCP Server tools.

## Overview

This skill enables developers to:
- List available pipelines
- View pipeline inputs and requirements
- Trigger pipeline executions with custom inputs
- Monitor execution progress
- Get execution results and artifacts

## Required MCP Tools

This skill requires the Harness MCP Server with these toolsets:
- `pipelines` - For pipeline listing and execution
- `default` - For input sets and triggers

## Workflow

### Step 1: Find the Pipeline

If pipeline not specified, list available pipelines:

```
Use MCP tool: list_pipelines
Parameters:
  - org_id: <organization>
  - project_id: <project>
  - search_term: <optional filter>
  - page_size: 20
```

### Step 2: Get Pipeline Details

Understand the pipeline structure and requirements:

```
Use MCP tool: get_pipeline
Parameters:
  - pipeline_id: <pipeline identifier>
  - org_id: <organization>
  - project_id: <project>
```

Extract from response:
- Pipeline inputs (variables)
- Required vs optional inputs
- Input types and defaults
- Stage structure

### Step 3: Get Available Input Sets

Check for predefined input configurations:

```
Use MCP tool: list_input_sets
Parameters:
  - pipeline_id: <pipeline identifier>
  - org_id: <organization>
  - project_id: <project>
```

Input sets provide pre-configured values that can be used or overridden.

### Step 4: Prepare Execution Inputs

Based on user request and pipeline requirements:

1. **Map user inputs to pipeline variables:**
   ```yaml
   # User says: "deploy version 1.2.3 to staging"
   # Map to pipeline inputs:
   variables:
     - name: version
       value: "1.2.3"
     - name: environment
       value: "staging"
   ```

2. **Validate required inputs are provided:**
   - Check all required inputs have values
   - Apply defaults for optional inputs
   - Validate input types (string, number, secret reference)

3. **Handle special input types:**
   - **Secrets:** Reference as `<+secrets.getValue("secret_id")>`
   - **Connectors:** Reference by identifier
   - **Artifacts:** Specify version/tag

### Step 5: Trigger Pipeline Execution

Execute the pipeline with inputs:

```
Use MCP tool: execute_pipeline (or equivalent)
Parameters:
  - pipeline_id: <pipeline identifier>
  - org_id: <organization>
  - project_id: <project>
  - inputs: <input yaml/json>
  - input_set_refs: <optional input set references>
  - notes: <execution notes>
```

### Step 6: Monitor Execution

Track execution progress:

```
Use MCP tool: get_execution
Parameters:
  - pipeline_id: <pipeline identifier>
  - execution_id: <execution id from trigger>
  - org_id: <organization>
  - project_id: <project>
```

Poll for status updates:
- `Running` - Execution in progress
- `Success` - Completed successfully
- `Failed` - Execution failed
- `Aborted` - User cancelled
- `Waiting` - Awaiting approval or input

### Step 7: Get Execution URL

Provide link to Harness UI:

```
Use MCP tool: fetch_execution_url
Parameters:
  - pipeline_id: <pipeline identifier>
  - execution_id: <execution id>
  - org_id: <organization>
  - project_id: <project>
```

## Response Format

### Before Execution

```markdown
## Pipeline Execution Request

**Pipeline:** <pipeline_name>
**Project:** <project_id>

### Required Inputs

| Input | Type | Description | Your Value |
|-------|------|-------------|------------|
| version | string | Version to deploy | 1.2.3 |
| environment | string | Target environment | staging |

### Optional Inputs (using defaults)

| Input | Default |
|-------|---------|
| notify_slack | true |
| run_tests | true |

### Available Input Sets

- `staging-defaults` - Standard staging configuration
- `prod-inputs` - Production with approvals

**Ready to execute?** Confirm to proceed or provide additional inputs.
```

### After Trigger

```markdown
## Pipeline Execution Started

**Pipeline:** <pipeline_name>
**Execution ID:** <execution_id>
**Status:** Running

**Inputs Used:**
- version: 1.2.3
- environment: staging

**View in Harness:** <execution_url>

I'll monitor the execution. Current stage: Build
```

### Execution Complete

```markdown
## Pipeline Execution Complete

**Pipeline:** <pipeline_name>
**Execution ID:** <execution_id>
**Status:** Success
**Duration:** 4m 32s

### Stage Results

| Stage | Status | Duration |
|-------|--------|----------|
| Build | Success | 2m 15s |
| Test | Success | 1m 45s |
| Deploy | Success | 32s |

### Outputs

- Image: myregistry.io/app:1.2.3
- Deployment URL: https://staging.myapp.com

**View Details:** <execution_url>
```

## Input Handling

### String Inputs

```yaml
variables:
  - name: version
    type: String
    value: "1.2.3"
```

### Number Inputs

```yaml
variables:
  - name: replicas
    type: Number
    value: 3
```

### Secret References

```yaml
variables:
  - name: api_key
    type: String
    value: "<+secrets.getValue(\"account.api_key\")>"
```

### Connector References

```yaml
variables:
  - name: docker_connector
    type: String
    value: "account.docker_hub"
```

### Runtime Inputs

For inputs marked as runtime (`<+input>`), user must provide values:

```yaml
# Pipeline defines:
# version: <+input>

# User provides:
variables:
  - name: version
    value: "1.2.3"
```

## Common Scenarios

### Deploy to Environment

```
/run-pipeline

Deploy the api-service to staging with version 2.1.0
```

Maps to:
- pipeline: api-service-deploy (or find by search)
- environment: staging
- version: 2.1.0

### Run CI Pipeline

```
/run-pipeline

Run CI for the main branch
```

Maps to:
- pipeline: ci-pipeline
- branch: main

### Deploy with Approval Bypass

```
/run-pipeline

Deploy to production with version 3.0.0
Skip the approval stage (I'm authorized)
```

Note: Only skip if user confirms authorization and pipeline allows.

### Use Input Set

```
/run-pipeline

Run the deploy pipeline using the staging-defaults input set
but override version to 1.5.0
```

Maps to:
- input_set_refs: ["staging-defaults"]
- Override: version = 1.5.0

## Safety Considerations

### Production Deployments

When deploying to production:

1. **Confirm intent:**
   ```
   You're about to deploy to PRODUCTION:
   - Pipeline: deploy-pipeline
   - Version: 3.0.0
   - Environment: production

   This will affect live users. Confirm? (yes/no)
   ```

2. **Check for approval stages:**
   - Notify user if pipeline has approval gates
   - Explain who needs to approve

3. **Verify prerequisites:**
   - Has this version been tested in staging?
   - Are there any active incidents?

### Destructive Operations

For pipelines that might:
- Delete resources
- Drop databases
- Modify infrastructure

Always confirm:
```
This pipeline includes destructive operations:
- Stage: Cleanup
- Action: Delete old deployments

Confirm you want to proceed? (yes/no)
```

## Error Handling

### Pipeline Not Found

```
I couldn't find a pipeline matching "api-deploy".

Available pipelines in project:
- api-service-deploy
- api-service-ci
- frontend-deploy

Did you mean one of these?
```

### Missing Required Inputs

```
The pipeline requires these inputs that weren't provided:

| Input | Type | Description |
|-------|------|-------------|
| version | string | Version to deploy |
| environment | string | Target environment |

Please provide values for these inputs.
```

### Execution Failed to Start

```
Failed to start pipeline execution:

Error: Delegate not available
Details: No delegate matched selector: linux-amd64

Suggestions:
1. Check delegate status in Harness UI
2. Verify delegate tags in pipeline match available delegates
3. Wait for delegate to come online
```

## Example Usage

### Basic Execution

```
/run-pipeline

Run the ci-pipeline
```

### With Inputs

```
/run-pipeline

Deploy version 2.0.0 of the backend service to staging
```

### List and Select

```
/run-pipeline

Show me available pipelines for the payments project
```

### Monitor Existing

```
/run-pipeline

What's the status of execution xyz123?
```

## Troubleshooting

### Pipeline Won't Start

1. **Check delegate availability:**
   - Verify delegate is online
   - Check delegate tags match pipeline requirements
   - Review delegate resource limits

2. **Validate inputs:**
   - All required inputs must have values
   - Check input types match expected
   - Verify secret references exist

3. **Check connector status:**
   - Test connectors before execution
   - Verify credentials haven't expired

### Input Set Issues

1. **Input set not found:**
   - Verify input set belongs to this pipeline
   - Check identifier spelling
   - Ensure input set is at correct scope

2. **Merge conflicts:**
   - Later input sets override earlier
   - Check for conflicting values
   - Verify all required inputs covered

3. **Validation failures:**
   - Input values must match pipeline variable types
   - Allowed values must be respected
   - Required fields cannot be empty

### Execution Monitoring Issues

1. **Status not updating:**
   - Poll at reasonable intervals (5-10 seconds)
   - Check for network timeouts
   - Verify execution ID is correct

2. **Execution URL not working:**
   - Check user has UI access
   - Verify account/org/project in URL
   - Ensure execution exists

### Common Execution Failures

1. **Timeout errors:**
   - Increase step/stage timeout
   - Check for deadlocks
   - Review resource availability

2. **Authentication failures:**
   - Rotate expired credentials
   - Verify connector configuration
   - Check secret references

3. **Resource not found:**
   - Verify service/environment exists
   - Check identifier spelling
   - Ensure resources at correct scope

## Instructions

When a user wants to run a pipeline:

1. **Identify the pipeline:**
   - Use exact name/ID if provided
   - Search if partial name given
   - List options if ambiguous

2. **Gather inputs:**
   - Get pipeline definition to see required inputs
   - Map user's natural language to input variables
   - Ask for missing required inputs

3. **Confirm before execution:**
   - Show what will be executed
   - List inputs being used
   - Highlight any risks (production, destructive)

4. **Execute and monitor:**
   - Trigger the pipeline
   - Provide execution URL immediately
   - Optionally poll for status updates

5. **Report results:**
   - Show final status
   - Include stage-by-stage results
   - Provide any output artifacts/values
