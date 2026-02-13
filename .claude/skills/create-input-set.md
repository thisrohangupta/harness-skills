---
name: create-input-set
description: Generate Harness.io Input Set YAML definitions and optionally create them via the Harness API. Input sets provide reusable runtime inputs for pipelines.
triggers:
  - harness input set
  - create input set
  - input set
  - runtime inputs
  - pipeline inputs
  - create input set api
---

# Create Input Set Skill

Generate Harness.io Input Set YAML definitions and create them via the API.

## Overview

Input Sets in Harness provide reusable collections of runtime inputs for pipelines. They allow you to:
- Pre-define values for pipeline inputs
- Create environment-specific configurations
- Combine multiple input sets (overlay sets)
- Standardize deployment parameters
- Create input sets via the Harness API

## Input Set Structure

Every input set follows this structure:

```yaml
inputSet:
  identifier: <unique_identifier>
  name: <display_name>
  description: <optional_description>
  tags:
    key: value
  orgIdentifier: <org_id>
  projectIdentifier: <project_id>
  pipeline:
    identifier: <pipeline_identifier>
    # Input values matching pipeline structure
```

## Basic Input Set

```yaml
inputSet:
  identifier: dev_inputs
  name: Development Inputs
  description: Input values for development deployments
  orgIdentifier: default
  projectIdentifier: my_project
  tags:
    environment: development
  pipeline:
    identifier: deploy_pipeline
    variables:
      - name: environment
        type: String
        value: dev
      - name: replicas
        type: Number
        value: 1
```

## Input Set Examples

### CI Pipeline Input Set

```yaml
inputSet:
  identifier: ci_main_branch
  name: CI Main Branch
  description: Inputs for main branch builds
  pipeline:
    identifier: ci_pipeline
    properties:
      ci:
        codebase:
          build:
            type: branch
            spec:
              branch: main
    stages:
      - stage:
          identifier: build
          type: CI
          spec:
            execution:
              steps:
                - step:
                    identifier: build_image
                    type: BuildAndPushDockerRegistry
                    spec:
                      tags:
                        - latest
                        - <+pipeline.sequenceId>
```

### CD Pipeline Input Set

```yaml
inputSet:
  identifier: deploy_staging
  name: Deploy to Staging
  description: Inputs for staging deployments
  pipeline:
    identifier: deploy_pipeline
    variables:
      - name: environment
        type: String
        value: staging
      - name: replicas
        type: Number
        value: 2
      - name: image_tag
        type: String
        value: <+trigger.artifact.build>
    stages:
      - stage:
          identifier: deploy
          type: Deployment
          spec:
            environment:
              environmentRef: staging
              infrastructureDefinitions:
                - identifier: k8s_staging
```

### Production Input Set

```yaml
inputSet:
  identifier: deploy_production
  name: Deploy to Production
  description: Inputs for production deployments
  pipeline:
    identifier: deploy_pipeline
    variables:
      - name: environment
        type: String
        value: production
      - name: replicas
        type: Number
        value: 5
      - name: enable_canary
        type: String
        value: "true"
    stages:
      - stage:
          identifier: approval
          type: Approval
          spec:
            execution:
              steps:
                - step:
                    identifier: manual_approval
                    type: HarnessApproval
                    spec:
                      approvers:
                        userGroups:
                          - prod_approvers
      - stage:
          identifier: deploy
          type: Deployment
          spec:
            environment:
              environmentRef: prod
              infrastructureDefinitions:
                - identifier: k8s_prod
```

### Artifact-Specific Input Set

```yaml
inputSet:
  identifier: release_v2
  name: Release v2.0.0
  description: Inputs for v2.0.0 release
  pipeline:
    identifier: deploy_pipeline
    stages:
      - stage:
          identifier: deploy
          type: Deployment
          spec:
            service:
              serviceRef: my_service
              serviceInputs:
                serviceDefinition:
                  type: Kubernetes
                  spec:
                    artifacts:
                      primary:
                        primaryArtifactRef: docker_image
                        sources:
                          - identifier: docker_image
                            type: DockerRegistry
                            spec:
                              tag: v2.0.0
```

