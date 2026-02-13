---
name: create-service
description: Generate Harness.io Service YAML definitions and optionally create them via the Harness API. Services define what you deploy (artifacts, manifests, config).
triggers:
  - harness service
  - create service
  - service definition
  - deployment service
  - service yaml
  - create service api
---

# Create Service Skill

Generate Harness.io Service YAML definitions and create them via the API.

## Overview

Services in Harness represent **what you deploy** - the artifact, manifest, and configuration that make up your deployable unit. This skill helps you:
- Define service configurations for different deployment types
- Create Kubernetes, Helm, Serverless, and other service types
- Configure artifact sources and manifest locations
- Create services via the Harness API

## Service Structure

Every Harness service follows this structure:

```yaml
service:
  identifier: <unique_identifier>
  name: <display_name>
  description: <optional_description>
  tags:
    key: value
  serviceDefinition:
    type: <deployment_type>
    spec:
      manifests: []
      artifacts: {}
      variables: []
```

## Deployment Types

### Kubernetes

```yaml
service:
  identifier: my_k8s_service
  name: My K8s Service
  serviceDefinition:
    type: Kubernetes
    spec:
      manifests:
        - manifest:
            identifier: manifests
            type: K8sManifest
            spec:
              store:
                type: Github
                spec:
                  connectorRef: github_connector
                  repoName: my-app
                  branch: main
                  paths:
                    - k8s/
              valuesPaths:
                - values.yaml
      artifacts:
        primary:
          primaryArtifactRef: <+input>
          sources:
            - identifier: docker_image
              type: DockerRegistry
              spec:
                connectorRef: dockerhub
                imagePath: myorg/myapp
                tag: <+input>
```

### Helm

```yaml
service:
  identifier: helm_service
  name: Helm Service
  serviceDefinition:
    type: NativeHelm
    spec:
      manifests:
        - manifest:
            identifier: helm_chart
            type: HelmChart
            spec:
              store:
                type: Http
                spec:
                  connectorRef: helm_repo
              chartName: my-chart
              chartVersion: <+input>
              helmVersion: V3
              skipResourceVersioning: false
      artifacts:
        primary:
          primaryArtifactRef: <+input>
          sources:
            - identifier: ecr_image
              type: Ecr
              spec:
                connectorRef: aws_connector
                region: us-east-1
                imagePath: my-app
                tag: <+input>
```

### AWS Lambda (Serverless)

```yaml
service:
  identifier: lambda_service
  name: Lambda Service
  serviceDefinition:
    type: ServerlessAwsLambda
    spec:
      manifests:
        - manifest:
            identifier: serverless_manifest
            type: ServerlessAwsLambda
            spec:
              store:
                type: Github
                spec:
                  connectorRef: github_connector
                  repoName: my-serverless-app
                  branch: main
                  paths:
                    - serverless.yml
              configOverridePath: config.yml
      artifacts:
        primary:
          primaryArtifactRef: <+input>
          sources:
            - identifier: s3_artifact
              type: AmazonS3
              spec:
                connectorRef: aws_connector
                region: us-east-1
                bucketName: my-artifacts
                filePath: lambda/function.zip
```

### ECS

```yaml
service:
  identifier: ecs_service
  name: ECS Service
  serviceDefinition:
    type: ECS
    spec:
      manifests:
        - manifest:
            identifier: task_definition
            type: EcsTaskDefinition
            spec:
              store:
                type: Github
                spec:
                  connectorRef: github_connector
                  repoName: my-ecs-app
                  branch: main
                  paths:
                    - ecs/task-definition.json
        - manifest:
            identifier: service_definition
            type: EcsServiceDefinition
            spec:
              store:
                type: Github
                spec:
                  connectorRef: github_connector
                  repoName: my-ecs-app
                  branch: main
                  paths:
                    - ecs/service-definition.json
      artifacts:
        primary:
          primaryArtifactRef: <+input>
          sources:
            - identifier: ecr_image
              type: Ecr
              spec:
                connectorRef: aws_connector
                region: us-east-1
                imagePath: my-ecs-app
                tag: <+input>
```

