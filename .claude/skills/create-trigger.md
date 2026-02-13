---
name: create-trigger
description: Generate Harness.io v0 Trigger YAML files to automatically start pipelines. Use when the user wants to create a Harness trigger, webhook trigger, scheduled trigger, cron trigger, or artifact trigger.
triggers:
  - harness trigger
  - create trigger
  - webhook trigger
  - cron trigger
  - scheduled trigger
  - artifact trigger
  - pipeline trigger
  - github trigger
  - gitlab trigger
---

# Create Trigger Skill

Generate Harness.io v0 Trigger YAML files to automatically start pipeline executions.

## Overview

This skill creates valid Harness trigger YAML configurations following the v0 schema specification. Triggers automatically start pipelines based on events like code pushes, pull requests, schedules, or artifact updates.

## Schema Reference

Schema source: https://github.com/harness/harness-schema/tree/main/v0/trigger

## Trigger Types

| Type | Description | Use Case |
|------|-------------|----------|
| `Webhook` | Git repository events | Push, PR, tags, releases |
| `Scheduled` | Cron-based scheduling | Nightly builds, periodic tasks |
| `Artifact` | Artifact/image updates | New Docker image, S3 upload |
| `Manifest` | Manifest changes | Helm chart updates |
| `MultiRegionArtifact` | Multi-region artifacts | Cross-region deployments |

## Trigger Structure

Every Harness trigger follows this root structure:

```yaml
trigger:
  identifier: <unique_identifier>    # Pattern: ^[a-zA-Z_][0-9a-zA-Z_]{0,127}$
  name: <display_name>               # Pattern: ^[a-zA-Z_0-9-.][-0-9a-zA-Z_\s.]{0,127}$
  orgIdentifier: <org_id>            # Required
  projectIdentifier: <project_id>    # Required
  pipelineIdentifier: <pipeline_id>  # Pipeline to trigger
  description: <description>         # Optional
  enabled: true                      # Enable/disable trigger
  tags: {}                           # Optional: key-value pairs
  source:
    type: <trigger_type>             # Webhook, Scheduled, Artifact, etc.
    spec:
      # Type-specific configuration
  inputYaml: |                       # Optional: Pipeline input overrides
    pipeline:
      variables:
        - name: varName
          value: "value"
```

## Webhook Triggers

### GitHub Push Trigger

Trigger on code pushes to GitHub:

```yaml
trigger:
  identifier: github_push_trigger
  name: GitHub Push Trigger
  orgIdentifier: my_org
  projectIdentifier: my_project
  pipelineIdentifier: my_pipeline
  description: "Trigger pipeline on push to main branch"
  enabled: true
  tags:
    type: ci
    repo: my-repo
  source:
    type: Webhook
    spec:
      type: Github
      spec:
        type: Push
        spec:
          connectorRef: github_connector
          repoName: my-org/my-repo
          autoAbortPreviousExecutions: true
          payloadConditions:
            - key: targetBranch
              operator: Equals
              value: main
            - key: sourceBranch
              operator: Regex
              value: ^feature/.*
          headerConditions: []
          jexlCondition: ""
```

### GitHub Pull Request Trigger

Trigger on PR events:

```yaml
trigger:
  identifier: github_pr_trigger
  name: GitHub PR Trigger
  orgIdentifier: my_org
  projectIdentifier: my_project
  pipelineIdentifier: pr_pipeline
  description: "Trigger on PR open, sync, or reopen"
  enabled: true
  source:
    type: Webhook
    spec:
      type: Github
      spec:
        type: PullRequest
        spec:
          connectorRef: github_connector
          repoName: my-org/my-repo
          autoAbortPreviousExecutions: true
          actions:
            - Open
            - Synchronize
            - Reopen
            - ReadyForReview
          payloadConditions:
            - key: targetBranch
              operator: In
              value: main, develop
          headerConditions: []
          jexlCondition: ""
  inputYaml: |
    pipeline:
      variables:
        - name: pr_number
          type: String
          value: <+trigger.prNumber>
        - name: source_branch
          type: String
          value: <+trigger.sourceBranch>
```

### GitHub Issue Comment Trigger

Trigger on PR/issue comments:

