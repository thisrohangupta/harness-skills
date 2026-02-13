---
name: create-template
description: Generate Harness.io v0 Template YAML files for reusable pipeline components and optionally create them via the Harness API. Use when the user wants to create a Harness template, step template, stage template, pipeline template, or stepgroup template.
triggers:
  - harness template
  - create template
  - step template
  - stage template
  - pipeline template
  - stepgroup template
  - reusable step
  - reusable stage
  - create template api
---

# Create Template Skill

Generate Harness.io v0 Template YAML files for reusable pipeline components.

## Overview

This skill creates valid Harness template YAML configurations following the v0 schema specification. Templates allow you to create reusable components that can be shared across pipelines, promoting consistency and reducing duplication.

## Schema Reference

Schema source: https://github.com/harness/harness-schema/tree/main/v0/template

## Template Types

Harness supports the following template types:

| Type | Description | Use Case |
|------|-------------|----------|
| `Step` | Reusable step definition | Common build/deploy steps |
| `Stage` | Reusable stage definition | Standard CI/CD stages |
| `Pipeline` | Reusable pipeline definition | Complete workflow templates |
| `StepGroup` | Reusable step group | Related steps bundled together |
| `SecretManager` | Secret manager configuration | Custom secret backends |
| `CustomDeployment` | Custom deployment logic | Non-standard deployments |
| `ArtifactSource` | Artifact source configuration | Custom artifact locations |

## Common Template Structure

All templates share this base structure:

```yaml
template:
  identifier: <unique_identifier>    # Pattern: ^[a-zA-Z_][0-9a-zA-Z_]{0,127}$
  name: <display_name>               # Pattern: ^[a-zA-Z_0-9-.][-0-9a-zA-Z_\s.]{0,127}$
  type: <template_type>              # Step, Stage, Pipeline, StepGroup
  versionLabel: <version>            # Pattern: ^[0-9a-zA-Z][^\s/&]{0,127}$
  description: <description>         # Optional
  tags: {}                           # Optional: key-value pairs
  icon: <icon_url>                   # Optional: custom icon
  variables:                         # Optional: template inputs
    - name: varName
      type: String
      description: "Variable description"
      default: "default_value"
  spec:
    # Template-specific content
```

## Step Template

Create reusable step definitions.

### Basic Step Template

```yaml
template:
  identifier: run_npm_tests
  name: Run NPM Tests
  type: Step
  versionLabel: "1.0.0"
  description: "Standard NPM test execution step"
  tags:
    category: testing
    language: nodejs
  variables:
    - name: test_command
      type: String
      description: "NPM test command to run"
      default: "test"
    - name: coverage_enabled
      type: String
      description: "Enable coverage reporting"
      default: "true"
  spec:
    type: Run
    spec:
      shell: Bash
      command: |
        npm run <+spec.variables.test_command>
        <+<+spec.variables.coverage_enabled> == "true" ? "npm run coverage" : "">
      reports:
        type: JUnit
        spec:
          paths:
            - "junit.xml"
      timeout: 10m
```

### Docker Build Step Template

```yaml
template:
  identifier: docker_build_push
  name: Docker Build and Push
  type: Step
  versionLabel: "1.0.0"
  description: "Build and push Docker image to registry"
  variables:
    - name: docker_connector
      type: String
      description: "Docker registry connector reference"
    - name: image_name
      type: String
      description: "Docker image name"
    - name: dockerfile_path
      type: String
      description: "Path to Dockerfile"
      default: "Dockerfile"
    - name: context
      type: String
      description: "Docker build context"
      default: "."
  spec:
    type: BuildAndPushDockerRegistry
    spec:
      connectorRef: <+spec.variables.docker_connector>
      repo: <+spec.variables.image_name>
      tags:
        - <+pipeline.sequenceId>
        - latest
      dockerfile: <+spec.variables.dockerfile_path>
      context: <+spec.variables.context>
      resources:
        limits:
          memory: 2Gi
          cpu: 1000m
    timeout: 20m
```

### Shell Script Step Template