### Azure Web App

```yaml
service:
  identifier: azure_webapp
  name: Azure Web App
  serviceDefinition:
    type: AzureWebApp
    spec:
      artifacts:
        primary:
          primaryArtifactRef: <+input>
          sources:
            - identifier: acr_image
              type: Acr
              spec:
                connectorRef: azure_connector
                subscriptionId: <subscription_id>
                registry: myregistry.azurecr.io
                repository: my-app
                tag: <+input>
      applicationSettings:
        - name: APP_SETTING_1
          value: value1
          type: String
      connectionStrings:
        - name: DATABASE_URL
          value: <+secrets.getValue("db_url")>
          type: SQLAzure
```

### Google Cloud Run

```yaml
service:
  identifier: cloud_run_service
  name: Cloud Run Service
  serviceDefinition:
    type: GoogleCloudRun
    spec:
      artifacts:
        primary:
          primaryArtifactRef: <+input>
          sources:
            - identifier: gcr_image
              type: Gcr
              spec:
                connectorRef: gcp_connector
                registryHostname: gcr.io
                imagePath: my-project/my-app
                tag: <+input>
```

## Artifact Sources

### Docker Registry

```yaml
- identifier: dockerhub
  type: DockerRegistry
  spec:
    connectorRef: dockerhub_connector
    imagePath: library/nginx
    tag: <+input>
```

### ECR (Elastic Container Registry)

```yaml
- identifier: ecr
  type: Ecr
  spec:
    connectorRef: aws_connector
    region: us-east-1
    imagePath: my-app
    tag: <+input>
```

### GCR (Google Container Registry)

```yaml
- identifier: gcr
  type: Gcr
  spec:
    connectorRef: gcp_connector
    registryHostname: gcr.io
    imagePath: my-project/my-app
    tag: <+input>
```

### ACR (Azure Container Registry)

```yaml
- identifier: acr
  type: Acr
  spec:
    connectorRef: azure_connector
    subscriptionId: <subscription_id>
    registry: myregistry.azurecr.io
    repository: my-app
    tag: <+input>
```

### Amazon S3

```yaml
- identifier: s3
  type: AmazonS3
  spec:
    connectorRef: aws_connector
    region: us-east-1
    bucketName: my-bucket
    filePath: artifacts/app.zip
```

### Artifactory

```yaml
- identifier: artifactory
  type: ArtifactoryRegistry
  spec:
    connectorRef: artifactory_connector
    repository: docker-local
    artifactPath: my-app
    repositoryFormat: docker
    tag: <+input>
```

## Manifest Types

### Kubernetes Manifests

```yaml
- manifest:
    identifier: k8s_manifests
    type: K8sManifest
    spec:
      store:
        type: Github
        spec:
          connectorRef: github_connector
          repoName: my-repo
          branch: main
          paths:
            - manifests/
      valuesPaths:
        - values.yaml
        - values-<+env.name>.yaml
      skipResourceVersioning: false
```

### Helm Chart (HTTP Repository)

```yaml
- manifest:
    identifier: helm_chart
    type: HelmChart
    spec:
      store:
        type: Http
        spec:
          connectorRef: helm_http_connector
      chartName: my-chart
      chartVersion: 1.2.3
      helmVersion: V3
```

### Helm Chart (OCI Repository)

```yaml
- manifest:
    identifier: helm_oci
    type: HelmChart
    spec:
      store:
        type: OciHelmChart
        spec:
          connectorRef: oci_connector
          basePath: /charts
      chartName: my-chart
      chartVersion: <+input>
      helmVersion: V3
```

### Kustomize

