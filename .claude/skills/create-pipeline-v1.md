---
name: create-pipeline-v1
description: Generate Harness.io v1 Pipeline YAML files using the new simplified syntax. Use when the user wants to create a v1 pipeline, new pipeline format, simplified pipeline, or mentions GitHub Actions compatible syntax.
triggers:
  - v1 pipeline
  - new pipeline
  - simplified pipeline
  - create v1 pipeline
  - github actions
  - harness v1
---

# Create Pipeline v1 Skill

Generate Harness.io v1 Pipeline YAML files using the new simplified syntax.

## Overview

The v1 pipeline format is a simplified, more intuitive YAML syntax that is also compatible with GitHub Actions workflows. It features:

- Cleaner, more concise syntax
- GitHub Actions compatibility
- Simplified step definitions
- Native matrix and looping support
- Built-in caching intelligence
- Expression syntax: `${{ }}`

## Schema Reference

Schema source: https://github.com/harness/harness-schema/tree/main/v1

## Pipeline Structure

### Minimal Pipeline

```yaml
pipeline:
  stages:
  - steps:
    - run: echo "Hello World"
```

### Full Pipeline Structure

```yaml
pipeline:
  # Optional: Pipeline inputs (variables)
  inputs:
    version:
      type: string
      default: "1.0.0"

  # Optional: Global environment variables
  env:
    NODE_ENV: production

  # Optional: Default delegate
  delegate: my-delegate

  # Optional: Repository override
  repo:
    name: org/repo
    connector: github_connector

  # Optional: Clone configuration
  clone:
    depth: 50
    disabled: false

  # Optional: Trigger configuration
  on:
    - push
    - pull_request

  # Optional: Service and environment (for CD)
  service: my-service
  environment: production

  # Optional: Pipeline-level timeout
  timeout: 1h

  # Optional: Conditional execution
  if: ${{ branch == "main" }}

  # Required: Stages
  stages:
    - steps:
        - run: echo "Build"
```

## Stages

### Simple Stage

```yaml
stages:
- steps:
  - run: npm install
  - run: npm test
```

### Stage with ID and Name

```yaml
stages:
- id: build
  name: Build Stage
  steps:
  - run: npm run build
```

### Stage with Runtime

```yaml
# Cloud runtime (short)
stages:
- runtime: cloud
  steps:
  - run: npm test

# Cloud runtime (long)
stages:
- runtime:
    cloud:
      image: ubuntu-latest
      size: large
  platform:
    os: linux
    arch: amd64
  steps:
  - run: npm test

# Kubernetes runtime
stages:
- runtime:
    kubernetes:
      namespace: build-ns
      connector: k8s_connector
  steps:
  - run: npm test

# Shell/VM runtime
stages:
- runtime: shell
  steps:
  - run: npm test
```

### Stage with Service and Environment (CD)

```yaml
stages:
- service: petstore
  environment: prod
  steps:
  - action:
      uses: kubernetes-rolling-deploy
```

### Multi-Service Multi-Environment

```yaml
stages:
- service:
    items:
    - frontend
    - backend
  environment:
    sequential: true
    items:
    - name: prod
      deploy-to: all
    - name: staging
      deploy-to:
      - infra1
      - infra2
  steps:
  - action:
      uses: kubernetes-rolling-deploy
```

### Parallel Stages

```yaml
stages:
- parallel:
    stages:
    - id: test-node
      steps:
      - run: npm test
    - id: test-go
      steps:
      - run: go test ./...
```

### Stage Groups (Sequential)

```yaml
stages:
- group:
    stages:
    - steps:
      - run: npm install
    - steps:
      - run: npm test
```

### Stage with Matrix Strategy

```yaml
stages:
- strategy:
    matrix:
      node: ["16", "18", "20"]
      os: [ubuntu-latest, macos-latest]
    max-parallel: 4
  steps:
  - run: node --version
    container: node:${{ matrix.node }}
```

### Stage with Caching

```yaml
stages:
- cache:
    path: node_modules
    key: npm-${{ hashFiles('package-lock.json') }}
  steps:
  - run: npm ci
  - run: npm test
```

### Approval Stage

```yaml
stages:
- approval:
    uses: harness
    with:
      timeout: 30m
      message: "Approve deployment to production?"
      groups: ["admins", "ops"]
      min-approvers: 1
      auto-reject: true
```

### Stage with Failure Strategy

