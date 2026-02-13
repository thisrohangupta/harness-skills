---
name: template-usage
description: Get template reference entities and usage information using the Harness API. Find which pipelines, stages, or steps are using a specific template.
triggers:
  - template usage
  - template references
  - who uses template
  - template dependencies
  - template impact
  - find template usage
  - list template usage
---

# Template Usage Skill

Get template reference entities and usage information using the Harness API.

## Overview

This skill helps platform and DevOps teams:
- Find which entities are using a specific template
- Analyze impact before updating templates
- Track template adoption across projects
- Identify unused templates for cleanup
- Ensure governance compliance

## API Reference

**Endpoint:** `GET /template/api/templates/entitySetupUsage/{templateIdentifier}`

**Documentation:** https://apidocs.harness.io/templates/listtemplateusage

### Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `templateIdentifier` | path | Yes | Template identifier |
| `accountIdentifier` | query | Yes | Account ID |
| `orgIdentifier` | query | No | Organization ID (for org/project templates) |
| `projectIdentifier` | query | No | Project ID (for project templates) |
| `versionLabel` | query | No | Specific version label |
| `isStableTemplate` | query | No | Filter for stable version (default: true) |
| `searchTerm` | query | No | Search within results |
| `pageIndex` | query | No | Page number (default: 0) |
| `pageSize` | query | No | Results per page (default: 100) |
| `allVersions` | query | No | Include all versions (default: false) |
| `templateEntityType` | query | No | Filter by type: Step, Stage, Pipeline, StepGroup |

### Request Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-api-key` | Yes | Harness API key |

## Workflow

### Step 1: Identify the Template

Gather template information:
- Template identifier
- Scope (account, org, project)
- Version (specific or stable)
- Template type (Step, Stage, Pipeline, StepGroup)

### Step 2: Construct API Request

Build the API call based on template scope:

**Account-level template:**
```bash
curl -X GET \
  'https://app.harness.io/template/api/templates/entitySetupUsage/{templateIdentifier}?accountIdentifier={accountId}&isStableTemplate=true&pageSize=100' \
  -H 'x-api-key: {apiKey}'
```

**Org-level template:**
```bash
curl -X GET \
  'https://app.harness.io/template/api/templates/entitySetupUsage/{templateIdentifier}?accountIdentifier={accountId}&orgIdentifier={orgId}&isStableTemplate=true&pageSize=100' \
  -H 'x-api-key: {apiKey}'
```

**Project-level template:**
```bash
curl -X GET \
  'https://app.harness.io/template/api/templates/entitySetupUsage/{templateIdentifier}?accountIdentifier={accountId}&orgIdentifier={orgId}&projectIdentifier={projectId}&isStableTemplate=true&pageSize=100' \
  -H 'x-api-key: {apiKey}'
```

### Step 3: Parse Response

The API returns a list of entities using the template:

```json
{
  "status": "SUCCESS",
  "data": {
    "content": [
      {
        "accountIdentifier": "account123",
        "orgIdentifier": "default",
        "projectIdentifier": "my-project",
        "referredEntity": {
          "type": "PIPELINES",
          "name": "build-and-deploy",
          "entityRef": {
            "accountIdentifier": "account123",
            "orgIdentifier": "default",
            "projectIdentifier": "my-project",
            "identifier": "build_and_deploy"
          }
        },
        "referredByEntity": {
          "type": "TEMPLATE",
          "name": "docker-build-push",
          "entityRef": {
            "accountIdentifier": "account123",
            "identifier": "docker_build_push"
          }
        }
      }
    ],
    "pageIndex": 0,
    "pageSize": 100,
    "totalPages": 1,
    "totalItems": 5,
    "empty": false
  }
}
```

### Step 4: Analyze and Present Results

Organize usage by:
- Entity type (Pipeline, Stage, Step)
- Project/Organization
- Version being used

## Response Format

### Template Usage Report

```markdown
## Template Usage Report

**Template:** docker-build-push
**Type:** Step Template
**Scope:** Account
**Version Analyzed:** Stable (v1.2.0)

### Summary

| Metric | Count |
|--------|-------|
| Total References | 23 |
| Pipelines Using | 18 |
| Stage Templates Using | 3 |
| Pipeline Templates Using | 2 |
| Projects | 8 |
| Organizations | 3 |

### Usage by Organization

| Organization | Projects | References |
|--------------|----------|------------|
| platform-org | 4 | 12 |
| product-org | 3 | 8 |
| devops-org | 1 | 3 |

### Usage by Project

| Project | Org | Pipelines | Templates |
|---------|-----|-----------|-----------|
| api-service | platform-org | 4 | 1 |
| web-frontend | platform-org | 3 | 0 |
| mobile-app | product-org | 3 | 1 |
| data-pipeline | product-org | 2 | 1 |
| infra-deploy | devops-org | 2 | 1 |
| ... | | | |

### Detailed References

#### Pipelines (18)

| Pipeline | Project | Stage Using Template |
|----------|---------|---------------------|
| build-and-deploy | api-service | Build Stage |
| ci-pipeline | api-service | CI Stage |
| release-pipeline | api-service | Build Stage |
| frontend-ci | web-frontend | Build Stage |
| frontend-cd | web-frontend | Deploy Stage |
| ... | | |

#### Templates Referencing This Template (5)

| Template | Type | Scope |
|----------|------|-------|
| full-ci-stage | Stage | platform-org |
| standard-build | Stage | account |
| release-pipeline | Pipeline | product-org |
| ... | | |
```

