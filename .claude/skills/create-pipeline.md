---
name: create-pipeline
description: Generate Harness.io v0 Pipeline YAML files for CI/CD workflows. Use when the user wants to create a Harness pipeline, build pipeline, deployment pipeline, or asks about Harness CI/CD configuration.
triggers:
  - harness pipeline
  - create pipeline
  - ci pipeline
  - cd pipeline
  - deployment pipeline
  - build pipeline
  - harness yaml
  - harness ci
  - harness cd
---

# Create Pipeline Skill

Generate Harness.io v0 Pipeline YAML files based on user requirements.

## Overview

This skill creates valid Harness CI/CD pipeline YAML configurations following the v0 schema specification. It supports CI (Continuous Integration), CD (Continuous Deployment), and other stage types.

## Schema Reference

Schema source: https://github.com/harness/harness-schema/tree/main/v0

## Pipeline Structure

Every Harness pipeline follows this root structure:

```yaml
pipeline:
  identifier: <unique_identifier>  # Pattern: ^[a-zA-Z_][0-9a-zA-Z_]{0,127}$
  name: <display_name>             # Pattern: ^[a-zA-Z_0-9-.][-0-9a-zA-Z_\s.]{0,127}$
  orgIdentifier: <org_id>          # Optional: Organization identifier
  projectIdentifier: <project_id>  # Optional: Project identifier
  description: <description>       # Optional
  tags: {}                         # Optional: key-value pairs
  stages:                          # Required: At least 1 stage
    - stage: ...
```

## Stage Types

### CI Stage (type: CI)

For build, test, and integration tasks:

```yaml
- stage:
    identifier: build
    name: Build
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
          - step: ...
```

**Infrastructure Options:**
- `Cloud` - Harness Cloud (hosted)
- `Kubernetes` - Self-hosted K8s
- `VM` - Self-hosted VM
- `Docker` - Local Docker

### CD Stage (type: Deployment)

For deployment workflows:

```yaml
- stage:
    identifier: deploy
    name: Deploy
    type: Deployment
    spec:
      deploymentType: Kubernetes  # See deployment types below
      service:
        serviceRef: <service_identifier>
      environment:
        environmentRef: <env_identifier>
        infrastructureDefinitions:
          - identifier: <infra_id>
      execution:
        steps:
          - step: ...
        rollbackSteps:
          - step: ...
    failureStrategies:
      - onFailure:
          errors:
            - AllErrors
          action:
            type: StageRollback
```

**Deployment Types:**
- `Kubernetes` - K8s deployments
- `NativeHelm` - Helm chart deployments
- `Ssh` - SSH-based deployments
- `WinRm` - Windows Remote Management
- `ServerlessAwsLambda` - AWS Lambda serverless
- `AzureWebApp` - Azure Web Apps
- `AzureFunction` - Azure Functions
- `ECS` - Amazon ECS
- `Elastigroup` - Spot Elastigroup
- `TAS` - Tanzu Application Service
- `Asg` - AWS Auto Scaling Groups
- `GoogleCloudFunctions` - GCP Cloud Functions
- `AwsLambda` - AWS Lambda
- `AWS_SAM` - AWS SAM
- `GoogleCloudRun` - Google Cloud Run
- `CustomDeployment` - Custom deployment scripts

### Approval Stage (type: Approval)

For manual or automated approvals:

```yaml
- stage:
    identifier: approval
    name: Approval
    type: Approval
    spec:
      execution:
        steps:
          - step:
              identifier: approve
              name: Approve
              type: HarnessApproval
              spec:
                approvalMessage: "Please review and approve"
                approvers:
                  userGroups:
                    - <user_group_id>
                  minimumCount: 1
                includePipelineExecutionHistory: true
              timeout: 1d
```

### Custom Stage (type: Custom)

For custom workflows:

```yaml
- stage:
    identifier: custom
    name: Custom Stage
    type: Custom
    spec:
      execution:
        steps:
          - step: ...
```

## Common Step Types

### Run Step

Execute shell commands:

```yaml
- step:
    identifier: run_step
    name: Run Commands
    type: Run
    spec:
      shell: Bash  # Sh, Bash, Powershell, Pwsh, Python
      command: |
        echo "Hello World"
        npm install
        npm test
      envVariables:
        MY_VAR: "value"
      outputVariables:
        - name: OUTPUT_VAR
          type: String
      resources:
        limits:
          memory: 500Mi
          cpu: 400m
```

### Run with Container Image

```yaml
- step:
    identifier: run_in_container
    name: Run in Container
    type: Run
    spec:
      connectorRef: <docker_connector>
      image: node:18
      shell: Bash
      command: |
        npm install
        npm test
```

### Git Clone Step

```yaml
- step:
    identifier: clone
    name: Clone Repository
    type: GitClone
    spec:
      connectorRef: <git_connector>
      repoName: my-repo
      build:
        type: branch
        spec:
          branch: main
```

