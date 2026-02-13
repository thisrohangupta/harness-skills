---
name: create-freeze
description: Generate Harness.io Deployment Freeze YAML definitions and optionally create them via the Harness API. Freeze windows prevent deployments during specified periods.
triggers:
  - harness freeze
  - create freeze
  - deployment freeze
  - freeze window
  - change freeze
  - blackout window
  - create freeze api
---

# Create Freeze Skill

Generate Harness.io Deployment Freeze YAML definitions and create them via the API.

## Overview

Deployment Freezes in Harness prevent deployments during specified time windows. They're essential for:
- Change management during critical periods
- Holiday and maintenance windows
- Compliance requirements
- Production stability during high-traffic events
- Creating freezes via the Harness API

## Freeze Structure

Every freeze follows this structure:

```yaml
freeze:
  identifier: <unique_identifier>
  name: <display_name>
  description: <optional_description>
  orgIdentifier: <org_id>
  projectIdentifier: <project_id>
  status: Enabled | Disabled
  entityConfigs:
    - name: <rule_name>
      entities:
        - type: <entity_type>
          filterType: All | Equals | NotEquals | StartsWith | EndsWith | Contains
          entityRefs:
            - <entity_ref>
  windows:
    - timeZone: <timezone>
      startTime: <start_datetime>
      duration: <duration> | endTime: <end_datetime>
      recurrence:
        type: Daily | Weekly | Monthly | Yearly
```

## Freeze Examples

### Simple Time-Based Freeze

```yaml
freeze:
  identifier: holiday_freeze
  name: Holiday Freeze
  description: No deployments during holiday period
  status: Enabled
  orgIdentifier: default
  projectIdentifier: my_project
  entityConfigs:
    - name: All Services
      entities:
        - type: Service
          filterType: All
        - type: EnvType
          filterType: Equals
          entityRefs:
            - Production
  windows:
    - timeZone: America/New_York
      startTime: 2024-12-23 00:00 AM
      endTime: 2024-12-26 11:59 PM
```

### Production-Only Freeze

```yaml
freeze:
  identifier: prod_freeze
  name: Production Freeze
  description: Freeze all production deployments
  status: Enabled
  orgIdentifier: default
  projectIdentifier: my_project
  entityConfigs:
    - name: Production Environment
      entities:
        - type: Service
          filterType: All
        - type: EnvType
          filterType: Equals
          entityRefs:
            - Production
  windows:
    - timeZone: UTC
      startTime: 2024-03-01 18:00 PM
      duration: 4h
```

### Recurring Weekly Freeze

```yaml
freeze:
  identifier: weekend_freeze
  name: Weekend Freeze
  description: No deployments on weekends
  status: Enabled
  entityConfigs:
    - name: All Environments
      entities:
        - type: Service
          filterType: All
        - type: EnvType
          filterType: All
  windows:
    - timeZone: America/Los_Angeles
      startTime: 2024-01-06 00:00 AM
      endTime: 2024-01-07 11:59 PM
      recurrence:
        type: Weekly
        spec:
          until: 2024-12-31 11:59 PM
```

### Service-Specific Freeze

```yaml
freeze:
  identifier: critical_service_freeze
  name: Critical Service Freeze
  description: Freeze deployments for payment service
  status: Enabled
  entityConfigs:
    - name: Payment Service Only
      entities:
        - type: Service
          filterType: Equals
          entityRefs:
            - payment_service
            - checkout_service
        - type: EnvType
          filterType: All
  windows:
    - timeZone: UTC
      startTime: 2024-03-15 00:00 AM
      duration: 24h
```

### Environment-Specific Freeze

```yaml
freeze:
  identifier: staging_freeze
  name: Staging Freeze
  description: Freeze staging during load testing
  status: Enabled
  entityConfigs:
    - name: Staging Environment
      entities:
        - type: Service
          filterType: All
        - type: Env
          filterType: Equals
          entityRefs:
            - staging
  windows:
    - timeZone: UTC
      startTime: 2024-03-10 08:00 AM
      duration: 8h
```

