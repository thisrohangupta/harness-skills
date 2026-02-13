---
name: create-environment
description: Generate Harness.io Environment YAML definitions and optionally create them via the Harness API. Environments define where you deploy (dev, staging, prod).
triggers:
  - harness environment
  - create environment
  - deployment environment
  - environment yaml
  - create env
  - create environment api
---

# Create Environment Skill

Generate Harness.io Environment YAML definitions and create them via the API.

## Overview

Environments in Harness represent **where you deploy** - the target deployment destination like development, staging, or production. This skill helps you:
- Define environments with proper classification
- Configure environment-level overrides
- Set up environment variables and configs
- Create environments via the Harness API

## Environment Structure

Every Harness environment follows this structure:

```yaml
environment:
  identifier: <unique_identifier>
  name: <display_name>
  description: <optional_description>
  tags:
    key: value
  type: PreProduction | Production
  orgIdentifier: <org_id>
  projectIdentifier: <project_id>
  variables: []
  overrides: {}
```

## Environment Types

### PreProduction

For development, testing, staging, and QA environments:

```yaml
environment:
  identifier: dev
  name: Development
  description: Development environment for testing
  type: PreProduction
  tags:
    tier: development
```

### Production

For production environments (enables additional safeguards):

```yaml
environment:
  identifier: prod
  name: Production
  description: Production environment
  type: Production
  tags:
    tier: production
    critical: "true"
```

## Environment Variables

Define environment-specific variables:

```yaml
environment:
  identifier: staging
  name: Staging
  type: PreProduction
  variables:
    - name: LOG_LEVEL
      type: String
      value: debug
    - name: REPLICA_COUNT
      type: Number
      value: 2
    - name: DB_HOST
      type: String
      value: staging-db.example.com
    - name: DB_PASSWORD
      type: Secret
      value: <+secrets.getValue("staging_db_password")>
```

## Environment Overrides

Override service configurations per environment:

```yaml
environment:
  identifier: prod
  name: Production
  type: Production
  overrides:
    manifests:
      - manifest:
          identifier: values_override
          type: Values
          spec:
            store:
              type: Github
              spec:
                connectorRef: github_connector
                repoName: my-app
                branch: main
                paths:
                  - values-prod.yaml
    configFiles:
      - configFile:
          identifier: app_config
          spec:
            store:
              type: Harness
              spec:
                files:
                  - /Config/prod/application.properties
```

## Complete Environment Examples

### Development Environment

```yaml
environment:
  identifier: dev
  name: Development
  description: Development environment for feature testing
  type: PreProduction
  orgIdentifier: default
  projectIdentifier: my_project
  tags:
    tier: development
    team: platform
  variables:
    - name: LOG_LEVEL
      type: String
      value: debug
    - name: ENABLE_DEBUG_MODE
      type: String
      value: "true"
    - name: REPLICA_COUNT
      type: Number
      value: 1
```

### Staging Environment

```yaml
environment:
  identifier: staging
  name: Staging
  description: Pre-production staging environment
  type: PreProduction
  orgIdentifier: default
  projectIdentifier: my_project
  tags:
    tier: staging
    team: platform
  variables:
    - name: LOG_LEVEL
      type: String
      value: info
    - name: ENABLE_DEBUG_MODE
      type: String
      value: "false"
    - name: REPLICA_COUNT
      type: Number
      value: 2
    - name: DB_HOST
      type: String
      value: staging-db.internal
    - name: DB_PASSWORD
      type: Secret
      value: <+secrets.getValue("staging_db_pass")>
```

### Production Environment

```yaml
environment:
  identifier: prod
  name: Production
  description: Production environment - requires approval
  type: Production
  orgIdentifier: default
  projectIdentifier: my_project
  tags:
    tier: production
    critical: "true"
    pci_compliant: "true"
  variables:
    - name: LOG_LEVEL
      type: String
      value: warn
    - name: ENABLE_DEBUG_MODE
      type: String
      value: "false"
    - name: REPLICA_COUNT
      type: Number
      value: 5
    - name: DB_HOST
      type: String
      value: prod-db.internal
    - name: DB_PASSWORD
      type: Secret
      value: <+secrets.getValue("prod_db_pass")>
    - name: ENABLE_MONITORING
      type: String
      value: "true"
```

### Multi-Region Production

```yaml
environment:
  identifier: prod_us_east
  name: Production US East
  description: Production environment in US East region
  type: Production
  tags:
    tier: production
    region: us-east-1
    cloud: aws
  variables:
    - name: REGION
      type: String
      value: us-east-1
    - name: AVAILABILITY_ZONES
      type: String
      value: us-east-1a,us-east-1b,us-east-1c
```

## Creating Environments via API

### API Reference

**Endpoint:** `POST /v1/orgs/{org}/projects/{project}/environments`

**Documentation:** https://apidocs.harness.io/tag/Project-Environments

### Request Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-api-key` | Yes | Harness API key |
| `Harness-Account` | Yes | Account identifier |
| `Content-Type` | Yes | `application/json` |

### Path Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `org` | Yes | Organization identifier |
| `project` | Yes | Project identifier |

### Request Body

```json
{
  "identifier": "dev",
  "name": "Development",
  "description": "Development environment",
  "type": "PreProduction",
  "tags": {
    "tier": "development"
  },
  "yaml": "environment:\n  identifier: dev\n  name: Development\n  type: PreProduction\n  ..."
}
```

### Example: Create Environment