### Multi-Service Input Set

```yaml
inputSet:
  identifier: full_stack_staging
  name: Full Stack Staging Deploy
  description: Deploy all services to staging
  pipeline:
    identifier: multi_service_deploy
    stages:
      - stage:
          identifier: deploy_backend
          spec:
            service:
              serviceRef: backend_service
            environment:
              environmentRef: staging
      - stage:
          identifier: deploy_frontend
          spec:
            service:
              serviceRef: frontend_service
            environment:
              environmentRef: staging
      - stage:
          identifier: deploy_worker
          spec:
            service:
              serviceRef: worker_service
            environment:
              environmentRef: staging
```

## Overlay Input Sets

Combine multiple input sets:

```yaml
overlayInputSet:
  identifier: staging_with_canary
  name: Staging with Canary
  description: Combines staging inputs with canary deployment
  orgIdentifier: default
  projectIdentifier: my_project
  pipeline:
    identifier: deploy_pipeline
  inputSetReferences:
    - deploy_staging
    - canary_config
  tags:
    type: overlay
```

### Overlay Input Set Example

```yaml
# Base Input Set
inputSet:
  identifier: base_config
  name: Base Configuration
  pipeline:
    identifier: deploy_pipeline
    variables:
      - name: log_level
        value: info
      - name: monitoring_enabled
        value: "true"

# Environment-Specific Input Set
inputSet:
  identifier: staging_env
  name: Staging Environment
  pipeline:
    identifier: deploy_pipeline
    stages:
      - stage:
          identifier: deploy
          spec:
            environment:
              environmentRef: staging

# Overlay combining both
overlayInputSet:
  identifier: staging_complete
  name: Complete Staging Config
  pipeline:
    identifier: deploy_pipeline
  inputSetReferences:
    - base_config
    - staging_env
```

## Creating Input Sets via API

### API Reference

**Endpoint:** `POST /v1/orgs/{org}/projects/{project}/input-sets`

**Documentation:** https://apidocs.harness.io/tag/Input-Sets

### Request Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-api-key` | Yes | Harness API key |
| `Harness-Account` | Yes | Account identifier |
| `Content-Type` | Yes | `application/json` |

### Query Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `pipeline` | Yes | Pipeline identifier |

### Request Body

```json
{
  "input_set_yaml": "inputSet:\n  identifier: dev_inputs\n  name: Development Inputs\n  ..."
}
```

### Example: Create Input Set

```bash
curl -X POST \
  'https://app.harness.io/v1/orgs/{org}/projects/{project}/input-sets?pipeline={pipelineId}' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}' \
  -H 'Content-Type: application/json' \
  -d '{
    "input_set_yaml": "inputSet:\n  identifier: dev_inputs\n  name: Development Inputs\n  orgIdentifier: default\n  projectIdentifier: my_project\n  pipeline:\n    identifier: deploy_pipeline\n    variables:\n      - name: environment\n        type: String\n        value: dev\n      - name: replicas\n        type: Number\n        value: 1"
  }'
```

### Example: Create Input Set from YAML File

```bash
INPUT_SET_YAML=$(cat input-set.yaml)

curl -X POST \
  'https://app.harness.io/v1/orgs/default/projects/my_project/input-sets?pipeline=deploy_pipeline' \
  -H 'x-api-key: pat.xxxx.yyyy.zzzz' \
  -H 'Harness-Account: abc123' \
  -H 'Content-Type: application/json' \
  -d "$(jq -n --arg yaml "$INPUT_SET_YAML" '{input_set_yaml: $yaml}')"
```

### Response

**Success (201 Created):**

```json
{
  "identifier": "dev_inputs",
  "name": "Development Inputs",
  "pipeline": "deploy_pipeline",
  "org": "default",
  "project": "my_project",
  "created": 1707500000000,
  "updated": 1707500000000
}
```

### Update Input Set

**Endpoint:** `PUT /v1/orgs/{org}/projects/{project}/input-sets/{input-set}`