```yaml
trigger:
  identifier: github_comment_trigger
  name: GitHub Comment Trigger
  orgIdentifier: my_org
  projectIdentifier: my_project
  pipelineIdentifier: deploy_pipeline
  description: "Trigger deployment on /deploy comment"
  enabled: true
  source:
    type: Webhook
    spec:
      type: Github
      spec:
        type: IssueComment
        spec:
          connectorRef: github_connector
          repoName: my-org/my-repo
          autoAbortPreviousExecutions: false
          payloadConditions:
            - key: <+trigger.payload.comment.body>
              operator: Contains
              value: "/deploy"
          actions:
            - Create
          jexlCondition: ""
```

### GitHub Release Trigger

Trigger on release events:

```yaml
trigger:
  identifier: github_release_trigger
  name: GitHub Release Trigger
  orgIdentifier: my_org
  projectIdentifier: my_project
  pipelineIdentifier: release_pipeline
  description: "Trigger on new release"
  enabled: true
  source:
    type: Webhook
    spec:
      type: Github
      spec:
        type: Release
        spec:
          connectorRef: github_connector
          repoName: my-org/my-repo
          actions:
            - Create
            - Publish
          payloadConditions:
            - key: <+trigger.payload.release.tag_name>
              operator: Regex
              value: ^v[0-9]+\.[0-9]+\.[0-9]+$
```

### GitLab Push Trigger

```yaml
trigger:
  identifier: gitlab_push_trigger
  name: GitLab Push Trigger
  orgIdentifier: my_org
  projectIdentifier: my_project
  pipelineIdentifier: my_pipeline
  enabled: true
  source:
    type: Webhook
    spec:
      type: Gitlab
      spec:
        type: Push
        spec:
          connectorRef: gitlab_connector
          repoName: my-group/my-repo
          autoAbortPreviousExecutions: true
          payloadConditions:
            - key: targetBranch
              operator: Equals
              value: main
```

### GitLab Merge Request Trigger

```yaml
trigger:
  identifier: gitlab_mr_trigger
  name: GitLab MR Trigger
  orgIdentifier: my_org
  projectIdentifier: my_project
  pipelineIdentifier: mr_pipeline
  enabled: true
  source:
    type: Webhook
    spec:
      type: Gitlab
      spec:
        type: MergeRequest
        spec:
          connectorRef: gitlab_connector
          repoName: my-group/my-repo
          autoAbortPreviousExecutions: true
          actions:
            - Open
            - Reopen
            - Sync
          payloadConditions:
            - key: targetBranch
              operator: Equals
              value: main
```

### GitLab Tag Trigger

```yaml
trigger:
  identifier: gitlab_tag_trigger
  name: GitLab Tag Trigger
  orgIdentifier: my_org
  projectIdentifier: my_project
  pipelineIdentifier: release_pipeline
  enabled: true
  source:
    type: Webhook
    spec:
      type: Gitlab
      spec:
        type: Tag
        spec:
          connectorRef: gitlab_connector
          repoName: my-group/my-repo
          payloadConditions:
            - key: <+trigger.payload.ref>
              operator: Regex
              value: ^refs/tags/v.*
```

### Bitbucket Push Trigger

```yaml
trigger:
  identifier: bitbucket_push_trigger
  name: Bitbucket Push Trigger
  orgIdentifier: my_org
  projectIdentifier: my_project
  pipelineIdentifier: my_pipeline
  enabled: true
  source:
    type: Webhook
    spec:
      type: Bitbucket
      spec:
        type: Push
        spec:
          connectorRef: bitbucket_connector
          repoName: my-workspace/my-repo
          autoAbortPreviousExecutions: true
          payloadConditions:
            - key: targetBranch
              operator: Equals
              value: main
```

### Bitbucket Pull Request Trigger

```yaml
trigger:
  identifier: bitbucket_pr_trigger
  name: Bitbucket PR Trigger
  orgIdentifier: my_org
  projectIdentifier: my_project
  pipelineIdentifier: pr_pipeline
  enabled: true
  source:
    type: Webhook
    spec:
      type: Bitbucket
      spec:
        type: PullRequest
        spec:
          connectorRef: bitbucket_connector
          repoName: my-workspace/my-repo
          autoAbortPreviousExecutions: true
          actions:
            - Open
            - Update
            - Reopen
          payloadConditions:
            - key: targetBranch
              operator: Equals
              value: main
```

### Azure Repos Push Trigger