```yaml
stages:
- steps:
  - run: npm test
  on-failure:
    errors: all
    action: abort
```

### Stage with Outputs

```yaml
stages:
- id: build
  steps:
  - id: version
    run: |
      echo "version=1.0.0" >> $HARNESS_OUTPUT
  outputs:
    app_version: ${{ steps.version.outputs.version }}
```

### Conditional Stage

```yaml
stages:
- if: ${{ branch == "main" }}
  steps:
  - run: npm run deploy
```

## Steps

### Run Step (Multiple Syntaxes)

```yaml
# Shortest syntax
steps:
- echo "Hello"

# Short syntax
steps:
- run: echo "Hello"

# Long syntax
steps:
- run:
    script: echo "Hello"
```

### Run Step with Container

```yaml
steps:
- run:
    script: npm test
    container: node:18

# Container with full configuration
steps:
- run:
    script: npm test
    container:
      image: node:18
      user: root
      privileged: false
      pull: if-not-exists
      memory: 2gb
      cpu: 1
      env:
        CI: "true"
```

### Run Step with Shell

```yaml
steps:
- run:
    shell: bash  # sh, bash, powershell, pwsh, python
    script: |
      echo "Multi-line"
      echo "Script"
```

### Run Step with Environment Variables

```yaml
steps:
- run:
    script: echo $MY_VAR
    env:
      MY_VAR: "value"
      SECRET_KEY: ${{ secrets.API_KEY }}
```

### Parallel Steps

```yaml
steps:
- parallel:
    steps:
    - run: npm run lint
    - run: npm run test
    - run: npm run build
```

### Step Groups

```yaml
steps:
- group:
    steps:
    - run: npm ci
    - run: npm run build
    - run: npm run test
```

### Action Step

```yaml
# Harness action (native steps)
steps:
- action:
    uses: http
    with:
      url: https://api.example.com/webhook
      method: POST

# Docker build and push action
steps:
- action:
    uses: docker-build-push
    with:
      push: true
      repo: myorg/myapp
      tags: latest,${{ inputs.version }}

# Kubernetes actions
steps:
- action:
    uses: kubernetes-rolling-deploy
    with:
      dry-run: false
```

### Template Step

```yaml
steps:
- template:
    uses: account.my-template
    with:
      param1: value1
      param2: value2

# Template with version
steps:
- template:
    uses: account.my-template@1.0.0
    with:
      param1: value1
```

### Approval Step

```yaml
steps:
- approval:
    uses: harness
    with:
      timeout: 1h
      message: "Please approve"
      groups: ["approvers"]
      min-approvers: 1
```

### Barrier Step

```yaml
steps:
- barrier:
    name: sync-point
```

### Queue Step

```yaml
steps:
- queue:
    key: deploy-queue
    scope: pipeline  # or stage
```

### Clone Step

```yaml
steps:
- clone:
    repo: org/other-repo
    connector: github_connector
    depth: 1
    path: ./other-repo
```

### Background Step (Service)

```yaml
steps:
- background:
    script: docker run -p 5432:5432 postgres
    container: docker:dind

# Or as a stage-level service
stages:
- services:
    postgres:
      image: postgres:14
      env:
        POSTGRES_PASSWORD: password
  steps:
  - run: npm test
```

### Run Tests Step (Test Intelligence)

```yaml
steps:
- run-test:
    script: npm test
    container: node:18
    match: "**/*.test.js"
    report:
      type: junit
      path: junit.xml
    intelligence:
      disabled: false
    splitting:
      concurrency: 4
```

### Step with Timeout

```yaml
steps:
- run: npm test
  timeout: 10m
```

### Step with Failure Strategy

```yaml
steps:
- run: npm test
  on-failure:
    errors: [unknown, connectivity]
    action: retry
    retry:
      count: 3
      interval: 10s
```

### Conditional Step

```yaml
steps:
- if: ${{ branch == "main" }}
  run: npm run deploy

- if: ${{ failure() }}
  run: echo "Previous step failed"
```

### Step with Matrix

```yaml
steps:
- run: node --version
  container: node:${{ matrix.version }}
  strategy:
    matrix:
      version: ["16", "18", "20"]
```

## Inputs (Variables)