```yaml
template:
  identifier: shell_script_runner
  name: Shell Script Runner
  type: Step
  versionLabel: "1.0.0"
  description: "Execute shell script with configurable options"
  variables:
    - name: script_content
      type: String
      description: "Shell script to execute"
    - name: shell_type
      type: String
      description: "Shell type (Bash, Sh, Powershell)"
      default: "Bash"
    - name: working_directory
      type: String
      description: "Working directory for script execution"
      default: "."
    - name: timeout_minutes
      type: Number
      description: "Execution timeout in minutes"
      default: 10
  spec:
    type: ShellScript
    spec:
      shell: <+spec.variables.shell_type>
      source:
        type: Inline
        spec:
          script: <+spec.variables.script_content>
      onDelegate: true
      executionTarget: {}
    timeout: <+spec.variables.timeout_minutes>m
```

### Kubernetes Deploy Step Template

```yaml
template:
  identifier: k8s_rolling_deploy
  name: Kubernetes Rolling Deploy
  type: Step
  versionLabel: "1.0.0"
  description: "Standard Kubernetes rolling deployment"
  variables:
    - name: skip_dry_run
      type: String
      description: "Skip dry run validation"
      default: "false"
    - name: prune_enabled
      type: String
      description: "Enable resource pruning"
      default: "false"
  spec:
    type: K8sRollingDeploy
    spec:
      skipDryRun: <+spec.variables.skip_dry_run>
      pruningEnabled: <+spec.variables.prune_enabled>
    timeout: 10m
    failureStrategies:
      - onFailure:
          errors:
            - AllErrors
          action:
            type: StageRollback
```

### HTTP Request Step Template

```yaml
template:
  identifier: http_health_check
  name: HTTP Health Check
  type: Step
  versionLabel: "1.0.0"
  description: "Perform HTTP health check on endpoint"
  variables:
    - name: endpoint_url
      type: String
      description: "URL to check"
    - name: expected_status
      type: Number
      description: "Expected HTTP status code"
      default: 200
    - name: method
      type: String
      description: "HTTP method"
      default: "GET"
  spec:
    type: Http
    spec:
      url: <+spec.variables.endpoint_url>
      method: <+spec.variables.method>
      assertion: <+httpResponseCode> == <+spec.variables.expected_status>
      headers: []
    timeout: 1m
```

### Terraform Step Template

```yaml
template:
  identifier: terraform_apply
  name: Terraform Apply
  type: Step
  versionLabel: "1.0.0"
  description: "Apply Terraform configuration"
  variables:
    - name: git_connector
      type: String
      description: "Git connector for Terraform files"
    - name: repo_name
      type: String
      description: "Repository containing Terraform files"
    - name: branch
      type: String
      description: "Git branch"
      default: "main"
    - name: folder_path
      type: String
      description: "Path to Terraform files"
      default: "terraform/"
    - name: provisioner_id
      type: String
      description: "Unique provisioner identifier"
  spec:
    type: TerraformApply
    spec:
      configuration:
        type: Inline
        spec:
          configFiles:
            store:
              type: Github
              spec:
                connectorRef: <+spec.variables.git_connector>
                repoName: <+spec.variables.repo_name>
                branch: <+spec.variables.branch>
                folderPath: <+spec.variables.folder_path>
      provisionerIdentifier: <+spec.variables.provisioner_id>
    timeout: 20m
```

### Approval Step Template

```yaml
template:
  identifier: manual_approval
  name: Manual Approval
  type: Step
  versionLabel: "1.0.0"
  description: "Standard manual approval gate"
  variables:
    - name: approval_message
      type: String
      description: "Message shown to approvers"
      default: "Please review and approve this deployment"
    - name: approver_groups
      type: String
      description: "Comma-separated list of approver user groups"
    - name: min_approvals
      type: Number
      description: "Minimum number of approvals required"
      default: 1
    - name: timeout_hours
      type: Number
      description: "Approval timeout in hours"
      default: 24
  spec:
    type: HarnessApproval
    spec:
      approvalMessage: <+spec.variables.approval_message>
      approvers:
        userGroups: <+spec.variables.approver_groups>.split(",")
        minimumCount: <+spec.variables.min_approvals>
      includePipelineExecutionHistory: true
    timeout: <+spec.variables.timeout_hours>h
```

## Stage Template

Create reusable stage definitions.

### CI Stage Template