### Build and Push Docker

```yaml
- step:
    identifier: build_push
    name: Build and Push
    type: BuildAndPushDockerRegistry
    spec:
      connectorRef: <docker_connector>
      repo: myrepo/myimage
      tags:
        - latest
        - <+pipeline.sequenceId>
      dockerfile: Dockerfile
      context: .
```

### Build and Push to ECR

```yaml
- step:
    identifier: build_push_ecr
    name: Build and Push to ECR
    type: BuildAndPushECR
    spec:
      connectorRef: <aws_connector>
      region: us-east-1
      account: "123456789012"
      imageName: my-image
      tags:
        - latest
```

### Build and Push to GCR

```yaml
- step:
    identifier: build_push_gcr
    name: Build and Push to GCR
    type: BuildAndPushGCR
    spec:
      connectorRef: <gcp_connector>
      host: gcr.io
      projectID: my-project
      imageName: my-image
      tags:
        - latest
```

### Build and Push to ACR

```yaml
- step:
    identifier: build_push_acr
    name: Build and Push to ACR
    type: BuildAndPushACR
    spec:
      connectorRef: <azure_connector>
      repository: myregistry.azurecr.io/myimage
      tags:
        - latest
```

### Restore Cache (GCS)

```yaml
- step:
    identifier: restore_cache
    name: Restore Cache
    type: RestoreCacheGCS
    spec:
      connectorRef: <gcp_connector>
      bucket: my-cache-bucket
      key: cache-{{ checksum "package-lock.json" }}
      archiveFormat: Tar
```

### Save Cache (GCS)

```yaml
- step:
    identifier: save_cache
    name: Save Cache
    type: SaveCacheGCS
    spec:
      connectorRef: <gcp_connector>
      bucket: my-cache-bucket
      key: cache-{{ checksum "package-lock.json" }}
      sourcePaths:
        - node_modules
      archiveFormat: Tar
```

### Upload Artifacts to GCS

```yaml
- step:
    identifier: upload_gcs
    name: Upload to GCS
    type: GCSUpload
    spec:
      connectorRef: <gcp_connector>
      bucket: my-bucket
      sourcePath: dist/
      target: artifacts/
```

### Upload Artifacts to S3

```yaml
- step:
    identifier: upload_s3
    name: Upload to S3
    type: S3Upload
    spec:
      connectorRef: <aws_connector>
      region: us-east-1
      bucket: my-bucket
      sourcePath: dist/
      target: artifacts/
```

### HTTP Step

Make HTTP requests:

```yaml
- step:
    identifier: http_call
    name: Call API
    type: Http
    spec:
      url: https://api.example.com/webhook
      method: POST
      headers:
        - key: Content-Type
          value: application/json
      body: '{"status": "deployed"}'
      assertion: <+httpResponseCode> == 200
    timeout: 30s
```

### Shell Script Step (CD)

```yaml
- step:
    identifier: shell
    name: Run Script
    type: ShellScript
    spec:
      shell: Bash
      source:
        type: Inline
        spec:
          script: |
            echo "Deploying..."
      onDelegate: true
    timeout: 10m
```

### Kubernetes Apply

```yaml
- step:
    identifier: k8s_apply
    name: Apply Manifests
    type: K8sApply
    spec:
      filePaths:
        - manifests/
      skipDryRun: false
      skipSteadyStateCheck: false
    timeout: 10m
```

### Kubernetes Rollout

```yaml
- step:
    identifier: rollout
    name: Rollout Deployment
    type: K8sRollingDeploy
    spec:
      skipDryRun: false
    timeout: 10m
```

### Kubernetes Rollback

```yaml
- step:
    identifier: rollback
    name: Rollback Deployment
    type: K8sRollingRollback
    spec: {}
    timeout: 10m
```

### Helm Deploy

```yaml
- step:
    identifier: helm_deploy
    name: Helm Deploy
    type: HelmDeploy
    spec:
      skipDryRun: false
    timeout: 10m
```

### Terraform Apply

```yaml
- step:
    identifier: tf_apply
    name: Terraform Apply
    type: TerraformApply
    spec:
      configuration:
        type: Inline
        spec:
          configFiles:
            store:
              type: Github
              spec:
                connectorRef: <git_connector>
                repoName: my-terraform-repo
                branch: main
                folderPath: terraform/
      provisionerIdentifier: my_provisioner
    timeout: 10m
```

### Background Step (Service Dependency)

```yaml
- step:
    identifier: db
    name: Database
    type: Background
    spec:
      connectorRef: <docker_connector>
      image: postgres:14
      envVariables:
        POSTGRES_PASSWORD: password
      portBindings:
        "5432": "5432"
```

### Plugin Step