### Pipeline-Specific Freeze

```yaml
freeze:
  identifier: pipeline_freeze
  name: Deployment Pipeline Freeze
  description: Freeze specific deployment pipelines
  status: Enabled
  entityConfigs:
    - name: Production Pipelines
      entities:
        - type: Pipeline
          filterType: Equals
          entityRefs:
            - prod_deploy_pipeline
            - release_pipeline
  windows:
    - timeZone: America/New_York
      startTime: 2024-03-20 17:00 PM
      duration: 12h
```

### Monthly Maintenance Window

```yaml
freeze:
  identifier: maintenance_window
  name: Monthly Maintenance
  description: Monthly maintenance freeze - first Sunday
  status: Enabled
  entityConfigs:
    - name: All Production
      entities:
        - type: Service
          filterType: All
        - type: EnvType
          filterType: Equals
          entityRefs:
            - Production
  windows:
    - timeZone: UTC
      startTime: 2024-01-07 02:00 AM
      duration: 4h
      recurrence:
        type: Monthly
        spec:
          until: 2024-12-31 11:59 PM
```

### Global Freeze

```yaml
freeze:
  identifier: global_freeze
  name: Global Deployment Freeze
  description: Company-wide deployment freeze
  status: Enabled
  entityConfigs:
    - name: Everything
      entities:
        - type: Service
          filterType: All
        - type: EnvType
          filterType: All
        - type: Pipeline
          filterType: All
  windows:
    - timeZone: UTC
      startTime: 2024-12-31 18:00 PM
      endTime: 2025-01-02 06:00 AM
```

## Entity Types

| Type | Description | Filter Options |
|------|-------------|----------------|
| `Service` | Services | All, Equals, NotEquals, StartsWith, EndsWith, Contains |
| `Env` | Specific environments | All, Equals, NotEquals |
| `EnvType` | Environment types | All, Equals (Production, PreProduction) |
| `Pipeline` | Pipelines | All, Equals, NotEquals |
| `Org` | Organizations | All, Equals |
| `Project` | Projects | All, Equals |

## Filter Types

| Filter | Description | Requires entityRefs |
|--------|-------------|-------------------|
| `All` | Match all entities | No |
| `Equals` | Exact match | Yes |
| `NotEquals` | Exclude specific | Yes |
| `StartsWith` | Prefix match | Yes |
| `EndsWith` | Suffix match | Yes |
| `Contains` | Substring match | Yes |

## Window Configuration

### Duration Format

- `30m` - 30 minutes
- `2h` - 2 hours
- `1d` - 1 day
- `1w` - 1 week

### Recurrence Types

| Type | Description |
|------|-------------|
| `Daily` | Every day at same time |
| `Weekly` | Every week on same day |
| `Monthly` | Every month on same date |
| `Yearly` | Every year on same date |

## Creating Freezes via API

### API Reference

**Endpoint:** `POST /ng/api/freeze`

**Documentation:** https://apidocs.harness.io/tag/Freeze-CRUD

### Request Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-api-key` | Yes | Harness API key |
| `Content-Type` | Yes | `application/yaml` or `application/json` |

### Query Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `accountIdentifier` | Yes | Account ID |
| `orgIdentifier` | No | Organization ID (for org/project scoped) |
| `projectIdentifier` | No | Project ID (for project scoped) |

### Example: Create Freeze (YAML)

```bash
curl -X POST \
  'https://app.harness.io/ng/api/freeze?accountIdentifier={accountId}&orgIdentifier={org}&projectIdentifier={project}' \
  -H 'x-api-key: {apiKey}' \
  -H 'Content-Type: application/yaml' \
  -d 'freeze:
  identifier: holiday_freeze
  name: Holiday Freeze
  status: Enabled
  entityConfigs:
    - name: All Services
      entities:
        - type: Service
          filterType: All
        - type: EnvType
          filterType: Equals
          entityRefs:
            - Production
  windows:
    - timeZone: America/New_York
      startTime: 2024-12-23 00:00 AM
      endTime: 2024-12-26 11:59 PM'
```