```yaml
pipeline:
  inputs:
    # String input
    version:
      type: string
      default: "1.0.0"

    # Required input
    environment:
      type: string
      required: true

    # Secret input
    api_key:
      type: secret

    # Enum input
    deploy_target:
      type: string
      enum:
        - staging
        - production

    # Pattern-validated input
    semver:
      type: string
      pattern: "^[0-9]+\\.[0-9]+\\.[0-9]+$"

    # Boolean input
    skip_tests:
      type: boolean
      default: false

    # Number input
    replicas:
      type: number
      default: 3

  stages:
  - steps:
    - run: echo "Deploying version ${{ inputs.version }}"
```

## Expressions

### Variable References

```yaml
# Input variables
${{ inputs.version }}

# Environment variables
${{ env.NODE_ENV }}

# Step outputs
${{ steps.step_id.outputs.variable }}

# Stage outputs
${{ stages.stage_id.outputs.variable }}

# Matrix values
${{ matrix.node }}
${{ matrix.os }}

# Loop iteration
${{ for.iteration }}

# Built-in context
${{ branch }}
${{ commit }}
${{ event }}
${{ repo }}
```

### Conditional Expressions

```yaml
# Simple condition
if: ${{ branch == "main" }}

# Multiple conditions
if: ${{ branch == "main" && event == "push" }}

# Status checks
if: ${{ success() }}
if: ${{ failure() }}
if: ${{ always() }}

# Contains check
if: ${{ contains(branch, "feature/") }}
```

## Triggers (on)

```yaml
# Single trigger
pipeline:
  on: push

# Multiple triggers
pipeline:
  on:
  - push
  - pull_request

# Triggers with filters
pipeline:
  on:
  - push:
      branches:
      - main
      - release/*
      paths:
      - src/**
      paths-ignore:
      - docs/**
  - pull_request:
      branches:
      - main
      types:
      - opened
      - synchronize
      review-approved: true

# Scheduled trigger
pipeline:
  on:
  - schedule:
      cron: "0 2 * * *"
```

## Looping Strategies

### Matrix

```yaml
strategy:
  matrix:
    node: ["16", "18", "20"]
    os: [ubuntu, macos]
  max-parallel: 4

# Matrix with exclude
strategy:
  matrix:
    node: ["16", "18", "20"]
    os: [ubuntu, macos]
    exclude:
      - node: "16"
        os: macos

# Matrix with include
strategy:
  matrix:
    include:
      - node: "20"
        os: ubuntu
        experimental: true
```

### For Loop

```yaml
strategy:
  for:
    iterations: 10
steps:
- run: echo "Iteration ${{ for.iteration }}"
```

### While Loop

```yaml
strategy:
  while:
    iterations: 10
    condition: ${{ status == "failure" }}
```

## Failure Strategies

```yaml
on-failure:
  errors: all  # or [unknown, connectivity, timeout]
  action: abort  # abort, ignore, retry, manual

# Retry configuration
on-failure:
  errors: all
  action: retry
  retry:
    count: 3
    interval: 10s
```

## Volumes

```yaml
stages:
- volumes:
  - name: cache
    path: /cache
    type: temp
  - name: config
    path: /config
    type: config-map
    config-map: my-config
  steps:
  - run: ls /cache
```

## GitHub Actions Compatibility

```yaml
# GitHub-compatible syntax
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm install
      - run: npm test

# Extended with Harness features
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm install
      - run: npm test
      - template:
          uses: account.docker-push
          with:
            repo: myorg/myapp
```

## Complete CI Pipeline Example

```yaml
pipeline:
  inputs:
    node_version:
      type: string
      default: "18"
    skip_tests:
      type: boolean
      default: false

  on:
  - push:
      branches: [main]
  - pull_request:
      branches: [main]

  stages:
  - id: build-and-test
    name: Build and Test
    runtime: cloud
    cache:
      path: node_modules
      key: npm-${{ hashFiles('package-lock.json') }}
    steps:
    - run: npm ci
    - parallel:
        steps:
        - run: npm run lint
        - if: ${{ !inputs.skip_tests }}
          run: npm test
    - run: npm run build
    - action:
        uses: docker-build-push
        with:
          repo: myorg/myapp
          tags: ${{ commit }}
    outputs:
      image_tag: ${{ commit }}
```

## Complete CD Pipeline Example