```yaml
trigger:
  identifier: azure_push_trigger
  name: Azure Repos Push Trigger
  orgIdentifier: my_org
  projectIdentifier: my_project
  pipelineIdentifier: my_pipeline
  enabled: true
  source:
    type: Webhook
    spec:
      type: AzureRepo
      spec:
        type: Push
        spec:
          connectorRef: azure_connector
          repoName: my-repo
          autoAbortPreviousExecutions: true
          payloadConditions:
            - key: targetBranch
              operator: Equals
              value: main
```

### Custom Webhook Trigger

For custom webhook integrations:

```yaml
trigger:
  identifier: custom_webhook_trigger
  name: Custom Webhook Trigger
  orgIdentifier: my_org
  projectIdentifier: my_project
  pipelineIdentifier: my_pipeline
  enabled: true
  encryptedWebhookSecretIdentifier: webhook_secret
  source:
    type: Webhook
    spec:
      type: Custom
      spec:
        payloadConditions:
          - key: <+trigger.payload.event_type>
            operator: Equals
            value: deployment
          - key: <+trigger.payload.environment>
            operator: In
            value: staging, production
        headerConditions:
          - key: X-Custom-Header
            operator: Equals
            value: expected-value
        jexlCondition: "<+trigger.payload.priority> >= 1"
  inputYaml: |
    pipeline:
      variables:
        - name: environment
          type: String
          value: <+trigger.payload.environment>
        - name: version
          type: String
          value: <+trigger.payload.version>
```

## Scheduled Triggers

### Cron Trigger

Schedule pipelines using cron expressions:

```yaml
trigger:
  identifier: nightly_build
  name: Nightly Build
  orgIdentifier: my_org
  projectIdentifier: my_project
  pipelineIdentifier: build_pipeline
  description: "Run every night at 2 AM UTC"
  enabled: true
  tags:
    schedule: nightly
  source:
    type: Scheduled
    spec:
      type: Cron
      spec:
        expression: "0 2 * * *"
        timezone: "UTC"
```

### Common Cron Expressions

```yaml
# Every hour
expression: "0 * * * *"

# Every day at midnight
expression: "0 0 * * *"

# Every day at 2 AM
expression: "0 2 * * *"

# Every Monday at 9 AM
expression: "0 9 * * 1"

# Every weekday at 6 PM
expression: "0 18 * * 1-5"

# First day of every month at midnight
expression: "0 0 1 * *"

# Every 15 minutes
expression: "*/15 * * * *"

# Every 6 hours
expression: "0 */6 * * *"
```

### Weekly Release Trigger

```yaml
trigger:
  identifier: weekly_release
  name: Weekly Release
  orgIdentifier: my_org
  projectIdentifier: my_project
  pipelineIdentifier: release_pipeline
  description: "Release every Friday at 5 PM EST"
  enabled: true
  source:
    type: Scheduled
    spec:
      type: Cron
      spec:
        expression: "0 17 * * 5"
        timezone: "America/New_York"
  inputYaml: |
    pipeline:
      variables:
        - name: release_type
          type: String
          value: "weekly"
```

### Multiple Schedule Trigger

Create separate triggers for different schedules:

```yaml
# Staging deployment - twice daily
trigger:
  identifier: staging_deploy_morning
  name: Staging Deploy Morning
  orgIdentifier: my_org
  projectIdentifier: my_project
  pipelineIdentifier: deploy_pipeline
  enabled: true
  source:
    type: Scheduled
    spec:
      type: Cron
      spec:
        expression: "0 9 * * 1-5"
        timezone: "UTC"
  inputYaml: |
    pipeline:
      variables:
        - name: environment
          type: String
          value: "staging"
---
trigger:
  identifier: staging_deploy_evening
  name: Staging Deploy Evening
  orgIdentifier: my_org
  projectIdentifier: my_project
  pipelineIdentifier: deploy_pipeline
  enabled: true
  source:
    type: Scheduled
    spec:
      type: Cron
      spec:
        expression: "0 17 * * 1-5"
        timezone: "UTC"
  inputYaml: |
    pipeline:
      variables:
        - name: environment
          type: String
          value: "staging"
```

## Artifact Triggers

### Docker Hub Trigger

Trigger when new Docker image is pushed:

