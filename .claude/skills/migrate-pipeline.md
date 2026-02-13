---
name: migrate-pipeline
description: Migrate Harness pipelines from v0 to v1 format. Reads existing pipelines via MCP and converts them to the simplified v1 syntax.
triggers:
  - migrate pipeline
  - convert pipeline
  - v0 to v1
  - upgrade pipeline
  - pipeline migration
  - modernize pipeline
---

# Migrate Pipeline Skill

Migrate Harness pipelines from v0 to v1 format using MCP tools to read existing pipelines.

## Overview

This skill helps teams modernize their pipelines by:
- Reading existing v0 pipeline definitions via MCP
- Converting to simplified v1 syntax
- Preserving all functionality
- Validating the migration
- Providing a diff/comparison

## Required MCP Tools

This skill requires the Harness MCP Server with these toolsets:
- `pipelines` - For reading existing pipeline definitions

### Tools Used
- `list_pipelines` - Find pipelines to migrate
- `get_pipeline` - Read v0 pipeline definition
- `get_pipeline_summary` - Get pipeline metadata

## v0 to v1 Conversion Guide

### Key Syntax Changes

| Feature | v0 Syntax | v1 Syntax |
|---------|-----------|-----------|
| Expressions | `<+variable>` | `${{ variable }}` |
| Run step | `type: Run` with `spec:` | `run: command` |
| Conditionals | `when:` block | `if: ${{ }}` |
| Stage definition | `stage:` with `type:` | Direct stage properties |
| Inputs | `variables:` with type | `inputs:` with type |
| Caching | `Cache` step | `cache:` at stage level |
| Triggers | Separate YAML | `on:` in pipeline |

### Expression Mapping

| v0 Expression | v1 Expression |
|---------------|---------------|
| `<+pipeline.name>` | `${{ pipeline.name }}` |
| `<+codebase.branch>` | `${{ branch }}` |
| `<+codebase.commitSha>` | `${{ commit }}` |
| `<+trigger.event>` | `${{ event }}` |
| `<+stage.name>` | `${{ stage.name }}` |
| `<+steps.stepId.output.var>` | `${{ steps.stepId.outputs.var }}` |
| `<+input>` | Use `inputs:` definition |
| `<+secrets.getValue("x")>` | `${{ secrets.x }}` |
| `<+env.VAR>` | `${{ env.VAR }}` |

## Workflow

### Step 1: Find Pipeline to Migrate

List available pipelines:

```
Use MCP tool: list_pipelines
Parameters:
  - org_id: <organization>
  - project_id: <project>
  - search_term: <optional filter>
```

### Step 2: Read v0 Pipeline

Get the full pipeline definition:

```
Use MCP tool: get_pipeline
Parameters:
  - pipeline_id: <pipeline identifier>
  - org_id: <organization>
  - project_id: <project>
```

### Step 3: Analyze Pipeline Structure

Identify components to migrate:
- Pipeline properties
- Variables/Inputs
- Stages and their types
- Steps within stages
- Conditions and expressions
- Failure strategies
- Notifications

### Step 4: Convert to v1 Format

Apply conversion rules (see detailed mapping below).

### Step 5: Validate Migration

Check for:
- All stages converted
- All steps converted
- Expressions updated
- Inputs properly defined
- No functionality lost

### Step 6: Present Both Versions

Show side-by-side comparison with changes highlighted.

## Detailed Conversion Rules

### Pipeline Root

**v0:**
```yaml
pipeline:
  name: My Pipeline
  identifier: my_pipeline
  projectIdentifier: my_project
  orgIdentifier: my_org
  tags: {}
  properties:
    ci:
      codebase:
        connectorRef: github
        repoName: org/repo
        build: <+input>
  stages:
    - stage:
        ...
```

**v1:**
```yaml
pipeline:
  inputs:
    # Converted from properties and variables

  repo:
    name: org/repo
    connector: github

  on:
    # Triggers if applicable

  stages:
    - # Direct stage definition
```

### CI Stage

**v0:**
```yaml
- stage:
    name: Build
    identifier: build
    type: CI
    spec:
      cloneCodebase: true
      infrastructure:
        type: KubernetesDirect
        spec:
          connectorRef: k8s_connector
          namespace: build
          os: Linux
      execution:
        steps:
          - step:
              type: Run
              name: Install
              identifier: install
              spec:
                shell: Sh
                command: npm install
          - step:
              type: Run
              name: Test
              identifier: test
              spec:
                shell: Sh
                command: npm test
```

**v1:**
```yaml
- id: build
  name: Build
  runtime:
    kubernetes:
      connector: k8s_connector
      namespace: build
  platform:
    os: linux
  steps:
    - run: npm install
    - name: Test
      run: npm test
```

### Run Step

**v0:**
```yaml
- step:
    type: Run
    name: Build
    identifier: build
    spec:
      shell: Bash
      command: |
        npm run build
      envVariables:
        NODE_ENV: production
      outputVariables:
        - name: BUILD_VERSION
```

**v1:**
```yaml
- id: build
  name: Build
  run:
    shell: bash
    script: |
      npm run build
    env:
      NODE_ENV: production
  outputs:
    BUILD_VERSION: ${{ steps.build.outputs.BUILD_VERSION }}
```