```yaml
template:
  identifier: nodejs_ci_stage
  name: Node.js CI Stage
  type: Stage
  versionLabel: "1.0.0"
  description: "Standard Node.js CI stage with build and test"
  tags:
    language: nodejs
    type: ci
  variables:
    - name: node_version
      type: String
      description: "Node.js version to use"
      default: "18"
    - name: test_command
      type: String
      description: "Test command"
      default: "npm test"
    - name: build_command
      type: String
      description: "Build command"
      default: "npm run build"
  spec:
    type: CI
    spec:
      cloneCodebase: true
      platform:
        os: Linux
        arch: Amd64
      runtime:
        type: Cloud
        spec: {}
      execution:
        steps:
          - step:
              identifier: install
              name: Install Dependencies
              type: Run
              spec:
                shell: Bash
                command: |
                  nvm use <+stage.variables.node_version> || nvm install <+stage.variables.node_version>
                  npm ci
          - parallel:
              - step:
                  identifier: lint
                  name: Lint
                  type: Run
                  spec:
                    shell: Bash
                    command: npm run lint
              - step:
                  identifier: test
                  name: Test
                  type: Run
                  spec:
                    shell: Bash
                    command: <+stage.variables.test_command>
                    reports:
                      type: JUnit
                      spec:
                        paths:
                          - "junit.xml"
          - step:
              identifier: build
              name: Build
              type: Run
              spec:
                shell: Bash
                command: <+stage.variables.build_command>
    failureStrategies:
      - onFailure:
          errors:
            - AllErrors
          action:
            type: MarkAsFailure
```

### CD Stage Template

```yaml
template:
  identifier: k8s_deploy_stage
  name: Kubernetes Deploy Stage
  type: Stage
  versionLabel: "1.0.0"
  description: "Standard Kubernetes deployment stage"
  tags:
    platform: kubernetes
    type: cd
  variables:
    - name: service_ref
      type: String
      description: "Harness service reference"
    - name: environment_ref
      type: String
      description: "Harness environment reference"
    - name: infra_ref
      type: String
      description: "Infrastructure definition reference"
  spec:
    type: Deployment
    spec:
      deploymentType: Kubernetes
      service:
        serviceRef: <+stage.variables.service_ref>
      environment:
        environmentRef: <+stage.variables.environment_ref>
        infrastructureDefinitions:
          - identifier: <+stage.variables.infra_ref>
      execution:
        steps:
          - step:
              identifier: rollout
              name: Rollout Deployment
              type: K8sRollingDeploy
              spec:
                skipDryRun: false
              timeout: 10m
        rollbackSteps:
          - step:
              identifier: rollback
              name: Rollback
              type: K8sRollingRollback
              spec: {}
              timeout: 10m
    failureStrategies:
      - onFailure:
          errors:
            - AllErrors
          action:
            type: StageRollback
```

### Approval Stage Template

```yaml
template:
  identifier: approval_stage
  name: Approval Stage
  type: Stage
  versionLabel: "1.0.0"
  description: "Standard approval stage with configurable approvers"
  variables:
    - name: stage_name
      type: String
      description: "Display name for the approval stage"
      default: "Approval"
    - name: approval_message
      type: String
      description: "Approval message"
    - name: approver_groups
      type: String
      description: "Approver user groups"
    - name: timeout_hours
      type: Number
      description: "Timeout in hours"
      default: 24
  spec:
    type: Approval
    spec:
      execution:
        steps:
          - step:
              identifier: approve
              name: <+stage.variables.stage_name>
              type: HarnessApproval
              spec:
                approvalMessage: <+stage.variables.approval_message>
                approvers:
                  userGroups:
                    - <+stage.variables.approver_groups>
                  minimumCount: 1
                includePipelineExecutionHistory: true
              timeout: <+stage.variables.timeout_hours>h
```

### Custom Stage Template

```yaml
template:
  identifier: custom_workflow_stage
  name: Custom Workflow Stage
  type: Stage
  versionLabel: "1.0.0"
  description: "Custom stage for arbitrary workflows"
  variables:
    - name: script
      type: String
      description: "Script to execute"
    - name: delegate_selector
      type: String
      description: "Delegate selector for execution"
      default: ""
  spec:
    type: Custom
    spec:
      execution:
        steps:
          - step:
              identifier: run_script
              name: Run Script
              type: ShellScript
              spec:
                shell: Bash
                source:
                  type: Inline
                  spec:
                    script: <+stage.variables.script>
                onDelegate: true
              timeout: 30m
    when:
      pipelineStatus: Success
```

## StepGroup Template

Create reusable groups of related steps.

### Test Suite StepGroup Template