```yaml
trigger:
  identifier: docker_image_trigger
  name: Docker Image Trigger
  orgIdentifier: my_org
  projectIdentifier: my_project
  pipelineIdentifier: deploy_pipeline
  description: "Deploy when new image is pushed"
  enabled: true
  source:
    type: Artifact
    pollInterval: "5m"
    spec:
      type: DockerRegistry
      spec:
        connectorRef: dockerhub_connector
        imagePath: myorg/myapp
        tag: <+trigger.artifact.build>
        eventConditions: []
        metaDataConditions:
          - key: <+trigger.artifact.metadata.tag>
            operator: Regex
            value: ^v[0-9]+\.[0-9]+\.[0-9]+$
        jexlCondition: ""
      stageIdentifier: deploy_stage
      artifactRef: primary
  inputYaml: |
    pipeline:
      variables:
        - name: image_tag
          type: String
          value: <+trigger.artifact.build>
```

### ECR Trigger

Trigger on new ECR image:

```yaml
trigger:
  identifier: ecr_image_trigger
  name: ECR Image Trigger
  orgIdentifier: my_org
  projectIdentifier: my_project
  pipelineIdentifier: deploy_pipeline
  enabled: true
  source:
    type: Artifact
    pollInterval: "5m"
    spec:
      type: Ecr
      spec:
        connectorRef: aws_connector
        region: us-east-1
        registryId: "123456789012"
        imagePath: my-app
        tag: <+trigger.artifact.build>
        eventConditions: []
        metaDataConditions:
          - key: <+trigger.artifact.metadata.tag>
            operator: Regex
            value: ^(latest|v.*)$
        jexlCondition: ""
      stageIdentifier: deploy_stage
      artifactRef: primary
```

### GCR Trigger

```yaml
trigger:
  identifier: gcr_image_trigger
  name: GCR Image Trigger
  orgIdentifier: my_org
  projectIdentifier: my_project
  pipelineIdentifier: deploy_pipeline
  enabled: true
  source:
    type: Artifact
    pollInterval: "5m"
    spec:
      type: Gcr
      spec:
        connectorRef: gcp_connector
        registryHostname: gcr.io
        imagePath: my-project/my-app
        tag: <+trigger.artifact.build>
      stageIdentifier: deploy_stage
      artifactRef: primary
```

### Google Artifact Registry Trigger

```yaml
trigger:
  identifier: gar_image_trigger
  name: GAR Image Trigger
  orgIdentifier: my_org
  projectIdentifier: my_project
  pipelineIdentifier: deploy_pipeline
  enabled: true
  source:
    type: Artifact
    pollInterval: "5m"
    spec:
      type: GoogleArtifactRegistry
      spec:
        connectorRef: gcp_connector
        project: my-gcp-project
        region: us-central1
        repositoryName: my-repo
        package: my-app
        version: <+trigger.artifact.build>
      stageIdentifier: deploy_stage
      artifactRef: primary
```

### ACR Trigger

```yaml
trigger:
  identifier: acr_image_trigger
  name: ACR Image Trigger
  orgIdentifier: my_org
  projectIdentifier: my_project
  pipelineIdentifier: deploy_pipeline
  enabled: true
  source:
    type: Artifact
    pollInterval: "5m"
    spec:
      type: Acr
      spec:
        connectorRef: azure_connector
        subscriptionId: my-subscription-id
        registry: myregistry
        repository: my-app
        tag: <+trigger.artifact.build>
      stageIdentifier: deploy_stage
      artifactRef: primary
```

### S3 Artifact Trigger

Trigger when file is uploaded to S3:

```yaml
trigger:
  identifier: s3_artifact_trigger
  name: S3 Artifact Trigger
  orgIdentifier: my_org
  projectIdentifier: my_project
  pipelineIdentifier: deploy_pipeline
  enabled: true
  source:
    type: Artifact
    pollInterval: "5m"
    spec:
      type: AmazonS3
      spec:
        connectorRef: aws_connector
        region: us-east-1
        bucketName: my-artifacts-bucket
        filePath: releases/
        filePathRegex: ".*\\.zip$"
      stageIdentifier: deploy_stage
      artifactRef: primary
```

### Nexus Artifact Trigger

```yaml
trigger:
  identifier: nexus_artifact_trigger
  name: Nexus Artifact Trigger
  orgIdentifier: my_org
  projectIdentifier: my_project
  pipelineIdentifier: deploy_pipeline
  enabled: true
  source:
    type: Artifact
    pollInterval: "10m"
    spec:
      type: Nexus3Registry
      spec:
        connectorRef: nexus_connector
        repository: maven-releases
        repositoryFormat: docker
        artifactPath: my-org/my-app
        tag: <+trigger.artifact.build>
      stageIdentifier: deploy_stage
      artifactRef: primary
```