### Impact Analysis Report

```markdown
## Template Update Impact Analysis

**Template:** docker-build-push
**Current Version:** v1.2.0
**Proposed Change:** Update to v2.0.0 (breaking change)

### Impact Summary

⚠️ **High Impact** - 23 entities will be affected

| Impact Level | Count | Action Required |
|--------------|-------|-----------------|
| Direct (Pipelines) | 18 | Test after update |
| Indirect (Templates) | 5 | Update references |
| Total Affected | 23 | |

### Breaking Change Analysis

The following entities use features being modified:

| Entity | Type | Affected Feature | Required Action |
|--------|------|------------------|-----------------|
| build-and-deploy | Pipeline | `imageName` param | Rename to `image` |
| ci-pipeline | Pipeline | `dockerfilePath` param | Now required |
| full-ci-stage | Stage Template | Inherits changes | Update version ref |

### Recommended Update Strategy

1. **Create v2.0.0** as new version (don't update stable)
2. **Test in non-production:**
   - Update 2-3 pipelines in dev/staging
   - Verify builds succeed
3. **Update dependent templates first:**
   - full-ci-stage
   - standard-build
4. **Gradual rollout:**
   - Week 1: platform-org projects
   - Week 2: product-org projects
   - Week 3: devops-org projects
5. **Mark v2.0.0 as stable** after validation

### Rollback Plan

If issues occur:
1. Revert template to v1.2.0 stable
2. Affected pipelines auto-recover
3. Notify teams via Slack
```

### Unused Template Report

```markdown
## Unused Templates Report

**Scope:** Account
**Analysis Date:** February 2024

### Templates with No References

| Template | Type | Created | Last Modified | Owner |
|----------|------|---------|---------------|-------|
| old-deploy-step | Step | 2022-03-15 | 2022-06-20 | @platform |
| legacy-build | Stage | 2021-11-10 | 2022-01-05 | @devops |
| test-template | Step | 2023-08-22 | 2023-08-22 | @developer |
| deprecated-ci | Pipeline | 2022-05-18 | 2022-07-30 | @ci-team |

### Recommendations

1. **Review for deletion:**
   - `old-deploy-step` - No usage in 18+ months
   - `legacy-build` - No usage in 24+ months

2. **Verify before deletion:**
   - `test-template` - Recently created, may be in development
   - `deprecated-ci` - Check if referenced in documentation

3. **Archive process:**
   - Export template YAML for records
   - Delete from Harness
   - Update documentation
```

## Common Scenarios

### 1. Check Template Usage

```
/template-usage

Which pipelines are using the docker-build-push template?
```

### 2. Impact Analysis

```
/template-usage

I want to update the k8s-deploy template.
Show me what will be affected.
```

### 3. Find Unused Templates

```
/template-usage

Which templates in our account have no references?
```

### 4. Version Usage

```
/template-usage

Which version of the ci-stage template are
different projects using?
```

### 5. Cross-Project Usage

```
/template-usage

Show me all usages of account-level templates
across all projects
```

### 6. Template Dependency Chain

```
/template-usage

Show me the full dependency chain for the
release-pipeline template
```

## Template Scopes

### Account-Level Templates

- Available to all orgs and projects
- Query with only `accountIdentifier`
- Broadest potential impact

### Org-Level Templates

- Available to all projects in the org
- Query with `accountIdentifier` + `orgIdentifier`
- Impact limited to org

### Project-Level Templates

- Available only within the project
- Query with all three identifiers
- Most limited scope

## Template Types

| Type | Description | Common Uses |
|------|-------------|-------------|
| `Step` | Single step definition | Build, test, deploy actions |
| `Stage` | Complete stage with steps | CI stage, CD stage, approval |
| `Pipeline` | Full pipeline | Standard workflows |
| `StepGroup` | Group of related steps | Test suites, deploy sequences |

## API Response Fields

### Entity Reference Object

```json
{
  "referredEntity": {
    "type": "PIPELINES",       // Entity type using the template
    "name": "my-pipeline",     // Display name
    "entityRef": {
      "accountIdentifier": "...",
      "orgIdentifier": "...",
      "projectIdentifier": "...",
      "identifier": "my_pipeline"
    }
  }
}
```

### Entity Types

| Type | Description |
|------|-------------|
| `PIPELINES` | Pipeline using the template |
| `TEMPLATE` | Another template referencing this one |
| `INPUT_SETS` | Input set referencing the template |

## Pagination

For templates with many references:

```bash
# First page
?pageIndex=0&pageSize=100

# Next page
?pageIndex=1&pageSize=100
```

