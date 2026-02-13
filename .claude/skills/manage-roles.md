---
name: manage-roles
description: Manage Harness.io Role Assignments via the API. Assign roles to users and groups for access control across accounts, organizations, and projects.
triggers:
  - harness roles
  - manage roles
  - role assignment
  - assign role
  - user permissions
  - rbac
  - access control
  - manage roles api
---

# Manage Roles Skill

Manage Harness.io Role Assignments via the API for access control.

## Overview

Role assignments in Harness control who can access and perform actions on resources. This skill helps you:
- Assign built-in and custom roles to users
- Manage group role assignments
- Configure service account permissions
- Handle multi-scope access control
- Manage roles via the Harness API

## Role Assignment Structure

Every role assignment follows this structure:

```yaml
roleAssignment:
  identifier: <unique_identifier>
  resourceGroupIdentifier: <resource_group_id>
  roleIdentifier: <role_id>
  principal:
    identifier: <user_or_group_id>
    type: USER | USER_GROUP | SERVICE_ACCOUNT
  disabled: false
```

## Built-in Roles

### Account-Level Roles

| Role Identifier | Description |
|-----------------|-------------|
| `_account_admin` | Full account access |
| `_account_viewer` | Read-only account access |
| `_organization_admin` | Manage all orgs |
| `_organization_viewer` | View all orgs |

### Organization-Level Roles

| Role Identifier | Description |
|-----------------|-------------|
| `_organization_admin` | Full org access |
| `_organization_viewer` | Read-only org access |
| `_project_admin` | Manage all projects |
| `_project_viewer` | View all projects |

### Project-Level Roles

| Role Identifier | Description |
|-----------------|-------------|
| `_project_admin` | Full project access |
| `_project_viewer` | Read-only project access |
| `_pipeline_executor` | Execute pipelines |
| `_pipeline_editor` | Create/edit pipelines |

### Module-Specific Roles

| Role Identifier | Module | Description |
|-----------------|--------|-------------|
| `_ci_admin` | CI | Full CI access |
| `_ci_developer` | CI | CI development access |
| `_cd_admin` | CD | Full CD access |
| `_cd_developer` | CD | CD development access |
| `_ff_admin` | Feature Flags | Full FF access |
| `_ccm_admin` | Cloud Cost | Full CCM access |
| `_sto_admin` | Security | Full STO access |

## Built-in Resource Groups

| Resource Group | Scope | Description |
|---------------|-------|-------------|
| `_all_resources_including_child_scopes` | Any | All resources including child scopes |
| `_all_account_level_resources` | Account | All account resources |
| `_all_organization_level_resources` | Org | All org resources |
| `_all_project_level_resources` | Project | All project resources |

## Role Assignment Examples

### Assign Account Admin to User

```yaml
roleAssignment:
  identifier: admin_john_doe
  resourceGroupIdentifier: _all_resources_including_child_scopes
  roleIdentifier: _account_admin
  principal:
    identifier: john.doe@example.com
    type: USER
  disabled: false
```

### Assign Project Viewer to Group

```yaml
roleAssignment:
  identifier: viewers_dev_team
  resourceGroupIdentifier: _all_project_level_resources
  roleIdentifier: _project_viewer
  principal:
    identifier: dev_team
    type: USER_GROUP
  disabled: false
```

### Assign Pipeline Executor to Service Account

```yaml
roleAssignment:
  identifier: sa_pipeline_executor
  resourceGroupIdentifier: _all_project_level_resources
  roleIdentifier: _pipeline_executor
  principal:
    identifier: ci_service_account
    type: SERVICE_ACCOUNT
  disabled: false
```

### Assign CD Developer Role

```yaml
roleAssignment:
  identifier: cd_dev_jane
  resourceGroupIdentifier: _all_project_level_resources
  roleIdentifier: _cd_developer
  principal:
    identifier: jane.smith@example.com
    type: USER
  disabled: false
```

## Creating Role Assignments via API

### API Reference

**Account-level:** `POST /v1/role-assignments`
**Org-level:** `POST /v1/orgs/{org}/role-assignments`
**Project-level:** `POST /v1/orgs/{org}/projects/{project}/role-assignments`

**Documentation:** https://apidocs.harness.io/tag/Account-Role-Assignments

### Request Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-api-key` | Yes | Harness API key |
| `Harness-Account` | Yes | Account identifier |
| `Content-Type` | Yes | `application/json` |

### Create Account-Level Role Assignment

```bash
curl -X POST \
  'https://app.harness.io/v1/role-assignments' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}' \
  -H 'Content-Type: application/json' \
  -d '{
    "identifier": "admin_john_doe",
    "resource_group_identifier": "_all_resources_including_child_scopes",
    "role_identifier": "_account_admin",
    "principal": {
      "identifier": "john.doe@example.com",
      "type": "USER"
    },
    "disabled": false
  }'
```