### GitHub Packages Trigger

```yaml
trigger:
  identifier: ghcr_trigger
  name: GitHub Packages Trigger
  orgIdentifier: my_org
  projectIdentifier: my_project
  pipelineIdentifier: deploy_pipeline
  enabled: true
  source:
    type: Artifact
    pollInterval: "5m"
    spec:
      type: GithubPackageRegistry
      spec:
        connectorRef: github_connector
        org: my-org
        packageName: my-app
        packageType: container
        version: <+trigger.artifact.build>
      stageIdentifier: deploy_stage
      artifactRef: primary
```

## Payload Conditions

Filter triggers using payload conditions:

### Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `Equals` | Exact match | `value: main` |
| `NotEquals` | Not equal | `value: develop` |
| `In` | In list | `value: main, develop, release` |
| `NotIn` | Not in list | `value: test, experimental` |
| `Regex` | Regex match | `value: ^feature/.*` |
| `StartsWith` | Starts with | `value: release-` |
| `EndsWith` | Ends with | `value: -hotfix` |
| `Contains` | Contains | `value: urgent` |
| `DoesNotContain` | Does not contain | `value: wip` |

### Common Payload Keys

**GitHub/GitLab Push:**
```yaml
payloadConditions:
  - key: targetBranch
    operator: Equals
    value: main
  - key: sourceBranch
    operator: Regex
    value: ^feature/.*
  - key: <+trigger.payload.commits[0].message>
    operator: DoesNotContain
    value: "[skip ci]"
```

**GitHub/GitLab PR:**
```yaml
payloadConditions:
  - key: targetBranch
    operator: In
    value: main, develop
  - key: sourceBranch
    operator: Regex
    value: ^(feature|bugfix)/.*
  - key: <+trigger.payload.pull_request.draft>
    operator: Equals
    value: "false"
```

## JEXL Conditions

Use JEXL expressions for complex conditions:

```yaml
# Trigger only if not a draft PR and has specific label
jexlCondition: >
  <+trigger.payload.pull_request.draft> == false &&
  <+trigger.payload.pull_request.labels>.contains("ready-for-ci")

# Trigger only for specific authors
jexlCondition: >
  <+trigger.payload.sender.login> != "dependabot[bot]"

# Trigger based on file changes
jexlCondition: >
  <+trigger.payload.commits>.any(c -> c.modified.any(f -> f.startsWith("src/")))
```

## Pipeline Input Overrides

Pass trigger data to pipeline variables:

```yaml
inputYaml: |
  pipeline:
    variables:
      - name: branch
        type: String
        value: <+trigger.branch>
      - name: commit_sha
        type: String
        value: <+trigger.commitSha>
      - name: pr_number
        type: String
        value: <+trigger.prNumber>
      - name: pr_title
        type: String
        value: <+trigger.prTitle>
      - name: source_branch
        type: String
        value: <+trigger.sourceBranch>
      - name: target_branch
        type: String
        value: <+trigger.targetBranch>
      - name: repo_url
        type: String
        value: <+trigger.repoUrl>
      - name: trigger_type
        type: String
        value: <+trigger.type>
```

### Artifact Trigger Variables

```yaml
inputYaml: |
  pipeline:
    variables:
      - name: image_tag
        type: String
        value: <+trigger.artifact.build>
      - name: artifact_path
        type: String
        value: <+trigger.artifact.artifactPath>
      - name: artifact_metadata
        type: String
        value: <+trigger.artifact.metadata.tag>
```

## Selective Stage Execution

Execute only specific stages:

```yaml
trigger:
  identifier: hotfix_trigger
  name: Hotfix Trigger
  orgIdentifier: my_org
  projectIdentifier: my_project
  pipelineIdentifier: full_pipeline
  enabled: true
  stagesToExecute:
    - build_stage
    - deploy_prod_stage  # Skip staging for hotfixes
  source:
    type: Webhook
    spec:
      type: Github
      spec:
        type: Push
        spec:
          connectorRef: github_connector
          repoName: my-org/my-repo
          payloadConditions:
            - key: targetBranch
              operator: Regex
              value: ^hotfix/.*
```