```yaml
pipeline:
  inputs:
    environment:
      type: string
      enum: [dev, staging, prod]
      required: true
    skip_approval:
      type: boolean
      default: false

  stages:
  - id: deploy
    service: my-service
    environment: ${{ inputs.environment }}
    steps:
    - action:
        uses: manifest-download
    - action:
        uses: kubernetes-rolling-deploy
        with:
          dry-run: true

  - if: ${{ inputs.environment == "prod" && !inputs.skip_approval }}
    approval:
      uses: harness
      with:
        timeout: 1h
        message: "Approve production deployment?"
        groups: [prod-approvers]
        min-approvers: 2

  - id: apply
    service: my-service
    environment: ${{ inputs.environment }}
    steps:
    - action:
        uses: kubernetes-rolling-deploy
        with:
          dry-run: false
    rollback:
    - action:
        uses: kubernetes-rollback
```

## Complete CI/CD Pipeline Example

```yaml
pipeline:
  inputs:
    version:
      type: string
      default: latest

  on:
  - push:
      branches: [main]

  env:
    DOCKER_REGISTRY: myregistry.io

  stages:
  # Build Stage
  - id: build
    name: Build
    runtime: cloud
    cache:
      path: node_modules
    steps:
    - run: npm ci
    - parallel:
        steps:
        - run: npm run lint
        - run: npm test
    - run: npm run build
    - action:
        uses: docker-build-push
        with:
          repo: ${{ env.DOCKER_REGISTRY }}/myapp
          tags: ${{ inputs.version }},${{ commit }}
    outputs:
      image: ${{ env.DOCKER_REGISTRY }}/myapp:${{ commit }}

  # Deploy to Staging
  - id: deploy-staging
    name: Deploy to Staging
    service: myapp
    environment: staging
    steps:
    - action:
        uses: kubernetes-rolling-deploy
        with:
          image: ${{ stages.build.outputs.image }}

  # Approval Gate
  - approval:
      uses: harness
      with:
        timeout: 24h
        message: "Approve production deployment?"
        groups: [prod-approvers]
        min-approvers: 1

  # Deploy to Production
  - id: deploy-prod
    name: Deploy to Production
    service: myapp
    environment: production
    steps:
    - action:
        uses: kubernetes-rolling-deploy
        with:
          image: ${{ stages.build.outputs.image }}
    rollback:
    - action:
        uses: kubernetes-rollback
    on-failure:
      errors: all
      action: abort
```

## Template Definition

```yaml
template:
  inputs:
    version:
      type: string
      default: latest
    registry:
      type: string
      required: true

  step:
    action:
      uses: docker-build-push
      with:
        repo: ${{ inputs.registry }}/myapp
        tags: ${{ inputs.version }}
```

## API Reference

### Create v1 Pipeline via API

**Endpoint:** `POST /v1/orgs/{org}/projects/{project}/pipelines`

```bash
curl -X POST 'https://app.harness.io/v1/orgs/default/projects/my_project/pipelines' \
  -H 'Content-Type: application/yaml' \
  -H 'x-api-key: YOUR_API_KEY' \
  -H 'Harness-Account: YOUR_ACCOUNT_ID' \
  -d 'pipeline:
  inputs:
    version:
      type: string
      default: "1.0.0"
  stages:
  - id: build
    runtime: cloud
    steps:
    - run: npm ci
    - run: npm test
    - run: npm run build'
```

### Update v1 Pipeline

**Endpoint:** `PUT /v1/orgs/{org}/projects/{project}/pipelines/{pipeline}`

```bash
curl -X PUT 'https://app.harness.io/v1/orgs/default/projects/my_project/pipelines/my_pipeline' \
  -H 'Content-Type: application/yaml' \
  -H 'x-api-key: YOUR_API_KEY' \
  -H 'Harness-Account: YOUR_ACCOUNT_ID' \
  -d '<updated_pipeline_yaml>'
```

### Get Pipeline

**Endpoint:** `GET /v1/orgs/{org}/projects/{project}/pipelines/{pipeline}`

```bash
curl -X GET 'https://app.harness.io/v1/orgs/default/projects/my_project/pipelines/my_pipeline' \
  -H 'x-api-key: YOUR_API_KEY' \
  -H 'Harness-Account: YOUR_ACCOUNT_ID'
```

### Execute Pipeline

**Endpoint:** `POST /v1/orgs/{org}/projects/{project}/pipelines/{pipeline}/execute`