```yaml
- step:
    identifier: plugin
    name: Run Plugin
    type: Plugin
    spec:
      connectorRef: <docker_connector>
      image: plugins/slack
      settings:
        webhook: <+secrets.getValue("slack_webhook")>
        channel: builds
```

### Run Tests with Intelligence

```yaml
- step:
    identifier: test
    name: Run Tests
    type: RunTests
    spec:
      language: Java
      buildTool: Maven
      args: test
      packages: com.mycompany
      runOnlySelectedTests: true
      testAnnotations: org.junit.Test
      preCommand: |
        mvn clean compile
      reports:
        type: JUnit
        spec:
          paths:
            - "**/target/surefire-reports/*.xml"
```

## Step Groups

Group steps together:

```yaml
- stepGroup:
    identifier: test_group
    name: Test Suite
    steps:
      - step:
          identifier: unit_tests
          name: Unit Tests
          type: Run
          spec:
            shell: Bash
            command: npm run test:unit
      - step:
          identifier: integration_tests
          name: Integration Tests
          type: Run
          spec:
            shell: Bash
            command: npm run test:integration
```

## Parallel Execution

Run steps in parallel:

```yaml
- parallel:
    - step:
        identifier: test_a
        name: Test A
        type: Run
        spec:
          shell: Bash
          command: npm run test:a
    - step:
        identifier: test_b
        name: Test B
        type: Run
        spec:
          shell: Bash
          command: npm run test:b
```

## Parallel Stages

```yaml
stages:
  - parallel:
      - stage:
          identifier: stage_a
          name: Stage A
          type: CI
          spec: ...
      - stage:
          identifier: stage_b
          name: Stage B
          type: CI
          spec: ...
```

## Variables

### Pipeline Variables

```yaml
pipeline:
  variables:
    - name: env
      type: String
      default: dev
      description: "Target environment"
    - name: replicas
      type: Number
      default: 3
    - name: api_key
      type: Secret
      value: <+secrets.getValue("api_key")>
```

### Stage Variables

```yaml
- stage:
    identifier: build
    variables:
      - name: BUILD_TYPE
        type: String
        value: production
```

### Reference Variables

```yaml
# Pipeline variable
<+pipeline.variables.env>

# Stage variable
<+stage.variables.BUILD_TYPE>

# Step output
<+steps.step_id.output.outputVariables.VAR_NAME>

# Execution info
<+pipeline.sequenceId>
<+pipeline.executionId>
<+pipeline.startTs>

# Trigger info
<+trigger.branch>
<+trigger.commitSha>
<+trigger.prNumber>
<+trigger.sourceBranch>
<+trigger.targetBranch>

# Codebase info
<+codebase.branch>
<+codebase.commitSha>
<+codebase.repoUrl>
```

## Codebase Configuration

For CI pipelines that clone repositories:

```yaml
pipeline:
  properties:
    ci:
      codebase:
        connectorRef: <git_connector>
        repoName: my-repo  # Optional if connector specifies repo
        build:
          type: branch
          spec:
            branch: <+trigger.branch>
        depth: 50          # Clone depth
        sslVerify: true
        prCloneStrategy: MergeCommit  # or SourceBranch
```

## Conditional Execution

### Stage Conditions

```yaml
- stage:
    when:
      pipelineStatus: Success  # Success, Failure, All
      condition: <+pipeline.variables.deploy> == "true"
```

### Step Conditions

```yaml
- step:
    when:
      stageStatus: Success
      condition: <+stage.variables.run_tests> == "true"
```

## Failure Strategies

```yaml
- stage:
    failureStrategies:
      - onFailure:
          errors:
            - AllErrors
          action:
            type: StageRollback
      - onFailure:
          errors:
            - Timeout
          action:
            type: Retry
            spec:
              retryCount: 3
              retryIntervals:
                - 10s
                - 30s
                - 1m
      - onFailure:
          errors:
            - Unknown
          action:
            type: Ignore
```

**Error Types:**
- `AllErrors`
- `Unknown`
- `Timeout`
- `Authentication`
- `Authorization`
- `Connectivity`
- `DelegateProvisioning`
- `Verification`
- `PolicyEvaluationFailure`
- `InputTimeoutError`
- `ApprovalRejection`

**Action Types:**
- `Ignore`
- `Retry`
- `MarkAsSuccess`
- `Abort`
- `StageRollback`
- `PipelineRollback`
- `ManualIntervention`
- `MarkAsFailure`

## Notification Rules

```yaml
pipeline:
  notificationRules:
    - name: Notify on Failure
      enabled: true
      pipelineEvents:
        - type: PipelineFailed
        - type: StageFailed
      notificationMethod:
        type: Slack
        spec:
          webhookUrl: <+secrets.getValue("slack_webhook")>
    - name: Email on Success
      enabled: true
      pipelineEvents:
        - type: PipelineSuccess
      notificationMethod:
        type: Email
        spec:
          recipients:
            - team@example.com
```

