---
name: webhook-manager
description: Manage Harness.io GitX Webhooks via the API. Configure webhooks for Git repository events to enable automated pipeline triggers and Git sync.
triggers:
  - harness webhook
  - manage webhooks
  - gitx webhook
  - create webhook
  - git webhook
  - webhook manager
  - webhook api
---

# Webhook Manager Skill

Manage Harness.io GitX Webhooks via the API for Git integration.

## Overview

GitX Webhooks in Harness enable integration with Git repositories for:
- Automatic entity sync from Git
- Event notifications for Git changes
- Bi-directional Git synchronization
- Pipeline trigger automation
- Managing webhooks via the Harness API

## Webhook Structure

Every GitX webhook follows this structure:

```yaml
webhook:
  identifier: <unique_identifier>
  name: <display_name>
  connectorRef: <git_connector_id>
  repo: <repository_name>
  folderPaths:
    - <folder_path>
```

## Webhook Examples

### Basic GitX Webhook

```yaml
webhook:
  identifier: main_repo_webhook
  name: Main Repository Webhook
  connectorRef: github_connector
  repo: my-org/my-repo
  folderPaths:
    - .harness/
```

### Multi-Folder Webhook

```yaml
webhook:
  identifier: harness_config_webhook
  name: Harness Config Sync
  connectorRef: github_connector
  repo: my-org/platform-config
  folderPaths:
    - .harness/pipelines/
    - .harness/templates/
    - .harness/services/
```

### Organization-Level Webhook

```yaml
webhook:
  identifier: org_config_webhook
  name: Organization Config Webhook
  connectorRef: github_org_connector
  repo: my-org/harness-org-config
  folderPaths:
    - org/
    - shared/
```

## Creating Webhooks via API

### API Reference

**Account-level:** `POST /v1/gitx-webhooks`
**Org-level:** `POST /v1/orgs/{org}/gitx-webhooks`
**Project-level:** `POST /v1/orgs/{org}/projects/{project}/gitx-webhooks`

**Documentation:** https://apidocs.harness.io/tag/GitX-Webhooks

### Request Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-api-key` | Yes | Harness API key |
| `Harness-Account` | Yes | Account identifier |
| `Content-Type` | Yes | `application/json` |

### Create Account-Level Webhook

```bash
curl -X POST \
  'https://app.harness.io/v1/gitx-webhooks' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}' \
  -H 'Content-Type: application/json' \
  -d '{
    "identifier": "account_config_webhook",
    "name": "Account Config Webhook",
    "connector_ref": "github_connector",
    "repo": "my-org/harness-account-config",
    "folder_paths": [".harness/"]
  }'
```

### Create Org-Level Webhook

```bash
curl -X POST \
  'https://app.harness.io/v1/orgs/{org}/gitx-webhooks' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}' \
  -H 'Content-Type: application/json' \
  -d '{
    "identifier": "org_pipelines_webhook",
    "name": "Organization Pipelines Webhook",
    "connector_ref": "github_connector",
    "repo": "my-org/org-pipelines",
    "folder_paths": [".harness/pipelines/", ".harness/templates/"]
  }'
```

### Create Project-Level Webhook

```bash
curl -X POST \
  'https://app.harness.io/v1/orgs/{org}/projects/{project}/gitx-webhooks' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}' \
  -H 'Content-Type: application/json' \
  -d '{
    "identifier": "project_webhook",
    "name": "Project Config Webhook",
    "connector_ref": "github_connector",
    "repo": "my-org/my-service",
    "folder_paths": [".harness/"]
  }'
```

### Response

**Success (201 Created):**

```json
{
  "identifier": "project_webhook",
  "name": "Project Config Webhook",
  "connector_ref": "github_connector",
  "repo": "my-org/my-service",
  "folder_paths": [".harness/"],
  "webhook_url": "https://app.harness.io/ng/api/gitx/webhook/{webhookToken}",
  "created": 1707500000000,
  "updated": 1707500000000
}
```

### List Webhooks

**Endpoint:** `GET /v1/gitx-webhooks`

```bash
curl -X GET \
  'https://app.harness.io/v1/gitx-webhooks?page=0&limit=100' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}'
```

### Get Webhook

**Endpoint:** `GET /v1/gitx-webhooks/{gitx-webhook}`

```bash
curl -X GET \
  'https://app.harness.io/v1/gitx-webhooks/{webhookIdentifier}' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}'
```

