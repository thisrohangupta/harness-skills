---
name: create-infrastructure
description: Generate Harness.io Infrastructure Definition YAML and optionally create them via the Harness API. Infrastructure definitions specify the deployment target (cluster, namespace, region).
triggers:
  - harness infrastructure
  - create infrastructure
  - infrastructure definition
  - infra definition
  - deployment target
  - create infra
  - create infrastructure api
---

# Create Infrastructure Skill

Generate Harness.io Infrastructure Definition YAML and create them via the API.

## Overview

Infrastructure definitions in Harness specify **the exact deployment target** within an environment - the cluster, namespace, region, or compute resources where your service runs. This skill helps you:
- Define infrastructure for Kubernetes, ECS, Lambda, and other platforms
- Configure deployment targets with connectors and credentials
- Set up namespace, cluster, and region specifications
- Create infrastructure via the Harness API

## Infrastructure Structure

Every infrastructure definition follows this structure:

```yaml
infrastructureDefinition:
  identifier: <unique_identifier>
  name: <display_name>
  description: <optional_description>
  tags:
    key: value
  orgIdentifier: <org_id>
  projectIdentifier: <project_id>
  environmentRef: <environment_identifier>
  deploymentType: <deployment_type>
  type: <infrastructure_type>
  spec:
    # Infrastructure-specific configuration
```

## Infrastructure Types

### Kubernetes Direct

Direct connection to Kubernetes cluster:

```yaml
infrastructureDefinition:
  identifier: k8s_dev
  name: Kubernetes Dev Cluster
  environmentRef: dev
  deploymentType: Kubernetes
  type: KubernetesDirect
  spec:
    connectorRef: k8s_connector
    namespace: my-app-dev
    releaseName: release-<+INFRA_KEY_SHORT_ID>
```

### Kubernetes GCP (GKE)

```yaml
infrastructureDefinition:
  identifier: gke_prod
  name: GKE Production
  environmentRef: prod
  deploymentType: Kubernetes
  type: KubernetesGcp
  spec:
    connectorRef: gcp_connector
    cluster: production-cluster
    namespace: my-app
    releaseName: release-<+INFRA_KEY_SHORT_ID>
```

### Kubernetes Azure (AKS)

```yaml
infrastructureDefinition:
  identifier: aks_staging
  name: AKS Staging
  environmentRef: staging
  deploymentType: Kubernetes
  type: KubernetesAzure
  spec:
    connectorRef: azure_connector
    subscriptionId: <subscription_id>
    resourceGroup: my-resource-group
    cluster: staging-aks-cluster
    namespace: my-app
    releaseName: release-<+INFRA_KEY_SHORT_ID>
```

### Kubernetes AWS (EKS)

```yaml
infrastructureDefinition:
  identifier: eks_prod
  name: EKS Production
  environmentRef: prod
  deploymentType: Kubernetes
  type: KubernetesAws
  spec:
    connectorRef: aws_connector
    cluster: production-eks
    region: us-east-1
    namespace: my-app
    releaseName: release-<+INFRA_KEY_SHORT_ID>
```

### Amazon ECS

```yaml
infrastructureDefinition:
  identifier: ecs_prod
  name: ECS Production
  environmentRef: prod
  deploymentType: ECS
  type: ECS
  spec:
    connectorRef: aws_connector
    region: us-east-1
    cluster: production-cluster
```

### AWS Lambda (Serverless)

```yaml
infrastructureDefinition:
  identifier: lambda_prod
  name: Lambda Production
  environmentRef: prod
  deploymentType: ServerlessAwsLambda
  type: ServerlessAwsLambda
  spec:
    connectorRef: aws_connector
    region: us-east-1
    stage: prod
```

### Azure Web App

```yaml
infrastructureDefinition:
  identifier: azure_webapp_prod
  name: Azure Web App Production
  environmentRef: prod
  deploymentType: AzureWebApp
  type: AzureWebApp
  spec:
    connectorRef: azure_connector
    subscriptionId: <subscription_id>
    resourceGroup: my-resource-group
    webApp: my-webapp
    deploymentSlot: production
```

### Google Cloud Run

```yaml
infrastructureDefinition:
  identifier: cloudrun_prod
  name: Cloud Run Production
  environmentRef: prod
  deploymentType: GoogleCloudRun
  type: GoogleCloudRun
  spec:
    connectorRef: gcp_connector
    project: my-gcp-project
    region: us-central1
```

### SSH/WinRM (Traditional)

```yaml
infrastructureDefinition:
  identifier: vm_prod
  name: VM Production
  environmentRef: prod
  deploymentType: Ssh
  type: SshWinRmAws
  spec:
    connectorRef: aws_connector
    region: us-east-1
    hostConnectionType: PrivateIP
    credentialsRef: ssh_key
    asgName: my-auto-scaling-group
```

### Helm (Native)

```yaml
infrastructureDefinition:
  identifier: helm_prod
  name: Helm Production
  environmentRef: prod
  deploymentType: NativeHelm
  type: KubernetesDirect
  spec:
    connectorRef: k8s_connector
    namespace: helm-releases
    releaseName: <+service.identifier>-<+env.identifier>
```