### Create Org-Level Role Assignment

```bash
curl -X POST \
  'https://app.harness.io/v1/orgs/{org}/role-assignments' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}' \
  -H 'Content-Type: application/json' \
  -d '{
    "identifier": "org_admin_team",
    "resource_group_identifier": "_all_organization_level_resources",
    "role_identifier": "_organization_admin",
    "principal": {
      "identifier": "platform_team",
      "type": "USER_GROUP"
    },
    "disabled": false
  }'
```

### Create Project-Level Role Assignment

```bash
curl -X POST \
  'https://app.harness.io/v1/orgs/{org}/projects/{project}/role-assignments' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}' \
  -H 'Content-Type: application/json' \
  -d '{
    "identifier": "dev_pipeline_executor",
    "resource_group_identifier": "_all_project_level_resources",
    "role_identifier": "_pipeline_executor",
    "principal": {
      "identifier": "developers",
      "type": "USER_GROUP"
    },
    "disabled": false
  }'
```

### Response

**Success (201 Created):**

```json
{
  "role_assignment": {
    "identifier": "admin_john_doe",
    "resource_group_identifier": "_all_resources_including_child_scopes",
    "role_identifier": "_account_admin",
    "principal": {
      "identifier": "john.doe@example.com",
      "type": "USER"
    },
    "scope": {
      "account": "abc123"
    },
    "disabled": false,
    "created": 1707500000000,
    "updated": 1707500000000
  }
}
```

### List Role Assignments

**Endpoint:** `GET /v1/role-assignments`

```bash
curl -X GET \
  'https://app.harness.io/v1/role-assignments?page=0&limit=100' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}'
```

### Delete Role Assignment

**Endpoint:** `DELETE /v1/role-assignments/{role-assignment}`

```bash
curl -X DELETE \
  'https://app.harness.io/v1/role-assignments/{roleAssignmentId}' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}'
```

## Custom Roles

### Create Custom Role

```bash
curl -X POST \
  'https://app.harness.io/v1/roles' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}' \
  -H 'Content-Type: application/json' \
  -d '{
    "identifier": "custom_deployer",
    "name": "Custom Deployer",
    "description": "Can execute pipelines and view services",
    "permissions": [
      "core_pipeline_execute",
      "core_pipeline_view",
      "core_service_view",
      "core_environment_view"
    ]
  }'
```

### Common Permissions

| Permission | Description |
|------------|-------------|
| `core_pipeline_view` | View pipelines |
| `core_pipeline_edit` | Edit pipelines |
| `core_pipeline_delete` | Delete pipelines |
| `core_pipeline_execute` | Execute pipelines |
| `core_service_view` | View services |
| `core_service_edit` | Edit services |
| `core_environment_view` | View environments |
| `core_environment_edit` | Edit environments |
| `core_secret_view` | View secrets |
| `core_secret_edit` | Edit secrets |
| `core_connector_view` | View connectors |
| `core_connector_edit` | Edit connectors |

## Custom Resource Groups

### Create Custom Resource Group

```bash
curl -X POST \
  'https://app.harness.io/v1/resource-groups' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}' \
  -H 'Content-Type: application/json' \
  -d '{
    "identifier": "production_resources",
    "name": "Production Resources",
    "description": "Production environment and related resources",
    "included_scopes": [
      {
        "filter": "INCLUDING_CHILD_SCOPES",
        "account_identifier": "{accountId}"
      }
    ],
    "resource_filter": {
      "resources": [
        {
          "resource_type": "ENVIRONMENT",
          "identifiers": ["prod", "production"]
        },
        {
          "resource_type": "SERVICE",
          "attribute_filter": {
            "attribute_name": "identifier",
            "attribute_values": ["*"]
          }
        }
      ]
    }
  }'
```

## Common Scenarios

### Onboard New Developer

```bash
# 1. Add user to developer group
# 2. Assign project-level developer role
curl -X POST \
  'https://app.harness.io/v1/orgs/{org}/projects/{project}/role-assignments' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}' \
  -H 'Content-Type: application/json' \
  -d '{
    "identifier": "dev_new_user",
    "resource_group_identifier": "_all_project_level_resources",
    "role_identifier": "_cd_developer",
    "principal": {
      "identifier": "new.developer@example.com",
      "type": "USER"
    }
  }'
```

### Grant CI Access to Service Account

```bash
curl -X POST \
  'https://app.harness.io/v1/orgs/{org}/projects/{project}/role-assignments' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}' \
  -H 'Content-Type: application/json' \
  -d '{
    "identifier": "sa_ci_access",
    "resource_group_identifier": "_all_project_level_resources",
    "role_identifier": "_ci_admin",
    "principal": {
      "identifier": "github_actions_sa",
      "type": "SERVICE_ACCOUNT"
    }
  }'
```