```bash
curl -X PUT \
  'https://app.harness.io/v1/orgs/{org}/projects/{project}/input-sets/{inputSetId}?pipeline={pipelineId}' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}' \
  -H 'Content-Type: application/json' \
  -d '{
    "input_set_yaml": "inputSet:\n  identifier: dev_inputs\n  name: Development Inputs (Updated)\n  ..."
  }'
```

### List Input Sets

**Endpoint:** `GET /v1/orgs/{org}/projects/{project}/input-sets`

```bash
curl -X GET \
  'https://app.harness.io/v1/orgs/{org}/projects/{project}/input-sets?pipeline={pipelineId}' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}'
```

### Get Merged Input Set

Merge multiple input sets for execution:

**Endpoint:** `POST /pipeline/api/inputSets/merge`

```bash
curl -X POST \
  'https://app.harness.io/pipeline/api/inputSets/merge?accountIdentifier={accountId}&orgIdentifier={org}&projectIdentifier={project}&pipelineIdentifier={pipeline}' \
  -H 'x-api-key: {apiKey}' \
  -H 'Content-Type: application/json' \
  -d '{
    "inputSetReferences": ["base_config", "staging_env"],
    "withMergedPipelineYaml": true
  }'
```

## Using Input Sets in Pipeline Execution

### Execute Pipeline with Input Set

```bash
curl -X POST \
  'https://app.harness.io/pipeline/api/pipeline/execute/{pipelineId}?accountIdentifier={accountId}&orgIdentifier={org}&projectIdentifier={project}' \
  -H 'x-api-key: {apiKey}' \
  -H 'Content-Type: application/json' \
  -d '{
    "inputSetReferences": ["dev_inputs"],
    "stageIdentifiers": []
  }'
```

### Execute with Multiple Input Sets

```bash
curl -X POST \
  'https://app.harness.io/pipeline/api/pipeline/execute/{pipelineId}?accountIdentifier={accountId}&orgIdentifier={org}&projectIdentifier={project}' \
  -H 'x-api-key: {apiKey}' \
  -H 'Content-Type: application/json' \
  -d '{
    "inputSetReferences": ["base_config", "staging_env", "canary_config"],
    "stageIdentifiers": []
  }'
```

## Best Practices

### Input Set Organization

| Category | Identifier Pattern | Example |
|----------|-------------------|---------|
| Environment | `{env}_inputs` | `prod_inputs` |
| Release | `release_{version}` | `release_v2_0_0` |
| Feature | `feature_{name}` | `feature_canary` |
| Overlay | `{env}_{feature}` | `staging_canary` |

### Layered Input Sets

Structure input sets in layers:

1. **Base** - Common configuration
2. **Environment** - Environment-specific values
3. **Feature** - Feature flags and options
4. **Release** - Version-specific artifacts

### Example Layered Structure

```yaml
# 1. Base (common to all)
inputSet:
  identifier: base_config
  pipeline:
    identifier: deploy_pipeline
    variables:
      - name: log_level
        value: info
      - name: monitoring
        value: "true"

# 2. Environment (staging)
inputSet:
  identifier: staging_env
  pipeline:
    identifier: deploy_pipeline
    stages:
      - stage:
          identifier: deploy
          spec:
            environment:
              environmentRef: staging

# 3. Feature (canary enabled)
inputSet:
  identifier: canary_enabled
  pipeline:
    identifier: deploy_pipeline
    variables:
      - name: canary_percentage
        value: 10

# 4. Overlay (combines all)
overlayInputSet:
  identifier: staging_canary_deploy
  inputSetReferences:
    - base_config
    - staging_env
    - canary_enabled
```

## Instructions

When creating an input set:

1. **Identify requirements:**
   - Which pipeline does it belong to?
   - What inputs need to be provided?
   - Is it standalone or part of an overlay?

2. **Generate valid YAML:**
   - Match pipeline structure exactly
   - Include only fields that need input values
   - Use correct identifier patterns

3. **Consider organization:**
   - Use layered approach for flexibility
   - Create overlay sets for common combinations
   - Document purpose in description

4. **Output the input set YAML** in a code block

5. **Optionally create via API** if the user requests it