## Input Set References

Use predefined input sets:

```yaml
trigger:
  identifier: prod_deploy_trigger
  name: Production Deploy Trigger
  orgIdentifier: my_org
  projectIdentifier: my_project
  pipelineIdentifier: deploy_pipeline
  enabled: true
  inputSetRefs:
    - production_inputs
    - security_scan_inputs
  source:
    type: Webhook
    spec:
      type: Github
      spec:
        type: Push
        spec:
          connectorRef: github_connector
          repoName: my-org/my-repo
          payloadConditions:
            - key: targetBranch
              operator: Equals
              value: main
```

## Complete Examples

### Full CI/CD Trigger Setup

```yaml
# PR Trigger - runs tests
trigger:
  identifier: pr_ci_trigger
  name: PR CI Trigger
  orgIdentifier: my_org
  projectIdentifier: my_project
  pipelineIdentifier: ci_pipeline
  description: "Run CI on pull requests"
  enabled: true
  tags:
    type: ci
  source:
    type: Webhook
    spec:
      type: Github
      spec:
        type: PullRequest
        spec:
          connectorRef: github_connector
          repoName: my-org/my-app
          autoAbortPreviousExecutions: true
          actions:
            - Open
            - Synchronize
            - Reopen
          payloadConditions:
            - key: targetBranch
              operator: In
              value: main, develop
          jexlCondition: "<+trigger.payload.pull_request.draft> == false"
  inputYaml: |
    pipeline:
      variables:
        - name: pr_number
          type: String
          value: <+trigger.prNumber>
        - name: commit_sha
          type: String
          value: <+trigger.commitSha>
---
# Main branch push - deploy to staging
trigger:
  identifier: staging_deploy_trigger
  name: Staging Deploy Trigger
  orgIdentifier: my_org
  projectIdentifier: my_project
  pipelineIdentifier: deploy_pipeline
  description: "Deploy to staging on merge to main"
  enabled: true
  tags:
    type: cd
    environment: staging
  source:
    type: Webhook
    spec:
      type: Github
      spec:
        type: Push
        spec:
          connectorRef: github_connector
          repoName: my-org/my-app
          autoAbortPreviousExecutions: false
          payloadConditions:
            - key: targetBranch
              operator: Equals
              value: main
  inputYaml: |
    pipeline:
      variables:
        - name: environment
          type: String
          value: staging
        - name: commit_sha
          type: String
          value: <+trigger.commitSha>
---
# Release tag - deploy to production
trigger:
  identifier: prod_release_trigger
  name: Production Release Trigger
  orgIdentifier: my_org
  projectIdentifier: my_project
  pipelineIdentifier: deploy_pipeline
  description: "Deploy to production on release tag"
  enabled: true
  tags:
    type: cd
    environment: production
  source:
    type: Webhook
    spec:
      type: Github
      spec:
        type: Release
        spec:
          connectorRef: github_connector
          repoName: my-org/my-app
          actions:
            - Publish
          payloadConditions:
            - key: <+trigger.payload.release.tag_name>
              operator: Regex
              value: ^v[0-9]+\.[0-9]+\.[0-9]+$
  inputYaml: |
    pipeline:
      variables:
        - name: environment
          type: String
          value: production
        - name: version
          type: String
          value: <+trigger.payload.release.tag_name>
```

## Instructions

When a user requests a trigger:

1. **Clarify requirements:**
   - What type of trigger? (Webhook, Scheduled, Artifact)
   - What events should trigger? (Push, PR, cron, new image)
   - Which pipeline should be triggered?
   - What conditions/filters are needed?
   - What pipeline inputs need to be passed?

2. **Generate valid YAML:**
   - Use correct identifier patterns: `^[a-zA-Z_][0-9a-zA-Z_]{0,127}$`
   - Use correct name patterns: `^[a-zA-Z_0-9-.][-0-9a-zA-Z_\s.]{0,127}$`
   - Include required fields (identifier, orgIdentifier, projectIdentifier)
   - Use proper indentation (2 spaces)

3. **Add appropriate filters:**
   - Branch conditions for webhooks
   - Tag/version filters for artifacts
   - JEXL conditions for complex logic

4. **Configure pipeline inputs:**
   - Pass relevant trigger variables
   - Use input set references when appropriate

5. **Output the trigger YAML** in a code block for easy copying.