## Complete Infrastructure Examples

### Multi-Environment Kubernetes Setup

**Development:**

```yaml
infrastructureDefinition:
  identifier: k8s_dev
  name: Kubernetes Development
  description: Development cluster in us-east-1
  environmentRef: dev
  deploymentType: Kubernetes
  type: KubernetesDirect
  orgIdentifier: default
  projectIdentifier: my_project
  tags:
    tier: development
    region: us-east-1
  spec:
    connectorRef: k8s_dev_connector
    namespace: <+service.identifier>-dev
    releaseName: release-<+INFRA_KEY_SHORT_ID>
```

**Staging:**

```yaml
infrastructureDefinition:
  identifier: k8s_staging
  name: Kubernetes Staging
  description: Staging cluster in us-east-1
  environmentRef: staging
  deploymentType: Kubernetes
  type: KubernetesDirect
  orgIdentifier: default
  projectIdentifier: my_project
  tags:
    tier: staging
    region: us-east-1
  spec:
    connectorRef: k8s_staging_connector
    namespace: <+service.identifier>-staging
    releaseName: release-<+INFRA_KEY_SHORT_ID>
```

**Production:**

```yaml
infrastructureDefinition:
  identifier: k8s_prod
  name: Kubernetes Production
  description: Production cluster with high availability
  environmentRef: prod
  deploymentType: Kubernetes
  type: KubernetesDirect
  orgIdentifier: default
  projectIdentifier: my_project
  tags:
    tier: production
    region: us-east-1
    ha: "true"
  spec:
    connectorRef: k8s_prod_connector
    namespace: <+service.identifier>
    releaseName: release-<+INFRA_KEY_SHORT_ID>
```

### ECS with Fargate

```yaml
infrastructureDefinition:
  identifier: ecs_fargate_prod
  name: ECS Fargate Production
  description: Serverless ECS with Fargate
  environmentRef: prod
  deploymentType: ECS
  type: ECS
  orgIdentifier: default
  projectIdentifier: my_project
  tags:
    platform: fargate
    tier: production
  spec:
    connectorRef: aws_connector
    region: us-east-1
    cluster: fargate-production
```

### Multi-Region Lambda

```yaml
infrastructureDefinition:
  identifier: lambda_us_east
  name: Lambda US East
  description: Lambda functions in US East
  environmentRef: prod
  deploymentType: ServerlessAwsLambda
  type: ServerlessAwsLambda
  tags:
    region: us-east-1
  spec:
    connectorRef: aws_connector
    region: us-east-1
    stage: production
```

## Creating Infrastructure via API

### API Reference

**Endpoint:** `POST /v1/orgs/{org}/projects/{project}/environments/{environment}/infrastructures`

**Documentation:** https://apidocs.harness.io/tag/Project-Infrastructures

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
| `environment` | Yes | Environment identifier |

### Request Body

```json
{
  "identifier": "k8s_dev",
  "name": "Kubernetes Dev",
  "description": "Development K8s cluster",
  "tags": {
    "tier": "development"
  },
  "yaml": "infrastructureDefinition:\n  identifier: k8s_dev\n  ..."
}
```

### Example: Create Infrastructure

```bash
curl -X POST \
  'https://app.harness.io/v1/orgs/{org}/projects/{project}/environments/{env}/infrastructures' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}' \
  -H 'Content-Type: application/json' \
  -d '{
    "identifier": "k8s_prod",
    "name": "Kubernetes Production",
    "description": "Production Kubernetes cluster",
    "tags": {
      "tier": "production"
    },
    "yaml": "infrastructureDefinition:\n  identifier: k8s_prod\n  name: Kubernetes Production\n  environmentRef: prod\n  deploymentType: Kubernetes\n  type: KubernetesDirect\n  spec:\n    connectorRef: k8s_prod_connector\n    namespace: production\n    releaseName: release-<+INFRA_KEY_SHORT_ID>"
  }'
```

### Example: Create Infrastructure from YAML File

```bash
INFRA_YAML=$(cat infrastructure.yaml)

curl -X POST \
  'https://app.harness.io/v1/orgs/default/projects/my_project/environments/prod/infrastructures' \
  -H 'x-api-key: pat.xxxx.yyyy.zzzz' \
  -H 'Harness-Account: abc123' \
  -H 'Content-Type: application/json' \
  -d "$(jq -n \
    --arg id "k8s_prod" \
    --arg name "Kubernetes Production" \
    --arg yaml "$INFRA_YAML" \
    '{identifier: $id, name: $name, yaml: $yaml}')"
```

### Response

**Success (201 Created):**

```json
{
  "infrastructure": {
    "identifier": "k8s_prod",
    "name": "Kubernetes Production",
    "description": "Production Kubernetes cluster",
    "environment": "prod",
    "deploymentType": "Kubernetes",
    "org": "default",
    "project": "my_project",
    "created": 1707500000000,
    "updated": 1707500000000
  }
}
```

### Update Existing Infrastructure