### Update Webhook

**Endpoint:** `PUT /v1/gitx-webhooks/{gitx-webhook}`

```bash
curl -X PUT \
  'https://app.harness.io/v1/gitx-webhooks/{webhookIdentifier}' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}' \
  -H 'Content-Type: application/json' \
  -d '{
    "identifier": "project_webhook",
    "name": "Project Config Webhook (Updated)",
    "connector_ref": "github_connector",
    "repo": "my-org/my-service",
    "folder_paths": [".harness/", "config/"]
  }'
```

### Delete Webhook

**Endpoint:** `DELETE /v1/gitx-webhooks/{gitx-webhook}`

```bash
curl -X DELETE \
  'https://app.harness.io/v1/gitx-webhooks/{webhookIdentifier}' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}'
```

## Webhook Events

### List Webhook Events

**Endpoint:** `GET /v1/gitx-webhooks-events`

```bash
curl -X GET \
  'https://app.harness.io/v1/gitx-webhooks-events?webhook_identifier={webhookId}&page=0&limit=50' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}'
```

### Event Response

```json
{
  "events": [
    {
      "event_id": "event123",
      "webhook_identifier": "project_webhook",
      "event_type": "PUSH",
      "status": "SUCCESS",
      "trigger_time": 1707500000000,
      "processing_time": 1500,
      "commit_id": "abc123def",
      "branch": "main",
      "files_changed": [
        ".harness/pipelines/deploy.yaml",
        ".harness/services/api.yaml"
      ]
    }
  ]
}
```

## Configuring Git Provider Webhooks

After creating a GitX webhook in Harness, you need to configure the webhook in your Git provider.

### GitHub Webhook Setup

1. Go to repository Settings → Webhooks → Add webhook
2. Use the webhook URL from Harness response
3. Content type: `application/json`
4. Events: Push events, Pull request events

```bash
# GitHub API to create webhook
curl -X POST \
  'https://api.github.com/repos/{owner}/{repo}/hooks' \
  -H 'Authorization: token {githubToken}' \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "web",
    "active": true,
    "events": ["push", "pull_request"],
    "config": {
      "url": "https://app.harness.io/ng/api/gitx/webhook/{webhookToken}",
      "content_type": "json"
    }
  }'
```

### GitLab Webhook Setup

1. Go to repository Settings → Webhooks
2. Add the webhook URL from Harness
3. Select triggers: Push events, Merge request events

```bash
# GitLab API to create webhook
curl -X POST \
  'https://gitlab.com/api/v4/projects/{projectId}/hooks' \
  -H 'PRIVATE-TOKEN: {gitlabToken}' \
  -H 'Content-Type: application/json' \
  -d '{
    "url": "https://app.harness.io/ng/api/gitx/webhook/{webhookToken}",
    "push_events": true,
    "merge_requests_events": true
  }'
```

### Bitbucket Webhook Setup

1. Go to repository Settings → Webhooks
2. Add webhook with Harness URL
3. Select triggers: Repository push

```bash
# Bitbucket API to create webhook
curl -X POST \
  'https://api.bitbucket.org/2.0/repositories/{workspace}/{repo}/hooks' \
  -H 'Authorization: Bearer {accessToken}' \
  -H 'Content-Type: application/json' \
  -d '{
    "description": "Harness GitX Webhook",
    "url": "https://app.harness.io/ng/api/gitx/webhook/{webhookToken}",
    "active": true,
    "events": ["repo:push", "pullrequest:created", "pullrequest:updated"]
  }'
```

## Common Scenarios

### Pipeline Git Sync

Sync pipeline definitions from Git:

```bash
curl -X POST \
  'https://app.harness.io/v1/orgs/{org}/projects/{project}/gitx-webhooks' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}' \
  -H 'Content-Type: application/json' \
  -d '{
    "identifier": "pipeline_sync",
    "name": "Pipeline Git Sync",
    "connector_ref": "github_connector",
    "repo": "my-org/my-service",
    "folder_paths": [".harness/pipelines/"]
  }'
```

### Template Repository Sync

Sync shared templates from a central repository:

```bash
curl -X POST \
  'https://app.harness.io/v1/orgs/{org}/gitx-webhooks' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}' \
  -H 'Content-Type: application/json' \
  -d '{
    "identifier": "template_sync",
    "name": "Template Repository Sync",
    "connector_ref": "github_connector",
    "repo": "my-org/harness-templates",
    "folder_paths": [
      "templates/steps/",
      "templates/stages/",
      "templates/pipelines/"
    ]
  }'
```