```yaml
- manifest:
    identifier: kustomize
    type: Kustomize
    spec:
      store:
        type: Github
        spec:
          connectorRef: github_connector
          repoName: my-repo
          branch: main
          folderPath: kustomize/
      overlayConfiguration:
        kustomizeYamlFolderPath: overlays/production
      patchesPaths:
        - patches/
```

### OpenShift Template

```yaml
- manifest:
    identifier: openshift
    type: OpenshiftTemplate
    spec:
      store:
        type: Github
        spec:
          connectorRef: github_connector
          repoName: my-repo
          branch: main
          paths:
            - openshift/template.yaml
      paramsPaths:
        - openshift/params.yaml
```

## Service Variables

```yaml
service:
  identifier: my_service
  name: My Service
  serviceDefinition:
    type: Kubernetes
    spec:
      variables:
        - name: replicas
          type: Number
          value: 3
        - name: image_tag
          type: String
          value: <+input>
        - name: db_password
          type: Secret
          value: <+secrets.getValue("db_password")>
```

## Creating Services via API

### API Reference

**Endpoint:** `POST /v1/orgs/{org}/projects/{project}/services`

**Documentation:** https://apidocs.harness.io/tag/Project-Services

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
  "identifier": "my_service",
  "name": "My Service",
  "description": "Service description",
  "tags": {
    "team": "platform"
  },
  "yaml": "service:\n  identifier: my_service\n  name: My Service\n  ..."
}
```

### Example: Create Service

```bash
curl -X POST \
  'https://app.harness.io/v1/orgs/{org}/projects/{project}/services' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}' \
  -H 'Content-Type: application/json' \
  -d '{
    "identifier": "backend_api",
    "name": "Backend API",
    "description": "Main backend API service",
    "tags": {
      "team": "backend"
    },
    "yaml": "service:\n  identifier: backend_api\n  name: Backend API\n  serviceDefinition:\n    type: Kubernetes\n    spec:\n      manifests:\n        - manifest:\n            identifier: manifests\n            type: K8sManifest\n            spec:\n              store:\n                type: Github\n                spec:\n                  connectorRef: github\n                  repoName: backend-api\n                  branch: main\n                  paths:\n                    - k8s/"
  }'
```

### Example: Create Service from YAML File

```bash
SERVICE_YAML=$(cat service.yaml)

curl -X POST \
  'https://app.harness.io/v1/orgs/default/projects/my_project/services' \
  -H 'x-api-key: pat.xxxx.yyyy.zzzz' \
  -H 'Harness-Account: abc123' \
  -H 'Content-Type: application/json' \
  -d "$(jq -n \
    --arg id "my_service" \
    --arg name "My Service" \
    --arg yaml "$SERVICE_YAML" \
    '{identifier: $id, name: $name, yaml: $yaml}')"
```

### Response

**Success (201 Created):**

```json
{
  "service": {
    "identifier": "backend_api",
    "name": "Backend API",
    "description": "Main backend API service",
    "org": "default",
    "project": "my_project",
    "created": 1707500000000,
    "updated": 1707500000000
  }
}
```

### Update Existing Service

**Endpoint:** `PUT /v1/orgs/{org}/projects/{project}/services/{service}`

```bash
curl -X PUT \
  'https://app.harness.io/v1/orgs/{org}/projects/{project}/services/{serviceIdentifier}' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}' \
  -H 'Content-Type: application/json' \
  -d '{
    "identifier": "backend_api",
    "name": "Backend API (Updated)",
    "yaml": "..."
  }'