```bash
curl -X POST 'https://app.harness.io/v1/orgs/default/projects/my_project/pipelines/my_pipeline/execute' \
  -H 'Content-Type: application/json' \
  -H 'x-api-key: YOUR_API_KEY' \
  -H 'Harness-Account: YOUR_ACCOUNT_ID' \
  -d '{
    "inputs": {
      "version": "2.0.0"
    }
  }'
```

## Error Handling

### Common Errors

| Error Code | Description | Solution |
|------------|-------------|----------|
| `INVALID_YAML` | YAML syntax error | Validate YAML syntax, check indentation |
| `INVALID_EXPRESSION` | Invalid `${{ }}` expression | Check expression syntax and variable names |
| `SERVICE_NOT_FOUND` | Referenced service doesn't exist | Create service or fix service reference |
| `ENVIRONMENT_NOT_FOUND` | Referenced environment doesn't exist | Create environment or fix reference |
| `ACTION_NOT_FOUND` | Unknown action in `uses:` | Check available actions, verify spelling |

### Expression Errors

```yaml
# Common expression mistakes:

# Wrong: Missing closing braces
${{ inputs.version }   # Error
${{ inputs.version }}  # Correct

# Wrong: Invalid context reference
${{ input.version }}   # Error (should be 'inputs')
${{ inputs.version }}  # Correct

# Wrong: Accessing undefined step output
${{ steps.unknown.outputs.value }}  # Error
${{ steps.build.outputs.value }}    # Correct (after step 'build' runs)
```

### Runtime Errors

```yaml
# Invalid runtime configuration
stages:
- runtime:
    kubernetes:
      # Missing required 'connector' field
      namespace: build-ns
  steps:
  - run: echo "test"

# Correct
stages:
- runtime:
    kubernetes:
      connector: k8s_connector
      namespace: build-ns
  steps:
  - run: echo "test"
```

## Troubleshooting

### Pipeline Won't Execute

1. **Check inputs are provided:**
   ```yaml
   inputs:
     version:
       type: string
       required: true  # Must be provided at execution
   ```

2. **Verify runtime configuration:**
   - For `runtime: cloud`, ensure Harness Cloud is enabled
   - For Kubernetes runtime, verify connector and namespace

3. **Check conditional expressions:**
   ```yaml
   if: ${{ branch == "main" }}  # Verify context is available
   ```

### Steps Not Running

1. **Check step conditions:**
   ```yaml
   - if: ${{ success() }}  # Only runs if previous steps succeeded
     run: echo "deploy"
   ```

2. **Verify parallel execution:**
   - Steps in `parallel:` run concurrently
   - Steps in `group:` run sequentially

### Caching Not Working

1. **Verify cache key:**
   ```yaml
   cache:
     path: node_modules
     key: npm-${{ hashFiles('package-lock.json') }}  # Ensure file exists
   ```

2. **Check cache path:**
   - Path must be relative to workspace
   - Directory must exist after step execution

### Matrix Strategy Issues

1. **Check matrix syntax:**
   ```yaml
   strategy:
     matrix:
       node: ["16", "18", "20"]  # Use proper array syntax
       os: [ubuntu-latest, macos-latest]
   ```

2. **Verify matrix references:**
   ```yaml
   - run: node --version
     container: node:${{ matrix.node }}  # Reference must match matrix key
   ```

### CD Stage Failures

1. **Verify service and environment exist:**
   - Service must be defined in Harness
   - Environment and infrastructure must be configured

2. **Check action availability:**
   ```yaml
   - action:
       uses: kubernetes-rolling-deploy  # Verify action name is correct
   ```

3. **Verify rollback steps:**
   ```yaml
   rollback:
   - action:
       uses: kubernetes-rollback  # Must match deployment type
   ```

## Instructions

When a user requests a v1 pipeline:

1. **Use the simplified syntax:**
   - Prefer short forms when possible (`run: echo` vs `run: { script: echo }`)
   - Use expression syntax: `${{ }}`
   - Leverage implicit features (auto-clone, caching intelligence)

2. **Match the use case:**
   - CI only: Focus on build/test steps with `runtime: cloud`
   - CD only: Include `service`, `environment`, and deployment actions
   - CI/CD: Combine both with proper stage separation

3. **Add appropriate features:**
   - Inputs for configurable values
   - Caching for build performance
   - Matrix for multi-version testing
   - Failure strategies for resilience

4. **Keep it concise:**
   - Only include fields that are needed
   - Use defaults where sensible
   - Avoid redundant configuration

5. **Output clean YAML** in a code block for easy copying.