### Read-Only Access for Auditors

```bash
curl -X POST \
  'https://app.harness.io/v1/role-assignments' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}' \
  -H 'Content-Type: application/json' \
  -d '{
    "identifier": "auditor_readonly",
    "resource_group_identifier": "_all_resources_including_child_scopes",
    "role_identifier": "_account_viewer",
    "principal": {
      "identifier": "auditors",
      "type": "USER_GROUP"
    }
  }'
```

### Production-Only Access

```bash
# First create a production resource group, then assign role
curl -X POST \
  'https://app.harness.io/v1/orgs/{org}/projects/{project}/role-assignments' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}' \
  -H 'Content-Type: application/json' \
  -d '{
    "identifier": "prod_deployers",
    "resource_group_identifier": "production_resources",
    "role_identifier": "_pipeline_executor",
    "principal": {
      "identifier": "prod_team",
      "type": "USER_GROUP"
    }
  }'
```

## Best Practices

### Role Assignment Strategy

1. **Use groups over individual users** - Easier to manage
2. **Follow least privilege** - Assign minimum needed permissions
3. **Use scoped roles** - Project > Org > Account
4. **Regular audits** - Review role assignments periodically
5. **Document assignments** - Track who has what access

### Naming Conventions

| Type | Pattern | Example |
|------|---------|---------|
| User assignment | `{role}_{username}` | `admin_john_doe` |
| Group assignment | `{role}_{group}` | `viewers_dev_team` |
| Service account | `sa_{purpose}` | `sa_pipeline_executor` |

### Security Considerations

- **Separate production access** - Use dedicated resource groups
- **Rotate service account tokens** - Regular credential rotation
- **Monitor privileged access** - Audit admin role usage
- **Emergency access process** - Document break-glass procedures

## Error Handling

### Common Errors

| Error Code | Description | Solution |
|------------|-------------|----------|
| `INVALID_REQUEST` | Malformed request or missing fields | Validate request structure |
| `DUPLICATE_IDENTIFIER` | Role assignment already exists | Check for existing assignment |
| `ROLE_NOT_FOUND` | Invalid role identifier | Use valid built-in or custom role |
| `RESOURCE_GROUP_NOT_FOUND` | Invalid resource group | Verify resource group exists |
| `PRINCIPAL_NOT_FOUND` | User/group/SA doesn't exist | Create principal first |

### Validation Errors

```json
// Common role assignment issues:

// Invalid principal type
{
  "principal": {
    "identifier": "user@example.com",
    "type": "user"  // Wrong (case-sensitive)
    "type": "USER"  // Correct
  }
}

// Invalid role identifier
{
  "role_identifier": "account_admin"  // Wrong
  "role_identifier": "_account_admin"  // Correct (built-in roles start with _)
}
```

## Troubleshooting

### User Can't Access Resources

1. **Check role assignment exists:**
   - Verify at correct scope (account/org/project)
   - Check assignment isn't disabled

2. **Verify role permissions:**
   - Built-in roles have fixed permissions
   - Custom roles may be missing permissions

3. **Check resource group scope:**
   - Ensure resource group includes target resources
   - Verify scope matches user needs

### Permission Denied Errors

1. **Check inheritance:**
   - Account roles inherit to org/project
   - Org roles inherit to project

2. **Verify specific permissions:**
   ```bash
   # List user's effective permissions
   curl -X GET \
     'https://app.harness.io/authz/api/acl?principal={userId}&...' \
     -H 'x-api-key: {apiKey}'
   ```

### Service Account Issues

1. **Token permissions:**
   - SA inherits permissions from role assignments
   - Check token isn't expired

2. **Scope limitations:**
   - SA can only act within assigned scope
   - Project SA can't access other projects

### Role Assignment Not Taking Effect

1. **Clear cache:**
   - Logout and login for UI changes
   - API changes may take a few minutes

2. **Check conflicts:**
   - Multiple role assignments combine
   - Most permissive wins for a resource

## Instructions

When managing roles:

1. **Identify requirements:**
   - Who needs access? (User, Group, Service Account)
   - What level of access? (Admin, Developer, Viewer)
   - What scope? (Account, Org, Project)
   - What resources? (All or specific)

2. **Select appropriate role:**
   - Use built-in roles when possible
   - Create custom roles for specific needs
   - Apply least privilege principle

3. **Configure resource scope:**
   - Use built-in resource groups when possible
   - Create custom resource groups for fine-grained control

4. **Execute via API** with proper authentication

5. **Document the assignment** for audit purposes