```yaml
template:
  identifier: test_suite
  name: Test Suite
  type: StepGroup
  versionLabel: "1.0.0"
  description: "Complete test suite with unit, integration, and e2e tests"
  variables:
    - name: run_unit
      type: String
      description: "Run unit tests"
      default: "true"
    - name: run_integration
      type: String
      description: "Run integration tests"
      default: "true"
    - name: run_e2e
      type: String
      description: "Run e2e tests"
      default: "false"
  spec:
    steps:
      - parallel:
          - step:
              identifier: unit_tests
              name: Unit Tests
              type: Run
              spec:
                shell: Bash
                command: npm run test:unit
              when:
                stageStatus: Success
                condition: <+stepGroup.variables.run_unit> == "true"
          - step:
              identifier: integration_tests
              name: Integration Tests
              type: Run
              spec:
                shell: Bash
                command: npm run test:integration
              when:
                stageStatus: Success
                condition: <+stepGroup.variables.run_integration> == "true"
      - step:
          identifier: e2e_tests
          name: E2E Tests
          type: Run
          spec:
            shell: Bash
            command: npm run test:e2e
          when:
            stageStatus: Success
            condition: <+stepGroup.variables.run_e2e> == "true"
          timeout: 30m
```

### Security Scan StepGroup Template

```yaml
template:
  identifier: security_scans
  name: Security Scans
  type: StepGroup
  versionLabel: "1.0.0"
  description: "Security scanning suite"
  variables:
    - name: fail_on_critical
      type: String
      description: "Fail pipeline on critical vulnerabilities"
      default: "true"
  spec:
    steps:
      - parallel:
          - step:
              identifier: sast_scan
              name: SAST Scan
              type: Security
              spec:
                privileged: true
                settings:
                  product_name: semgrep
                  product_config_name: default
                  policy_type: orchestratedScan
                  scan_type: repository
                  repository_project: <+pipeline.name>
                  repository_branch: <+codebase.branch>
              timeout: 15m
          - step:
              identifier: sca_scan
              name: SCA Scan
              type: Security
              spec:
                privileged: true
                settings:
                  product_name: snyk
                  product_config_name: default
                  policy_type: orchestratedScan
                  scan_type: repository
              timeout: 15m
      - step:
          identifier: container_scan
          name: Container Scan
          type: AquaTrivy
          spec:
            mode: orchestration
            config: default
            target:
              type: container
              detection: auto
            advanced:
              log:
                level: info
            privileged: true
          timeout: 15m
```

### Build and Push StepGroup Template

```yaml
template:
  identifier: build_and_push
  name: Build and Push
  type: StepGroup
  versionLabel: "1.0.0"
  description: "Build application and push Docker image"
  variables:
    - name: build_command
      type: String
      description: "Build command"
      default: "npm run build"
    - name: docker_connector
      type: String
      description: "Docker registry connector"
    - name: image_repo
      type: String
      description: "Docker image repository"
    - name: dockerfile
      type: String
      description: "Dockerfile path"
      default: "Dockerfile"
  spec:
    steps:
      - step:
          identifier: build
          name: Build Application
          type: Run
          spec:
            shell: Bash
            command: <+stepGroup.variables.build_command>
      - step:
          identifier: docker_build_push
          name: Build and Push Image
          type: BuildAndPushDockerRegistry
          spec:
            connectorRef: <+stepGroup.variables.docker_connector>
            repo: <+stepGroup.variables.image_repo>
            tags:
              - <+pipeline.sequenceId>
              - <+codebase.commitSha>
              - latest
            dockerfile: <+stepGroup.variables.dockerfile>
            context: .
          timeout: 20m
```

## Pipeline Template

Create reusable complete pipeline definitions.

### Standard CI/CD Pipeline Template