### Multi-Environment Config Sync

Sync environment-specific configurations:

```bash
curl -X POST \
  'https://app.harness.io/v1/orgs/{org}/projects/{project}/gitx-webhooks' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}' \
  -H 'Content-Type: application/json' \
  -d '{
    "identifier": "env_config_sync",
    "name": "Environment Config Sync",
    "connector_ref": "github_connector",
    "repo": "my-org/my-service",
    "folder_paths": [
      ".harness/environments/",
      ".harness/infrastructures/",
      ".harness/services/"
    ]
  }'
```

## Error Handling

### Common Errors

| Error Code | Description | Solution |
|------------|-------------|----------|
| `INVALID_REQUEST` | Malformed request or missing fields | Validate request structure |
| `DUPLICATE_IDENTIFIER` | Webhook with same ID exists | Use unique identifier |
| `CONNECTOR_NOT_FOUND` | Invalid connector reference | Verify connector exists |
| `REPOSITORY_NOT_FOUND` | Invalid repository | Check repo name and connector access |

### Validation Errors

```json
// Common webhook validation issues:

// Invalid folder path format
{
  "folder_paths": [
    "harness/"  // Should start with . or be relative
    ".harness/"  // Correct
  ]
}

// Missing required field
{
  "identifier": "my_webhook",
  "name": "My Webhook"
  // Missing: connector_ref, repo, folder_paths
}
```

## Troubleshooting

### Check Webhook Status

```bash
# List recent webhook events
curl -X GET \
  'https://app.harness.io/v1/gitx-webhooks-events?webhook_identifier={webhookId}&limit=10' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}'
```

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Events not received | Webhook URL not configured in Git | Add webhook URL to Git provider |
| Events failing | Invalid connector permissions | Check connector has repo access |
| Sync not working | Wrong folder paths | Verify folder paths match repository |
| Authentication errors | Expired credentials | Rotate connector credentials |

### Validate Webhook Connection

```bash
# Get webhook details to verify configuration
curl -X GET \
  'https://app.harness.io/v1/gitx-webhooks/{webhookId}' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}'
```

### Webhook Events Not Triggering

1. **Check Git provider configuration:**
   - Webhook URL must be correctly configured
   - Content type must be `application/json`
   - Required events must be selected

2. **Verify payload signature:**
   - Some providers require secret for validation
   - Check webhook secret matches

3. **Test webhook delivery:**
   - Use Git provider's webhook test feature
   - Check for delivery failures in Git UI

### Entities Not Syncing

1. **Verify folder paths:**
   - Paths must match exactly (case-sensitive)
   - Check for leading/trailing slashes

2. **Check file format:**
   - Files must be valid Harness YAML
   - Check for syntax errors

3. **Connector permissions:**
   - Connector must have read access to repo
   - Verify branch exists and is accessible

## Best Practices

### Webhook Organization

| Scope | Use Case |
|-------|----------|
| Account | Shared templates, account-level configs |
| Org | Organization templates, cross-project resources |
| Project | Project-specific pipelines, services |

### Folder Structure

```
.harness/
├── pipelines/
│   ├── ci.yaml
│   └── deploy.yaml
├── templates/
│   └── docker-build.yaml
├── services/
│   └── api-service.yaml
├── environments/
│   ├── dev.yaml
│   └── prod.yaml
└── infrastructures/
    ├── k8s-dev.yaml
    └── k8s-prod.yaml
```

### Naming Conventions

| Type | Pattern | Example |
|------|---------|---------|
| Project webhook | `{project}_webhook` | `api_service_webhook` |
| Template sync | `template_sync` | `template_sync` |
| Org sync | `org_{purpose}` | `org_pipelines` |

## Instructions

When managing webhooks:

1. **Identify requirements:**
   - What repository needs sync?
   - Which folders contain Harness configs?
   - What scope? (Account, Org, Project)

2. **Create the webhook:**
   - Use appropriate API endpoint
   - Configure correct folder paths
   - Use proper connector reference

3. **Configure Git provider:**
   - Add webhook URL to Git provider
   - Select appropriate events
   - Verify webhook is active

4. **Verify sync:**
   - Check webhook events
   - Confirm entities sync correctly
   - Monitor for errors

5. **Execute via API** with proper authentication