### Example: Create Freeze (JSON)

```bash
curl -X POST \
  'https://app.harness.io/ng/api/freeze?accountIdentifier={accountId}&orgIdentifier={org}&projectIdentifier={project}' \
  -H 'x-api-key: {apiKey}' \
  -H 'Content-Type: application/json' \
  -d '{
    "freeze": {
      "identifier": "holiday_freeze",
      "name": "Holiday Freeze",
      "status": "Enabled",
      "entityConfigs": [
        {
          "name": "All Services",
          "entities": [
            {"type": "Service", "filterType": "All"},
            {"type": "EnvType", "filterType": "Equals", "entityRefs": ["Production"]}
          ]
        }
      ],
      "windows": [
        {
          "timeZone": "America/New_York",
          "startTime": "2024-12-23 00:00 AM",
          "endTime": "2024-12-26 11:59 PM"
        }
      ]
    }
  }'
```

### Response

**Success (200 OK):**

```json
{
  "status": "SUCCESS",
  "data": {
    "accountId": "abc123",
    "orgIdentifier": "default",
    "projectIdentifier": "my_project",
    "identifier": "holiday_freeze",
    "name": "Holiday Freeze",
    "status": "Enabled",
    "freezeScope": "project",
    "windows": [...],
    "createdAt": 1707500000000,
    "lastUpdatedAt": 1707500000000
  }
}
```

### Update Freeze

**Endpoint:** `PUT /ng/api/freeze/{freezeIdentifier}`

```bash
curl -X PUT \
  'https://app.harness.io/ng/api/freeze/{freezeId}?accountIdentifier={accountId}&orgIdentifier={org}&projectIdentifier={project}' \
  -H 'x-api-key: {apiKey}' \
  -H 'Content-Type: application/yaml' \
  -d 'freeze:
  identifier: holiday_freeze
  name: Holiday Freeze (Extended)
  status: Enabled
  ...'
```

### Enable/Disable Freeze

**Endpoint:** `POST /ng/api/freeze/updateFreezeStatus`

```bash
curl -X POST \
  'https://app.harness.io/ng/api/freeze/updateFreezeStatus?accountIdentifier={accountId}&orgIdentifier={org}&projectIdentifier={project}' \
  -H 'x-api-key: {apiKey}' \
  -H 'Content-Type: application/json' \
  -d '{
    "freezeIdentifiers": ["holiday_freeze"],
    "status": "Disabled"
  }'
```

### Delete Freeze

**Endpoint:** `DELETE /ng/api/freeze/{freezeIdentifier}`

```bash
curl -X DELETE \
  'https://app.harness.io/ng/api/freeze/{freezeId}?accountIdentifier={accountId}&orgIdentifier={org}&projectIdentifier={project}' \
  -H 'x-api-key: {apiKey}'
```

### List Freezes

**Endpoint:** `GET /ng/api/freeze`

```bash
curl -X GET \
  'https://app.harness.io/ng/api/freeze?accountIdentifier={accountId}&orgIdentifier={org}&projectIdentifier={project}' \
  -H 'x-api-key: {apiKey}'
```

## Global Freeze

Create account-level freeze affecting all orgs/projects:

```bash
curl -X POST \
  'https://app.harness.io/ng/api/freeze/manageGlobalFreeze?accountIdentifier={accountId}' \
  -H 'x-api-key: {apiKey}' \
  -H 'Content-Type: application/yaml' \
  -d 'freeze:
  identifier: global_freeze
  name: Global Freeze
  status: Enabled
  windows:
    - timeZone: UTC
      startTime: 2024-12-31 00:00 AM
      endTime: 2025-01-01 11:59 PM'
```