```yaml
template:
  identifier: standard_cicd
  name: Standard CI/CD Pipeline
  type: Pipeline
  versionLabel: "1.0.0"
  description: "Standard CI/CD pipeline with build, test, and deploy"
  tags:
    type: cicd
  variables:
    - name: service_name
      type: String
      description: "Service name"
    - name: git_connector
      type: String
      description: "Git connector reference"
    - name: repo_name
      type: String
      description: "Repository name"
    - name: docker_connector
      type: String
      description: "Docker registry connector"
    - name: k8s_service_ref
      type: String
      description: "Harness service reference"
    - name: dev_env_ref
      type: String
      description: "Dev environment reference"
    - name: prod_env_ref
      type: String
      description: "Production environment reference"
  spec:
    properties:
      ci:
        codebase:
          connectorRef: <+pipeline.variables.git_connector>
          repoName: <+pipeline.variables.repo_name>
          build:
            type: branch
            spec:
              branch: <+trigger.branch>
    stages:
      - stage:
          identifier: build
          name: Build and Test
          type: CI
          spec:
            cloneCodebase: true
            platform:
              os: Linux
              arch: Amd64
            runtime:
              type: Cloud
              spec: {}
            execution:
              steps:
                - step:
                    identifier: build
                    name: Build
                    type: Run
                    spec:
                      shell: Bash
                      command: |
                        npm ci
                        npm run build
                - step:
                    identifier: test
                    name: Test
                    type: Run
                    spec:
                      shell: Bash
                      command: npm test
                - step:
                    identifier: push
                    name: Push Image
                    type: BuildAndPushDockerRegistry
                    spec:
                      connectorRef: <+pipeline.variables.docker_connector>
                      repo: <+pipeline.variables.service_name>
                      tags:
                        - <+pipeline.sequenceId>
      - stage:
          identifier: deploy_dev
          name: Deploy to Dev
          type: Deployment
          spec:
            deploymentType: Kubernetes
            service:
              serviceRef: <+pipeline.variables.k8s_service_ref>
            environment:
              environmentRef: <+pipeline.variables.dev_env_ref>
              infrastructureDefinitions:
                - identifier: k8s_dev
            execution:
              steps:
                - step:
                    identifier: rollout
                    name: Rollout
                    type: K8sRollingDeploy
                    spec:
                      skipDryRun: false
              rollbackSteps:
                - step:
                    identifier: rollback
                    name: Rollback
                    type: K8sRollingRollback
                    spec: {}
          failureStrategies:
            - onFailure:
                errors:
                  - AllErrors
                action:
                  type: StageRollback
      - stage:
          identifier: approval
          name: Production Approval
          type: Approval
          spec:
            execution:
              steps:
                - step:
                    identifier: approve
                    name: Approve
                    type: HarnessApproval
                    spec:
                      approvalMessage: "Approve production deployment?"
                      approvers:
                        userGroups:
                          - prod_approvers
                        minimumCount: 1
                    timeout: 24h
      - stage:
          identifier: deploy_prod
          name: Deploy to Production
          type: Deployment
          spec:
            deploymentType: Kubernetes
            service:
              serviceRef: <+pipeline.variables.k8s_service_ref>
            environment:
              environmentRef: <+pipeline.variables.prod_env_ref>
              infrastructureDefinitions:
                - identifier: k8s_prod
            execution:
              steps:
                - step:
                    identifier: rollout
                    name: Rollout
                    type: K8sRollingDeploy
                    spec:
                      skipDryRun: false
              rollbackSteps:
                - step:
                    identifier: rollback
                    name: Rollback
                    type: K8sRollingRollback
                    spec: {}
          failureStrategies:
            - onFailure:
                errors:
                  - AllErrors
                action:
                  type: StageRollback
```

## Using Templates in Pipelines

### Reference a Step Template

```yaml
- step:
    identifier: my_step
    name: My Step
    template:
      templateRef: run_npm_tests
      versionLabel: "1.0.0"
      templateInputs:
        spec:
          variables:
            test_command: "test:coverage"
            coverage_enabled: "true"
```

### Reference a Stage Template

```yaml
- stage:
    identifier: build
    name: Build
    template:
      templateRef: nodejs_ci_stage
      versionLabel: "1.0.0"
      templateInputs:
        spec:
          variables:
            node_version: "20"
            test_command: "npm run test:ci"
```

### Reference a Pipeline Template

```yaml
pipeline:
  identifier: my_pipeline
  name: My Pipeline
  template:
    templateRef: standard_cicd
    versionLabel: "1.0.0"
    templateInputs:
      variables:
        service_name: my-service
        git_connector: github_connector
        repo_name: my-repo
```

## Template Variables

### Variable Types

```yaml
variables:
  # String variable
  - name: environment
    type: String
    description: "Target environment"
    default: "dev"

  # Number variable
  - name: replicas
    type: Number
    description: "Number of replicas"
    default: 3

  # Secret variable
  - name: api_key
    type: Secret
    description: "API key for external service"
```

### Referencing Variables

```yaml
# In Step templates
<+spec.variables.varName>

# In Stage templates
<+stage.variables.varName>

# In StepGroup templates
<+stepGroup.variables.varName>

# In Pipeline templates
<+pipeline.variables.varName>
```