```bash
curl -X POST \
  'https://app.harness.io/v1/orgs/{org}/projects/{project}/environments' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}' \
  -H 'Content-Type: application/json' \
  -d '{
    "identifier": "staging",
    "name": "Staging",
    "description": "Pre-production staging environment",
    "type": "PreProduction",
    "tags": {
      "tier": "staging"
    }
  }'
```

### Example: Create Environment with YAML

```bash
curl -X POST \
  'https://app.harness.io/v1/orgs/default/projects/my_project/environments' \
  -H 'x-api-key: pat.xxxx.yyyy.zzzz' \
  -H 'Harness-Account: abc123' \
  -H 'Content-Type: application/json' \
  -d '{
    "identifier": "prod",
    "name": "Production",
    "description": "Production environment",
    "type": "Production",
    "tags": {
      "tier": "production",
      "critical": "true"
    },
    "yaml": "environment:\n  identifier: prod\n  name: Production\n  type: Production\n  variables:\n    - name: REPLICA_COUNT\n      type: Number\n      value: 5"
  }'
```

### Response

**Success (201 Created):**

```json
{
  "environment": {
    "identifier": "staging",
    "name": "Staging",
    "description": "Pre-production staging environment",
    "type": "PreProduction",
    "org": "default",
    "project": "my_project",
    "created": 1707500000000,
    "updated": 1707500000000
  }
}
```

### Update Existing Environment

**Endpoint:** `PUT /v1/orgs/{org}/projects/{project}/environments/{environment}`

```bash
curl -X PUT \
  'https://app.harness.io/v1/orgs/{org}/projects/{project}/environments/{envIdentifier}' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}' \
  -H 'Content-Type: application/json' \
  -d '{
    "identifier": "staging",
    "name": "Staging (Updated)",
    "type": "PreProduction",
    "yaml": "..."
  }'
```

### Environment Scopes

Environments can be created at different scopes:

**Project-level (most common):**
```
POST /v1/orgs/{org}/projects/{project}/environments
```

**Org-level (shared across projects):**
```
POST /v1/orgs/{org}/environments
```

**Account-level (shared across orgs):**
```
POST /v1/environments
```

## Environment Groups

Group related environments for batch operations:

```yaml
environmentGroup:
  identifier: non_prod
  name: Non-Production
  description: All non-production environments
  orgIdentifier: default
  projectIdentifier: my_project
  envIdentifiers:
    - dev
    - staging
    - qa
```

## Best Practices

### Naming Conventions

| Environment | Identifier | Type |
|-------------|------------|------|
| Development | `dev` | PreProduction |
| Testing | `test` | PreProduction |
| QA | `qa` | PreProduction |
| Staging | `staging` | PreProduction |
| UAT | `uat` | PreProduction |
| Production | `prod` | Production |
| DR | `dr` | Production |

### Variable Strategy

1. **Common variables** - Define at org/account level
2. **Environment-specific** - Override at environment level
3. **Secrets** - Always use secret references
4. **Runtime inputs** - Use `<+input>` for flexibility

### Production Safeguards

For production environments:
- Set `type: Production` for approval enforcement
- Add critical tags for visibility
- Configure freeze windows
- Enable deployment notifications

## Error Handling

### Common Errors

| Error Code | Description | Solution |
|------------|-------------|----------|
| `INVALID_REQUEST` | Malformed YAML or missing fields | Validate YAML structure |
| `DUPLICATE_IDENTIFIER` | Environment with same ID exists | Use unique identifier |
| `INVALID_ENVIRONMENT_TYPE` | Invalid type value | Use `PreProduction` or `Production` |
| `SECRET_NOT_FOUND` | Referenced secret doesn't exist | Create secret first |

### Validation Errors

```yaml
# Common environment validation issues:

# Invalid environment type
type: production  # Wrong (case-sensitive)
type: Production  # Correct

# Invalid variable type
variables:
  - name: count
    type: number  # Wrong
    type: Number  # Correct

# Missing secret reference
variables:
  - name: password
    type: Secret
    value: my_password  # Wrong (should be secret reference)
    value: <+secrets.getValue("my_password")>  # Correct
```

## Troubleshooting

### Environment Not Available in Pipeline

1. **Check environment scope:**
   - Project environments: Available in that project only
   - Org environments: Available across projects in org
   - Account environments: Available everywhere

2. **Verify environment type:**
   - Production environments may have additional restrictions
   - Check for freeze windows affecting the environment

### Variable Resolution Issues

1. **Check variable syntax:**
   ```yaml
   # Reference in pipeline
   <+env.variables.LOG_LEVEL>
   ```

2. **Verify variable exists:**
   - Check environment YAML for variable definition
   - Ensure variable name matches exactly (case-sensitive)

### Override Not Applied

1. **Verify override structure:**
   - Overrides must match service manifest structure
   - Check file paths are correct

2. **Check precedence:**
   - Environment overrides take precedence over service defaults
   - Infrastructure overrides take precedence over environment

### Production Safeguards

1. **Approval requirements:**
   - Production type environments may require approvals
   - Check governance policies

2. **Freeze windows:**
   - Verify no active freeze affects the environment
   - Check freeze scope configuration

## Instructions

When creating an environment:

1. **Identify requirements:**
   - What type? (PreProduction or Production)
   - What scope? (Project, Org, Account)
   - Environment-specific variables needed?
   - Any service overrides required?

2. **Generate valid YAML:**
   - Use correct identifier patterns
   - Set appropriate type
   - Include relevant tags

3. **Configure variables:**
   - Use appropriate types (String, Number, Secret)
   - Reference secrets properly
   - Consider environment hierarchy

4. **Output the environment YAML** in a code block

5. **Optionally create via API** if the user requests it