### Build and Push Step

**v0:**
```yaml
- step:
    type: BuildAndPushDockerRegistry
    name: Push to Docker
    identifier: push_docker
    spec:
      connectorRef: docker_hub
      repo: myorg/myapp
      tags:
        - latest
        - <+codebase.commitSha>
```

**v1:**
```yaml
- name: Push to Docker
  action:
    uses: docker-build-push
    with:
      connector: docker_hub
      repo: myorg/myapp
      tags: |
        latest
        ${{ commit }}
```

### Parallel Steps

**v0:**
```yaml
- parallel:
    - step:
        type: Run
        name: Lint
        identifier: lint
        spec:
          command: npm run lint
    - step:
        type: Run
        name: Test
        identifier: test
        spec:
          command: npm test
```

**v1:**
```yaml
- parallel:
    steps:
      - name: Lint
        run: npm run lint
      - name: Test
        run: npm test
```

### Conditions

**v0:**
```yaml
- step:
    type: Run
    name: Deploy
    identifier: deploy
    when:
      stageStatus: Success
      condition: <+codebase.branch> == "main"
    spec:
      command: ./deploy.sh
```

**v1:**
```yaml
- name: Deploy
  if: ${{ branch == "main" }}
  run: ./deploy.sh
```

### Deployment Stage

**v0:**
```yaml
- stage:
    name: Deploy to Prod
    identifier: deploy_prod
    type: Deployment
    spec:
      deploymentType: Kubernetes
      service:
        serviceRef: my_service
      environment:
        environmentRef: production
        infrastructureDefinitions:
          - identifier: k8s_prod
      execution:
        steps:
          - step:
              type: K8sRollingDeploy
              name: Rolling Deploy
              identifier: rolling
              spec:
                skipDryRun: false
```

**v1:**
```yaml
- id: deploy-prod
  name: Deploy to Prod
  service: my_service
  environment: production
  steps:
    - action:
        uses: kubernetes-rolling-deploy
        with:
          dry-run: false
```

### Approval Stage

**v0:**
```yaml
- stage:
    name: Approval
    identifier: approval
    type: Approval
    spec:
      execution:
        steps:
          - step:
              type: HarnessApproval
              name: Approve Prod
              identifier: approve
              spec:
                approvalMessage: Deploy to production?
                approvers:
                  userGroups:
                    - prod_approvers
                  minimumCount: 1
                timeout: 1d
```

**v1:**
```yaml
- approval:
    uses: harness
    with:
      message: Deploy to production?
      groups: [prod_approvers]
      min-approvers: 1
      timeout: 24h
```

### Variables to Inputs

**v0:**
```yaml
pipeline:
  variables:
    - name: version
      type: String
      default: latest
      value: <+input>
    - name: environment
      type: String
      value: staging
      allowedValues:
        - staging
        - production
```

**v1:**
```yaml
pipeline:
  inputs:
    version:
      type: string
      default: latest
    environment:
      type: string
      default: staging
      enum: [staging, production]
```

### Failure Strategy

**v0:**
```yaml
failureStrategies:
  - onFailure:
      errors:
        - AllErrors
      action:
        type: Retry
        spec:
          retryCount: 3
          retryIntervals:
            - 10s
```

**v1:**
```yaml
on-failure:
  errors: all
  action: retry
  retry:
    count: 3
    interval: 10s
```

## Response Format

### Migration Report

```markdown
## Pipeline Migration Report

**Pipeline:** build-and-deploy
**Project:** my-project
**Migration:** v0 → v1

### Summary

| Component | v0 Count | v1 Count | Status |
|-----------|----------|----------|--------|
| Stages | 4 | 4 | ✅ Converted |
| Steps | 12 | 12 | ✅ Converted |
| Variables | 3 | 3 | ✅ → Inputs |
| Conditions | 2 | 2 | ✅ Converted |
| Expressions | 15 | 15 | ✅ Updated |

### Changes Made

1. **Syntax Updates:**
   - 15 expressions converted (`<+...>` → `${{ }}`)
   - 12 steps simplified (removed `spec:` wrapper)
   - 4 stages restructured

2. **Feature Conversions:**
   - Variables → Inputs
   - `when:` → `if:`
   - Infrastructure → runtime/platform

3. **Preserved Features:**
   - Parallel execution
   - Failure strategies
   - Approval gates
   - Service/environment refs

### v1 Pipeline

```yaml
pipeline:
  inputs:
    version:
      type: string
      default: latest
    environment:
      type: string
      enum: [staging, production]

  stages:
    - id: build
      name: Build
      runtime: cloud
      cache:
        path: node_modules
        key: npm-${{ hashFiles('package-lock.json') }}
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
              repo: myorg/app
              tags: ${{ inputs.version }}
      outputs:
        image: myorg/app:${{ inputs.version }}

    - approval:
        uses: harness
        with:
          message: Deploy to ${{ inputs.environment }}?
          groups: [approvers]

    - id: deploy
      service: my-app
      environment: ${{ inputs.environment }}
      steps:
        - action:
            uses: kubernetes-rolling-deploy