```

### Service Scopes

Services can be created at different scopes:

**Project-level (most common):**
```
POST /v1/orgs/{org}/projects/{project}/services
```

**Org-level (shared across projects):**
```
POST /v1/orgs/{org}/services
```

**Account-level (shared across org):**
```
POST /v1/services
```

## Complete Service Example

```yaml
service:
  identifier: ecommerce_api
  name: E-Commerce API
  description: Main e-commerce backend service
  tags:
    team: platform
    tier: backend
  serviceDefinition:
    type: Kubernetes
    spec:
      manifests:
        - manifest:
            identifier: k8s_manifests
            type: K8sManifest
            spec:
              store:
                type: Github
                spec:
                  connectorRef: github_connector
                  repoName: ecommerce-api
                  branch: main
                  paths:
                    - deploy/k8s/
              valuesPaths:
                - deploy/values.yaml
                - deploy/values-<+env.name>.yaml
      artifacts:
        primary:
          primaryArtifactRef: <+input>
          sources:
            - identifier: ecr_image
              type: Ecr
              spec:
                connectorRef: aws_connector
                region: us-east-1
                imagePath: ecommerce/api
                tag: <+input>
      variables:
        - name: LOG_LEVEL
          type: String
          value: info
        - name: REPLICA_COUNT
          type: Number
          value: <+input>
        - name: DB_PASSWORD
          type: Secret
          value: <+secrets.getValue("ecommerce_db_password")>
```

## Error Handling

### Common Errors

| Error Code | Description | Solution |
|------------|-------------|----------|
| `INVALID_REQUEST` | Malformed YAML or missing fields | Validate YAML structure |
| `DUPLICATE_IDENTIFIER` | Service with same ID exists | Use unique identifier or update existing |
| `CONNECTOR_NOT_FOUND` | Referenced connector doesn't exist | Create connector first |
| `INVALID_DEPLOYMENT_TYPE` | Unsupported deployment type | Use valid type (Kubernetes, NativeHelm, ECS, etc.) |
| `MANIFEST_NOT_FOUND` | Manifest path doesn't exist | Verify Git path and connector |

### Validation Errors

```yaml
# Common service validation issues:

# Missing artifact source configuration
artifacts:
  primary:
    primaryArtifactRef: <+input>
    # Missing: sources: [...]

# Invalid manifest store type
store:
  type: github  # Wrong (case-sensitive)
  type: Github  # Correct

# Wrong deployment type
serviceDefinition:
  type: kubernetes  # Wrong
  type: Kubernetes  # Correct
```

## Troubleshooting

### Service Not Appearing in Pipeline

1. **Check service scope:**
   - Verify service is in same project
   - For org/account services, use proper prefix

2. **Verify service is saved:**
   - Check for validation errors
   - Confirm service appears in Services list

### Artifact Resolution Failures

1. **Check connector credentials:**
   - Verify registry connector is valid
   - Test connector connectivity

2. **Verify artifact path:**
   - Check image path is correct
   - Verify tag exists in registry

3. **Check permissions:**
   - Ensure connector has pull permissions
   - Verify registry access from delegates

### Manifest Fetch Failures

1. **Git connectivity:**
   - Verify Git connector is valid
   - Check repository exists and is accessible

2. **Path verification:**
   - Confirm manifest paths exist
   - Check file names match exactly

3. **Branch/tag issues:**
   - Verify branch exists
   - Check for case sensitivity

### Values Override Issues

```yaml
# Debug values resolution
valuesPaths:
  - values.yaml
  - values-<+env.name>.yaml  # Check env.name resolves correctly
```

## Instructions

When creating a service:

1. **Identify requirements:**
   - What deployment type? (Kubernetes, Helm, ECS, Lambda, etc.)
   - Where are manifests stored? (Git, Helm repo, inline)
   - What artifact sources? (Docker, ECR, S3, etc.)
   - Any service variables needed?

2. **Generate valid YAML:**
   - Use correct identifier patterns: `^[a-zA-Z_][0-9a-zA-Z_$]{0,127}$`
   - Include all required fields for the deployment type
   - Reference connectors appropriately

3. **Use expressions for flexibility:**
   - `<+input>` for runtime inputs
   - `<+env.name>` for environment-specific values
   - `<+secrets.getValue("...")>` for secrets

4. **Output the service YAML** in a code block

5. **Optionally create via API** if the user requests it