Handle pagination in results:
- Check `totalItems` for total count
- Check `totalPages` for page count
- Iterate through all pages for complete list

## Example API Calls

### Get All Usage for Step Template

```bash
curl -X GET \
  'https://app.harness.io/template/api/templates/entitySetupUsage/docker_build_push?accountIdentifier=abc123&templateEntityType=Step&pageSize=100' \
  -H 'x-api-key: pat.abc123.xyz789'
```

### Get Usage for Specific Version

```bash
curl -X GET \
  'https://app.harness.io/template/api/templates/entitySetupUsage/ci_stage?accountIdentifier=abc123&versionLabel=v1.0.0&isStableTemplate=false&pageSize=100' \
  -H 'x-api-key: pat.abc123.xyz789'
```

### Get Usage Across All Versions

```bash
curl -X GET \
  'https://app.harness.io/template/api/templates/entitySetupUsage/deploy_stage?accountIdentifier=abc123&allVersions=true&pageSize=100' \
  -H 'x-api-key: pat.abc123.xyz789'
```

### Search Within Results

```bash
curl -X GET \
  'https://app.harness.io/template/api/templates/entitySetupUsage/k8s_deploy?accountIdentifier=abc123&searchTerm=production&pageSize=100' \
  -H 'x-api-key: pat.abc123.xyz789'
```

## Example Usage

### Quick Check

```
/template-usage

Who is using the docker-build-push template?
```

### Detailed Analysis

```
/template-usage

Show me detailed usage for the ci-stage template
including which versions each pipeline is using
```

### Pre-Update Check

```
/template-usage

I need to make a breaking change to the k8s-deploy template.
Give me an impact analysis.
```

### Governance Report

```
/template-usage

Generate a template usage report for compliance audit
showing all templates and their adoption
```

## Error Handling

### Common Errors

| Error Code | Description | Solution |
|------------|-------------|----------|
| `TEMPLATE_NOT_FOUND` | Template doesn't exist | Verify template identifier and scope |
| `VERSION_NOT_FOUND` | Version label doesn't exist | Check version label spelling |
| `INVALID_SCOPE` | Wrong scope parameters | Match scope to template location |
| `ACCESS_DENIED` | Cannot access template | Verify template view permissions |
| `PAGINATION_ERROR` | Invalid page parameters | Use valid pageIndex and pageSize |

### API Response Errors

```json
// Common API errors:

// Template not found
{
  "status": "ERROR",
  "code": "TEMPLATE_NOT_FOUND",
  "message": "Template with identifier 'my_template' not found"
}
→ Verify template ID and scope (account/org/project)

// Invalid version
{
  "status": "ERROR",
  "code": "VERSION_NOT_FOUND",
  "message": "Version 'v1.0.0' not found for template"
}
→ Check available versions or use isStableTemplate=true

// Scope mismatch
{
  "status": "ERROR",
  "code": "INVALID_REQUEST",
  "message": "orgIdentifier required for org-level template"
}
→ Include correct scope parameters
```

## Troubleshooting

### No Usage Data Returned

1. **Template actually unused:**
   - Template may have no references
   - Check if recently created
   - Verify not orphaned

2. **Wrong scope:**
   - Query scope must match template scope
   - Account templates need only accountId
   - Project templates need all three identifiers

3. **Version filtering:**
   - isStableTemplate=true only checks stable
   - Use allVersions=true for all references
   - Check specific version with versionLabel

### Incomplete Usage List

1. **Pagination needed:**
   - Default pageSize may be too small
   - Check totalItems vs returned items
   - Iterate through all pages

2. **Cross-scope references:**
   - Account templates may be used in any project
   - Query may need broader scope
   - Check different orgs/projects

3. **Template type filter:**
   - templateEntityType filters results
   - Remove filter to see all usages
   - Check correct type used

### Impact Analysis Issues

1. **Indirect references:**
   - Templates can reference other templates
   - Build full dependency chain
   - Check transitive dependencies

2. **Version complexity:**
   - Different entities may use different versions
   - Track version usage separately
   - Consider version-specific impacts

3. **Breaking changes:**
   - Identify what's changing
   - Map to affected entities
   - Plan staged rollout

### API Performance

1. **Large result sets:**
   - Use pagination for many references
   - Consider filtering options
   - Cache results if needed

2. **Multiple queries needed:**
   - May need multiple API calls
   - Aggregate results client-side
   - Consider batching requests

## Instructions

When checking template usage:

1. **Gather template info:**
   - Template identifier
   - Scope (account/org/project)
   - Version to check (stable or specific)

2. **Make API call:**
   - Construct URL with appropriate parameters
   - Handle pagination for large result sets
   - Filter by type if needed

3. **Analyze results:**
   - Count by entity type
   - Group by project/org
   - Identify patterns

4. **Present findings:**
   - Summary statistics
   - Detailed reference list
   - Impact assessment if updating

5. **Recommend actions:**
   - For updates: rollout strategy
   - For unused: cleanup process
   - For governance: compliance status