## Template Inputs

When using templates, you can make certain fields configurable as runtime inputs:

```yaml
template:
  identifier: configurable_deploy
  name: Configurable Deploy
  type: Step
  versionLabel: "1.0.0"
  spec:
    type: K8sRollingDeploy
    spec:
      skipDryRun: <+input>  # Runtime input
    timeout: <+input>.default(10m).allowedValues(5m,10m,15m,30m)  # With constraints
```

## Template Scope

Templates can be created at different scopes:

- **Account Level**: Available to all organizations and projects
- **Organization Level**: Available to all projects in the organization
- **Project Level**: Available only within the project

Reference templates by scope:

```yaml
# Account level template
templateRef: account.my_template

# Organization level template
templateRef: org.my_template

# Project level template (default)
templateRef: my_template
```

## Best Practices

1. **Version your templates** - Use semantic versioning (1.0.0, 1.1.0, 2.0.0)

2. **Add descriptions** - Document what the template does and its variables

3. **Use meaningful variable names** - Make variables self-documenting

4. **Set sensible defaults** - Provide defaults for optional parameters

5. **Add tags** - Categorize templates for easy discovery

6. **Test templates** - Verify templates work before publishing

7. **Keep templates focused** - One template should do one thing well

## Creating Templates via API

After generating the template YAML, you can create it directly in Harness using the API.

### API Reference

**Endpoint:** `POST /template/api/templates`
**Documentation:** https://apidocs.harness.io/templates/createtemplate

### Request Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Content-Type` | Yes | Must be `application/yaml` |
| `x-api-key` | Yes | Harness API key |

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `accountIdentifier` | string | Yes | Account ID |
| `orgIdentifier` | string | No | Organization ID (for org/project scope) |
| `projectIdentifier` | string | No | Project ID (for project scope) |
| `storeType` | string | No | `INLINE` (default) or `REMOTE` |
| `isNewTemplate` | boolean | No | `true` for new template, `false` for update |
| `setDefaultTemplate` | boolean | No | Set as default/stable version |
| `comments` | string | No | Version comments |

### Git Storage Parameters (for REMOTE storeType)

| Parameter | Type | Description |
|-----------|------|-------------|
| `connectorRef` | string | Git connector reference |
| `repoName` | string | Repository name |
| `branch` | string | Branch name |
| `filePath` | string | File path in repo |
| `commitMsg` | string | Commit message |
| `isNewBranch` | boolean | Create new branch |
| `baseBranch` | string | Base branch for new branch |

### Example: Create Account-Level Template (Inline)

```bash
curl -X POST \
  'https://app.harness.io/template/api/templates?accountIdentifier=ACCOUNT_ID&storeType=INLINE&isNewTemplate=true&comments=Initial%20version' \
  -H 'Content-Type: application/yaml' \
  -H 'x-api-key: YOUR_API_KEY' \
  -d 'template:
  identifier: docker_build_push
  name: Docker Build and Push
  type: Step
  versionLabel: "1.0.0"
  description: "Build and push Docker image to registry"
  tags:
    category: build
  variables:
    - name: docker_connector
      type: String
      description: "Docker registry connector reference"
    - name: image_name
      type: String
      description: "Docker image name"
  spec:
    type: BuildAndPushDockerRegistry
    spec:
      connectorRef: <+spec.variables.docker_connector>
      repo: <+spec.variables.image_name>
      tags:
        - <+pipeline.sequenceId>
        - latest
    timeout: 20m'
```

### Example: Create Org-Level Template

```bash
curl -X POST \
  'https://app.harness.io/template/api/templates?accountIdentifier=ACCOUNT_ID&orgIdentifier=ORG_ID&storeType=INLINE&isNewTemplate=true' \
  -H 'Content-Type: application/yaml' \
  -H 'x-api-key: YOUR_API_KEY' \
  -d 'template:
  identifier: k8s_deploy_stage
  name: Kubernetes Deploy Stage
  type: Stage
  versionLabel: "1.0.0"
  orgIdentifier: default
  description: "Standard Kubernetes deployment stage"
  spec:
    type: Deployment
    spec:
      deploymentType: Kubernetes
      service:
        serviceRef: <+input>
      environment:
        environmentRef: <+input>
        infrastructureDefinitions: <+input>
      execution:
        steps:
          - step:
              identifier: rollout
              name: Rollout Deployment
              type: K8sRollingDeploy
              spec:
                skipDryRun: false
              timeout: 10m
        rollbackSteps:
          - step:
              identifier: rollback
              name: Rollback
              type: K8sRollingRollback
              spec: {}
    failureStrategies:
      - onFailure:
          errors:
            - AllErrors
          action:
            type: StageRollback'
```