**Event Types:**
- `AllEvents`
- `PipelineStart`
- `PipelineSuccess`
- `PipelineFailed`
- `PipelinePaused`
- `StageStart`
- `StageSuccess`
- `StageFailed`
- `StepFailed`

## Looping Strategies

### Matrix Strategy

```yaml
- stage:
    strategy:
      matrix:
        node_version:
          - "16"
          - "18"
          - "20"
        os:
          - linux
          - macos
      maxConcurrency: 3
```

### Repeat Strategy

```yaml
- step:
    strategy:
      repeat:
        times: 3
        maxConcurrency: 1
```

### Parallelism Strategy

```yaml
- step:
    strategy:
      parallelism: 5
```

## Timeouts

```yaml
# Pipeline level
pipeline:
  timeout: 1h

# Stage level
- stage:
    timeout: 30m

# Step level
- step:
    timeout: 10m
```

**Timeout Format:** `<number><unit>` where unit is:
- `s` - seconds
- `m` - minutes
- `h` - hours
- `d` - days
- `w` - weeks

## Complete CI Pipeline Example

```yaml
pipeline:
  identifier: nodejs_ci
  name: Node.js CI Pipeline
  projectIdentifier: my_project
  orgIdentifier: my_org
  tags:
    team: platform
    language: nodejs
  properties:
    ci:
      codebase:
        connectorRef: github_connector
        repoName: my-nodejs-app
        build:
          type: branch
          spec:
            branch: <+trigger.branch>
  stages:
    - stage:
        identifier: build_and_test
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
                  identifier: install
                  name: Install Dependencies
                  type: Run
                  spec:
                    shell: Bash
                    command: npm ci
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
                        command: npm test
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
                    command: npm run build
              - step:
                  identifier: docker_build_push
                  name: Build and Push Docker Image
                  type: BuildAndPushDockerRegistry
                  spec:
                    connectorRef: dockerhub_connector
                    repo: myorg/my-nodejs-app
                    tags:
                      - <+pipeline.sequenceId>
                      - latest
        failureStrategies:
          - onFailure:
              errors:
                - AllErrors
              action:
                type: MarkAsFailure
```

## Complete CD Pipeline Example

```yaml
pipeline:
  identifier: k8s_deploy
  name: Kubernetes Deployment
  projectIdentifier: my_project
  orgIdentifier: my_org
  stages:
    - stage:
        identifier: deploy_dev
        name: Deploy to Dev
        type: Deployment
        spec:
          deploymentType: Kubernetes
          service:
            serviceRef: my_service
          environment:
            environmentRef: dev
            infrastructureDefinitions:
              - identifier: k8s_dev
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
    - stage:
        identifier: approval
        name: Production Approval
        type: Approval
        spec:
          execution:
            steps:
              - step:
                  identifier: approve
                  name: Approve Production Deploy
                  type: HarnessApproval
                  spec:
                    approvalMessage: "Approve deployment to production?"
                    approvers:
                      userGroups:
                        - prod_approvers
                      minimumCount: 1
                    includePipelineExecutionHistory: true
                  timeout: 1d
    - stage:
        identifier: deploy_prod
        name: Deploy to Production
        type: Deployment
        spec:
          deploymentType: Kubernetes
          service:
            serviceRef: my_service
          environment:
            environmentRef: prod
            infrastructureDefinitions:
              - identifier: k8s_prod
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
  notificationRules:
    - name: Slack Notifications
      enabled: true
      pipelineEvents:
        - type: PipelineFailed
        - type: PipelineSuccess
      notificationMethod:
        type: Slack
        spec:
          webhookUrl: <+secrets.getValue("slack_webhook")>
```

## Instructions

When a user requests a pipeline:

1. **Clarify requirements:**
   - What type of pipeline? (CI, CD, or both)
   - What language/framework?
   - What deployment target? (K8s, serverless, VMs, etc.)
   - What cloud provider?
   - Any specific steps needed? (testing, security scanning, approvals)

2. **Generate valid YAML:**
   - Use correct identifier patterns: `^[a-zA-Z_][0-9a-zA-Z_]{0,127}$`
   - Use correct name patterns: `^[a-zA-Z_0-9-.][-0-9a-zA-Z_\s.]{0,127}$`
   - Include all required fields
   - Use proper indentation (2 spaces)

3. **Add appropriate defaults:**
   - Reasonable timeouts
   - Failure strategies
   - Resource limits where applicable

4. **Use expressions appropriately:**
   - `<+pipeline.variables.*>` for pipeline variables
   - `<+stage.variables.*>` for stage variables
   - `<+secrets.getValue("...")>` for secrets
   - `<+trigger.*>` for trigger information

5. **Output the pipeline YAML** in a code block for easy copying.