```

### Validation

✅ All stages converted successfully
✅ All expressions updated
✅ Functionality preserved
✅ Ready for testing
```

## Common Scenarios

### 1. Migrate Single Pipeline

```
/migrate-pipeline

Migrate the build-and-deploy pipeline from v0 to v1
```

### 2. Preview Migration

```
/migrate-pipeline

Show me what the ci-pipeline would look like in v1 format
without actually changing anything
```

### 3. Complex Pipeline

```
/migrate-pipeline

Convert our main production pipeline with all its stages
and show me any features that need manual review
```

### 4. Bulk Assessment

```
/migrate-pipeline

Which pipelines in this project would benefit most
from migration to v1?
```

## Migration Considerations

### Features That Migrate Easily

- Run steps
- Parallel execution
- Basic conditionals
- Docker build/push
- Kubernetes deploy
- Approval gates
- Environment variables

### Features Requiring Review

- Custom steps with complex specs
- Plugin steps
- Template references
- Advanced failure strategies
- Custom delegates
- Complex expressions

### Features Not Yet in v1

Some v0 features may not have v1 equivalents yet:
- Certain deployment types
- Some specialized steps
- Advanced pipeline configurations

For these, note them in the migration report and suggest alternatives.

## Example Usage

### Quick Migration

```
/migrate-pipeline

Convert ci-pipeline to v1 format
```

### Detailed Analysis

```
/migrate-pipeline

Analyze the deploy-pipeline for v1 migration:
- Show all expression changes
- Highlight any complex conversions
- Note any features that need review
```

### Validate Only

```
/migrate-pipeline

Check if the staging-pipeline can be migrated to v1
without any issues
```

## Error Handling

### Common Errors

| Error Code | Description | Solution |
|------------|-------------|----------|
| `PIPELINE_NOT_FOUND` | Pipeline doesn't exist | Verify pipeline identifier and scope |
| `UNSUPPORTED_STEP_TYPE` | Step type not supported in v1 | Note for manual migration |
| `EXPRESSION_PARSE_ERROR` | Cannot parse v0 expression | Review expression syntax |
| `INVALID_YAML` | Generated YAML is invalid | Check for conversion errors |
| `FEATURE_NOT_AVAILABLE` | v1 doesn't support this feature | Document as limitation |

### Migration Errors

```
# Common migration issues:

# Unsupported step type
Warning: Step type 'CustomScript' has no v1 equivalent
→ Note for manual migration or use 'run' with custom logic

# Complex expression
Warning: Expression '<+matrix.strategy>' may need manual review
→ Complex expressions may need adjustment

# Plugin step
Warning: Plugin step 'my-plugin' requires verification
→ Check plugin compatibility with v1
```

## Troubleshooting

### Expression Conversion Issues

1. **Complex expressions:**
   - Some v0 expressions have different v1 syntax
   - Matrix/strategy expressions may differ
   - Nested expressions need careful handling

2. **Common expression patterns:**
   ```yaml
   # v0 → v1 conversion issues
   <+pipeline.variables.x> → ${{ inputs.x }}  # If variable is input
   <+matrix.index> → May need manual adjustment
   <+strategy.iteration> → May need manual adjustment
   ```

3. **Validation steps:**
   - Test expressions in v1 pipeline
   - Verify output values match
   - Check conditional logic works

### Step Type Not Supported

1. **Identify alternatives:**
   - Most steps have v1 equivalents
   - Custom steps may need 'run' with script
   - Plugin steps may need actions

2. **Manual conversion needed:**
   - Document unsupported steps
   - Provide suggested alternatives
   - Note any functionality gaps

3. **Partial migration:**
   - Convert supported portions
   - Flag steps needing attention
   - Provide migration notes

### YAML Validation Failures

1. **Syntax errors:**
   - Check indentation consistency
   - Verify YAML special characters escaped
   - Validate against v1 schema

2. **Schema violations:**
   - Ensure required fields present
   - Check field types match schema
   - Verify structure is correct

3. **Semantic issues:**
   - Stage references must exist
   - Input references must be defined
   - Service/environment refs valid

### Feature Gaps

1. **v1 limitations:**
   - Some v0 features not yet in v1
   - Document as migration blockers
   - Suggest workarounds if available

2. **Complex pipelines:**
   - Nested templates may need review
   - Advanced failure strategies differ
   - Custom delegates may need adjustment

3. **Mitigation strategies:**
   - Partial migration (some stages)
   - Hybrid approach (v0 + v1)
   - Wait for v1 feature parity

## Instructions

When migrating pipelines:

1. **Read the source pipeline:**
   - Use MCP to get full v0 definition
   - Understand all components

2. **Analyze complexity:**
   - Identify standard vs custom components
   - Note any features needing special handling

3. **Apply conversions:**
   - Update all expressions
   - Convert all step types
   - Restructure stages

4. **Validate result:**
   - Ensure all functionality preserved
   - Check for missing conversions
   - Verify expression correctness

5. **Present clearly:**
   - Show both versions
   - Highlight changes
   - Note any manual review needed