## Best Practices

### Naming Conventions

| Purpose | Identifier Pattern | Example |
|---------|-------------------|---------|
| Holiday | `holiday_{year}` | `holiday_2024` |
| Maintenance | `maintenance_{schedule}` | `maintenance_monthly` |
| Event | `{event}_freeze` | `blackfriday_freeze` |
| Emergency | `emergency_{date}` | `emergency_20240315` |

### Planning Freeze Windows

1. **Communicate early** - Announce freezes to all teams
2. **Allow exceptions** - Have process for emergency deployments
3. **Test before freeze** - Complete deployments before window
4. **Monitor during freeze** - Watch for issues requiring rollback
5. **Document exemptions** - Track any freeze overrides

### Freeze Scope Strategy

| Scope | Use Case |
|-------|----------|
| Account | Company-wide events (holidays, outages) |
| Org | Department-specific freezes |
| Project | Application-specific windows |

## Error Handling

### Common Errors

| Error Code | Description | Solution |
|------------|-------------|----------|
| `INVALID_REQUEST` | Malformed YAML or missing fields | Validate YAML structure |
| `DUPLICATE_IDENTIFIER` | Freeze with same ID exists | Use unique identifier |
| `INVALID_DATETIME` | Invalid date/time format | Use `YYYY-MM-DD HH:MM AM/PM` |
| `INVALID_TIMEZONE` | Unknown timezone | Use valid IANA timezone |
| `INVALID_ENTITY_TYPE` | Unsupported entity type | Use Service, Env, EnvType, Pipeline |

### Validation Errors

```yaml
# Common freeze validation issues:

# Invalid datetime format
windows:
  - startTime: 2024-12-23T00:00:00  # Wrong format
    startTime: 2024-12-23 00:00 AM  # Correct

# Invalid timezone
windows:
  - timeZone: EST  # Wrong - use IANA
    timeZone: America/New_York  # Correct

# Invalid entity type
entities:
  - type: environment  # Wrong (case-sensitive)
    type: Env  # Correct
```

## Troubleshooting

### Freeze Not Blocking Deployments

1. **Check freeze is enabled:**
   ```yaml
   status: Enabled  # Must be Enabled, not Disabled
   ```

2. **Verify window timing:**
   - Check timezone is correct
   - Confirm start/end times are accurate
   - Consider daylight saving time

3. **Check entity configuration:**
   - Verify service/environment matches
   - Check filterType is correct

### Freeze Blocking Unexpected Deployments

1. **Review entity scope:**
   - `filterType: All` affects all entities
   - Check for overly broad filters

2. **Check hierarchy:**
   - Account freezes affect all orgs/projects
   - Org freezes affect all projects

### Recurring Freeze Issues

1. **Verify recurrence config:**
   ```yaml
   recurrence:
     type: Weekly
     spec:
       until: 2024-12-31 11:59 PM  # Must have end date
   ```

2. **Check recurrence calculation:**
   - Weekly: Same day each week
   - Monthly: Same date each month

### Override Freeze for Emergency

```bash
# Disable freeze temporarily
curl -X POST \
  'https://app.harness.io/ng/api/freeze/updateFreezeStatus?...' \
  -d '{"freezeIdentifiers": ["holiday_freeze"], "status": "Disabled"}'
```

## Instructions

When creating a freeze:

1. **Identify requirements:**
   - What period needs to be frozen?
   - Which services/environments affected?
   - Is it recurring?
   - What scope? (Account, Org, Project)

2. **Generate valid YAML:**
   - Use correct datetime format: `YYYY-MM-DD HH:MM AM/PM`
   - Specify timezone
   - Configure appropriate entity filters

3. **Consider impact:**
   - Communicate to affected teams
   - Plan for emergency procedures
   - Document freeze purpose

4. **Output the freeze YAML** in a code block

5. **Optionally create via API** if the user requests it