**Endpoint:** `PUT /v1/orgs/{org}/projects/{project}/environments/{env}/infrastructures/{infra}`

```bash
curl -X PUT \
  'https://app.harness.io/v1/orgs/{org}/projects/{project}/environments/{env}/infrastructures/{infraId}' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}' \
  -H 'Content-Type: application/json' \
  -d '{
    "identifier": "k8s_prod",
    "name": "Kubernetes Production (Updated)",
    "yaml": "..."
  }'
```

## Dynamic Infrastructure

Use expressions for dynamic values:

```yaml
infrastructureDefinition:
  identifier: k8s_dynamic
  name: Dynamic K8s Infrastructure
  environmentRef: <+input>
  deploymentType: Kubernetes
  type: KubernetesDirect
  spec:
    connectorRef: <+input>
    namespace: <+pipeline.variables.namespace>
    releaseName: <+service.identifier>-<+env.identifier>-<+pipeline.sequenceId>
```

## Best Practices

### Naming Conventions

| Platform | Identifier Pattern | Example |
|----------|-------------------|---------|
| Kubernetes | `k8s_{env}` | `k8s_prod` |
| ECS | `ecs_{env}_{region}` | `ecs_prod_useast1` |
| Lambda | `lambda_{env}_{region}` | `lambda_prod_useast1` |
| Azure | `azure_{env}` | `azure_prod` |
| GCP | `gcp_{env}` | `gcp_prod` |

### Release Name Strategy

Use meaningful release names:

```yaml
# Include service and environment
releaseName: <+service.identifier>-<+env.identifier>

# Include unique ID for parallel deployments
releaseName: release-<+INFRA_KEY_SHORT_ID>

# Include pipeline execution
releaseName: <+service.identifier>-<+pipeline.sequenceId>
```

### Namespace Strategy

```yaml
# Environment-based
namespace: <+env.identifier>

# Service-based
namespace: <+service.identifier>

# Combined
namespace: <+service.identifier>-<+env.identifier>

# Static
namespace: production
```

## Error Handling

### Common Errors

| Error Code | Description | Solution |
|------------|-------------|----------|
| `INVALID_REQUEST` | Malformed YAML or missing fields | Validate YAML structure |
| `DUPLICATE_IDENTIFIER` | Infrastructure with same ID exists | Use unique identifier |
| `CONNECTOR_NOT_FOUND` | Referenced connector doesn't exist | Create connector first |
| `ENVIRONMENT_NOT_FOUND` | Referenced environment doesn't exist | Create environment first |
| `TYPE_MISMATCH` | Infrastructure type doesn't match deployment type | Ensure compatibility |

### Validation Errors

```yaml
# Common infrastructure validation issues:

# Mismatched deployment and infrastructure types
deploymentType: Kubernetes
type: ECS  # Wrong - doesn't match deployment type
type: KubernetesDirect  # Correct

# Missing required namespace
type: KubernetesDirect
spec:
  connectorRef: k8s_connector
  # Missing: namespace

# Invalid release name expression
releaseName: <+service.name>  # Wrong (has spaces)
releaseName: <+service.identifier>  # Correct
```

## Troubleshooting

### Infrastructure Not Selectable in Pipeline

1. **Check environment reference:**
   - Infrastructure must be linked to correct environment
   - Verify environmentRef matches

2. **Verify deployment type compatibility:**
   - Infrastructure type must match pipeline deployment type
   - Check stage deploymentType setting

### Deployment Failures

1. **Connector connectivity:**
   - Test connector in Harness UI
   - Verify delegate can reach cluster

2. **Namespace issues:**
   - Check namespace exists in cluster
   - Verify service account permissions

3. **Cluster authentication:**
   - Verify credentials are valid
   - Check certificate expiration

### Kubernetes-Specific Issues

1. **Permission errors:**
   ```bash
   # Check service account permissions
   kubectl auth can-i create deployments --namespace=<namespace>
   ```

2. **Namespace not found:**
   - Create namespace before deployment
   - Use auto-create if supported

3. **Release name conflicts:**
   ```yaml
   # Use unique release names
   releaseName: release-<+INFRA_KEY_SHORT_ID>
   ```

### Cloud Provider Issues

1. **GKE/EKS/AKS connectivity:**
   - Verify cloud connector credentials
   - Check cluster is accessible

2. **Region/zone mismatch:**
   - Ensure region in infrastructure matches cluster location

## Instructions

When creating infrastructure:

1. **Identify requirements:**
   - What deployment type? (Kubernetes, ECS, Lambda, etc.)
   - What infrastructure type? (Direct, GKE, EKS, etc.)
   - Target environment?
   - Required connectors?

2. **Generate valid YAML:**
   - Use correct identifier patterns
   - Reference correct connectors
   - Set appropriate namespace/cluster

3. **Use expressions appropriately:**
   - Dynamic namespaces with `<+service.identifier>`
   - Unique release names with `<+INFRA_KEY_SHORT_ID>`
   - Runtime inputs with `<+input>`

4. **Output the infrastructure YAML** in a code block

5. **Optionally create via API** if the user requests it