### Example: Create Project-Level Template

```bash
curl -X POST \
  'https://app.harness.io/template/api/templates?accountIdentifier=ACCOUNT_ID&orgIdentifier=ORG_ID&projectIdentifier=PROJECT_ID&storeType=INLINE&isNewTemplate=true' \
  -H 'Content-Type: application/yaml' \
  -H 'x-api-key: YOUR_API_KEY' \
  -d 'template:
  identifier: ci_stage_nodejs
  name: Node.js CI Stage
  type: Stage
  versionLabel: "1.0.0"
  projectIdentifier: my_project
  orgIdentifier: default
  description: "CI stage for Node.js applications"
  spec:
    type: CI
    spec:
      cloneCodebase: true
      platform:
        os: Linux
        arch: Amd64
      runtime:
        type: Cloud
        spec: {}
      execution:
        steps:
          - step:
              identifier: install
              name: Install Dependencies
              type: Run
              spec:
                shell: Bash
                command: npm ci
          - step:
              identifier: test
              name: Run Tests
              type: Run
              spec:
                shell: Bash
                command: npm test'
```

### Example: Create Pipeline Template

```bash
curl -X POST \
  'https://app.harness.io/template/api/templates?accountIdentifier=ACCOUNT_ID&orgIdentifier=ORG_ID&projectIdentifier=PROJECT_ID&storeType=INLINE&isNewTemplate=true' \
  -H 'Content-Type: application/yaml' \
  -H 'x-api-key: YOUR_API_KEY' \
  -d 'template:
  identifier: pipelineTemplate
  name: Standard Pipeline Template
  type: Pipeline
  versionLabel: v1
  projectIdentifier: PROJECT_ID
  orgIdentifier: ORG_ID
  tags: {}
  spec:
    stages:
      - stage:
          identifier: stage1
          name: Build Stage
          type: Deployment
          spec:
            deploymentType: Kubernetes
            service:
              serviceRef: <+input>
              serviceInputs: <+input>
            environment:
              environmentRef: <+input>
              deployToAll: false
              environmentInputs: <+input>
              infrastructureDefinitions: <+input>
            execution:
              steps:
                - step:
                    type: ShellScript
                    name: Shell Script
                    identifier: ShellScript_1
                    spec:
                      shell: Bash
                      onDelegate: true
                      source:
                        type: Inline
                        spec:
                          script: <+input>
                      environmentVariables: []
                      outputVariables: []
                    timeout: 10m
              rollbackSteps: []
          tags: {}
          failureStrategies:
            - onFailure:
                errors:
                  - AllErrors
                action:
                  type: StageRollback'
```

### Example: Create Template in Git (Remote Storage)

```bash
curl -X POST \
  'https://app.harness.io/template/api/templates?accountIdentifier=ACCOUNT_ID&orgIdentifier=ORG_ID&projectIdentifier=PROJECT_ID&storeType=REMOTE&connectorRef=github_connector&repoName=harness-templates&branch=main&filePath=.harness/templates/docker-build.yaml&commitMsg=Add%20docker%20build%20template&isNewTemplate=true' \
  -H 'Content-Type: application/yaml' \
  -H 'x-api-key: YOUR_API_KEY' \
  -d 'template:
  identifier: docker_build_push
  name: Docker Build and Push
  type: Step
  versionLabel: "1.0.0"
  spec:
    type: BuildAndPushDockerRegistry
    spec:
      connectorRef: <+input>
      repo: <+input>
      tags:
        - <+pipeline.sequenceId>
    timeout: 20m'
```

### Example: Update Existing Template (New Version)

```bash
curl -X POST \
  'https://app.harness.io/template/api/templates?accountIdentifier=ACCOUNT_ID&storeType=INLINE&isNewTemplate=false&comments=Added%20caching%20support' \
  -H 'Content-Type: application/yaml' \
  -H 'x-api-key: YOUR_API_KEY' \
  -d 'template:
  identifier: docker_build_push
  name: Docker Build and Push
  type: Step
  versionLabel: "1.1.0"
  description: "Build and push Docker image with caching"
  spec:
    type: BuildAndPushDockerRegistry
    spec:
      connectorRef: <+input>
      repo: <+input>
      tags:
        - <+pipeline.sequenceId>
      caching: true
    timeout: 20m'
```

### API Response

**Success Response (200):**

```json
{
  "status": "SUCCESS",
  "data": {
    "accountId": "ACCOUNT_ID",
    "orgIdentifier": "ORG_ID",
    "projectIdentifier": "PROJECT_ID",
    "identifier": "docker_build_push",
    "name": "Docker Build and Push",
    "description": "Build and push Docker image to registry",
    "tags": {
      "category": "build"
    },
    "yaml": "template:\n  identifier: docker_build_push\n  ...",
    "versionLabel": "1.0.0",
    "templateEntityType": "Step",
    "templateScope": "project",
    "version": 0,
    "gitDetails": null,
    "entityValidityDetails": {
      "valid": true
    },
    "lastUpdatedAt": 1705320000000,
    "storeType": "INLINE",
    "stableTemplate": false
  }
}
```

**Error Response (400):**

```json
{
  "status": "ERROR",
  "code": "INVALID_REQUEST",
  "message": "Template with identifier [docker_build_push] and version [1.0.0] already exists",
  "correlationId": "abc123"
}
```

### Setting Stable Version

After creating a template, you can set it as the stable (default) version:

```bash
curl -X POST \
  'https://app.harness.io/template/api/templates?accountIdentifier=ACCOUNT_ID&storeType=INLINE&isNewTemplate=false&setDefaultTemplate=true' \
  -H 'Content-Type: application/yaml' \
  -H 'x-api-key: YOUR_API_KEY' \
  -d 'template:
  identifier: docker_build_push
  name: Docker Build and Push
  type: Step
  versionLabel: "1.0.0"
  spec:
    ...'
```

### Common API Errors

| Error | Cause | Solution |
|-------|-------|----------|
| Template already exists | Duplicate identifier + version | Use new version or update existing |
| Invalid YAML | Syntax error | Validate YAML structure |
| Missing required field | Missing identifier/name/type | Add required fields |
| Invalid identifier | Pattern mismatch | Use `^[a-zA-Z_][0-9a-zA-Z_]{0,127}$` |
| Connector not found | Invalid connectorRef | Verify connector exists |
| Permission denied | Insufficient access | Check RBAC permissions |

### Workflow: Generate and Create

1. **Generate template YAML** using this skill
2. **Review the YAML** for correctness
3. **Construct API call** with appropriate scope
4. **Execute API call** to create template
5. **Verify creation** in Harness UI or via GET API

## Instructions

When a user requests a template:

1. **Clarify requirements:**
   - What type of template? (Step, Stage, Pipeline, StepGroup)
   - What functionality should it provide?
   - What should be configurable via variables?
   - What scope? (Account, Org, Project)
   - Should it be created via API or just generate YAML?

2. **Generate valid YAML:**
   - Use correct identifier patterns: `^[a-zA-Z_][0-9a-zA-Z_]{0,127}$`
   - Use correct name patterns: `^[a-zA-Z_0-9-.][-0-9a-zA-Z_\s.]{0,127}$`
   - Use correct version pattern: `^[0-9a-zA-Z][^\s/&]{0,127}$`
   - Include all required fields (identifier, name, type, versionLabel)

3. **Design for reusability:**
   - Identify what should be parameterized
   - Add appropriate variables with descriptions
   - Set sensible defaults

4. **Add metadata:**
   - Description explaining the template's purpose
   - Tags for categorization
   - Variable descriptions

5. **Output the template YAML** in a code block for easy copying.

6. **Optionally create via API:**
   - If user wants to create the template in Harness directly
   - Provide the curl command with appropriate parameters
   - Include scope parameters (account/org/project)
   - Choose storage type (INLINE or REMOTE for Git)
   - Set `isNewTemplate=true` for new templates
   - Set `isNewTemplate=false` for new versions of existing templates

### API Creation Checklist

When creating templates via API:

- [ ] Account identifier is provided
- [ ] Org/Project identifiers match template scope
- [ ] API key has template create permissions
- [ ] YAML is valid and includes all required fields
- [ ] Version label is unique for the template
- [ ] For Git storage: connector, repo, branch, filePath provided
