---
name: create-connector
description: Generate Harness.io Connector YAML definitions and optionally create them via the Harness API. Connectors integrate with Git, cloud providers, container registries, and other external systems.
triggers:
  - harness connector
  - create connector
  - git connector
  - aws connector
  - docker connector
  - cloud connector
  - create connector api
---

# Create Connector Skill

Generate Harness.io Connector YAML definitions and create them via the API.

## Overview

Connectors in Harness enable integration with external systems - source control, cloud providers, artifact registries, and monitoring tools. This skill helps you:
- Define connectors for various platforms
- Configure authentication and credentials
- Set up delegate selectors
- Create connectors via the Harness API

## Connector Structure

Every connector follows this structure:

```yaml
connector:
  identifier: <unique_identifier>
  name: <display_name>
  description: <optional_description>
  orgIdentifier: <org_id>
  projectIdentifier: <project_id>
  tags:
    key: value
  type: <connector_type>
  spec:
    # Connector-specific configuration
```

## Git Connectors

### GitHub (HTTP)

```yaml
connector:
  identifier: github_connector
  name: GitHub
  type: Github
  spec:
    url: https://github.com/my-org
    authentication:
      type: Http
      spec:
        type: UsernameToken
        spec:
          username: my-username
          tokenRef: github_pat
    apiAccess:
      type: Token
      spec:
        tokenRef: github_pat
    delegateSelectors:
      - my-delegate
    executeOnDelegate: true
```

### GitHub (SSH)

```yaml
connector:
  identifier: github_ssh
  name: GitHub SSH
  type: Github
  spec:
    url: git@github.com:my-org
    authentication:
      type: Ssh
      spec:
        sshKeyRef: github_ssh_key
    apiAccess:
      type: Token
      spec:
        tokenRef: github_pat
```

### GitHub App

```yaml
connector:
  identifier: github_app
  name: GitHub App
  type: Github
  spec:
    url: https://github.com/my-org
    authentication:
      type: Http
      spec:
        type: GitHubApp
        spec:
          installationId: "12345678"
          applicationId: "123456"
          privateKeyRef: github_app_private_key
    apiAccess:
      type: GitHubApp
      spec:
        installationId: "12345678"
        applicationId: "123456"
        privateKeyRef: github_app_private_key
```

### GitLab

```yaml
connector:
  identifier: gitlab_connector
  name: GitLab
  type: Gitlab
  spec:
    url: https://gitlab.com/my-org
    authentication:
      type: Http
      spec:
        type: UsernameToken
        spec:
          username: my-username
          tokenRef: gitlab_token
    apiAccess:
      type: Token
      spec:
        tokenRef: gitlab_token
```

### Bitbucket

```yaml
connector:
  identifier: bitbucket_connector
  name: Bitbucket
  type: Bitbucket
  spec:
    url: https://bitbucket.org/my-org
    authentication:
      type: Http
      spec:
        type: UsernameToken
        spec:
          username: my-username
          tokenRef: bitbucket_app_password
    apiAccess:
      type: UsernameToken
      spec:
        username: my-username
        tokenRef: bitbucket_app_password
```

### Azure Repos

```yaml
connector:
  identifier: azure_repos
  name: Azure Repos
  type: AzureRepo
  spec:
    url: https://dev.azure.com/my-org
    authentication:
      type: Http
      spec:
        type: UsernameToken
        spec:
          username: my-username
          tokenRef: azure_pat
    apiAccess:
      type: Token
      spec:
        tokenRef: azure_pat
```

## Cloud Provider Connectors

### AWS

```yaml
connector:
  identifier: aws_connector
  name: AWS
  type: Aws
  spec:
    credential:
      type: ManualConfig
      spec:
        accessKey: AKIAXXXXXXXXXXXXXXXX
        secretKeyRef: aws_secret_key
      region: us-east-1
    delegateSelectors:
      - aws-delegate
```

### AWS (IRSA - IAM Roles for Service Accounts)

```yaml
connector:
  identifier: aws_irsa
  name: AWS IRSA
  type: Aws
  spec:
    credential:
      type: Irsa
      spec:
        region: us-east-1
    delegateSelectors:
      - eks-delegate
```

### AWS (Assume Role)

```yaml
connector:
  identifier: aws_assume_role
  name: AWS Assume Role
  type: Aws
  spec:
    credential:
      type: ManualConfig
      spec:
        accessKey: AKIAXXXXXXXXXXXXXXXX
        secretKeyRef: aws_secret_key
      region: us-east-1
      crossAccountAccess:
        crossAccountRoleArn: arn:aws:iam::123456789012:role/HarnessCrossAccountRole
        externalId: harness-external-id
```

### GCP

```yaml
connector:
  identifier: gcp_connector
  name: GCP
  type: Gcp
  spec:
    credential:
      type: ManualConfig
      spec:
        secretKeyRef: gcp_service_account_key
    delegateSelectors:
      - gcp-delegate
```

### GCP (Workload Identity)

```yaml
connector:
  identifier: gcp_workload_identity
  name: GCP Workload Identity
  type: Gcp
  spec:
    credential:
      type: InheritFromDelegate
    delegateSelectors:
      - gke-delegate
```

### Azure

```yaml
connector:
  identifier: azure_connector
  name: Azure
  type: Azure
  spec:
    credential:
      type: ManualConfig
      spec:
        applicationId: <app_id>
        tenantId: <tenant_id>
        auth:
          type: Secret
          spec:
            secretRef: azure_client_secret
    azureEnvironmentType: AZURE
    delegateSelectors:
      - azure-delegate
```

### Azure (Managed Identity)

```yaml
connector:
  identifier: azure_msi
  name: Azure Managed Identity
  type: Azure
  spec:
    credential:
      type: InheritFromDelegate
      spec:
        auth:
          type: UserAssignedManagedIdentity
          spec:
            clientId: <managed_identity_client_id>
    azureEnvironmentType: AZURE
    delegateSelectors:
      - aks-delegate
```

## Kubernetes Connectors

### Kubernetes (Direct)

```yaml
connector:
  identifier: k8s_connector
  name: Kubernetes Cluster
  type: K8sCluster
  spec:
    credential:
      type: ManualConfig
      spec:
        masterUrl: https://k8s-api.example.com
        auth:
          type: ServiceAccount
          spec:
            serviceAccountTokenRef: k8s_sa_token
            caCertRef: k8s_ca_cert
    delegateSelectors:
      - k8s-delegate
```

### Kubernetes (Inherit from Delegate)

```yaml
connector:
  identifier: k8s_delegate
  name: Kubernetes via Delegate
  type: K8sCluster
  spec:
    credential:
      type: InheritFromDelegate
    delegateSelectors:
      - in-cluster-delegate
```

## Container Registry Connectors

### Docker Hub

```yaml
connector:
  identifier: dockerhub
  name: Docker Hub
  type: DockerRegistry
  spec:
    dockerRegistryUrl: https://index.docker.io/v2/
    providerType: DockerHub
    auth:
      type: UsernamePassword
      spec:
        username: my-username
        passwordRef: dockerhub_password
```

### Amazon ECR

```yaml
connector:
  identifier: ecr_connector
  name: Amazon ECR
  type: Aws
  spec:
    credential:
      type: ManualConfig
      spec:
        accessKey: AKIAXXXXXXXXXXXXXXXX
        secretKeyRef: aws_secret_key
      region: us-east-1
```

### Google Container Registry (GCR)

```yaml
connector:
  identifier: gcr_connector
  name: Google Container Registry
  type: Gcr
  spec:
    registryHostname: gcr.io
    credential:
      type: ManualConfig
      spec:
        secretKeyRef: gcp_service_account_key
```

### Google Artifact Registry

```yaml
connector:
  identifier: gar_connector
  name: Google Artifact Registry
  type: Gar
  spec:
    credential:
      type: ManualConfig
      spec:
        secretKeyRef: gcp_service_account_key
```

### Azure Container Registry (ACR)

```yaml
connector:
  identifier: acr_connector
  name: Azure Container Registry
  type: Acr
  spec:
    subscriptionId: <subscription_id>
    resourceGroup: my-resource-group
    azureEnvironmentType: AZURE
    credential:
      type: ManualConfig
      spec:
        applicationId: <app_id>
        tenantId: <tenant_id>
        auth:
          type: Secret
          spec:
            secretRef: azure_client_secret
```

### Artifactory

```yaml
connector:
  identifier: artifactory
  name: JFrog Artifactory
  type: Artifactory
  spec:
    artifactoryServerUrl: https://my-org.jfrog.io/artifactory
    auth:
      type: UsernamePassword
      spec:
        username: my-username
        passwordRef: artifactory_password
```

## Helm Connectors

### Helm HTTP Repository

```yaml
connector:
  identifier: helm_http
  name: Helm Repository
  type: HttpHelmRepo
  spec:
    helmRepoUrl: https://charts.example.com
    auth:
      type: UsernamePassword
      spec:
        username: helm-user
        passwordRef: helm_password
```

### Helm OCI Repository

```yaml
connector:
  identifier: helm_oci
  name: Helm OCI Registry
  type: OciHelmRepo
  spec:
    helmRepoUrl: oci://registry.example.com/charts
    auth:
      type: UsernamePassword
      spec:
        username: oci-user
        passwordRef: oci_password
```

## Creating Connectors via API

### API Reference

**Endpoint:** `POST /v1/orgs/{org}/projects/{project}/connectors`

**Documentation:** https://apidocs.harness.io/tag/Project-Connector

### Request Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-api-key` | Yes | Harness API key |
| `Harness-Account` | Yes | Account identifier |
| `Content-Type` | Yes | `application/json` |

### Request Body

```json
{
  "connector": {
    "identifier": "github_connector",
    "name": "GitHub",
    "type": "Github",
    "spec": {
      "type": "GitHttp",
      "url": "https://github.com/my-org",
      "validationRepo": "my-repo",
      "authentication": {
        "type": "Http",
        "spec": {
          "type": "UsernameToken",
          "spec": {
            "username": "my-username",
            "tokenRef": "github_pat"
          }
        }
      }
    }
  }
}
```

### Example: Create Connector

```bash
curl -X POST \
  'https://app.harness.io/v1/orgs/{org}/projects/{project}/connectors' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}' \
  -H 'Content-Type: application/json' \
  -d '{
    "connector": {
      "identifier": "github_connector",
      "name": "GitHub",
      "description": "GitHub connector for my-org",
      "type": "Github",
      "spec": {
        "type": "GitHttp",
        "url": "https://github.com/my-org",
        "authentication": {
          "type": "Http",
          "spec": {
            "type": "UsernameToken",
            "spec": {
              "username": "my-username",
              "tokenRef": "github_pat"
            }
          }
        }
      }
    }
  }'
```

### Response

**Success (201 Created):**

```json
{
  "connector": {
    "identifier": "github_connector",
    "name": "GitHub",
    "type": "Github",
    "org": "default",
    "project": "my_project",
    "created": 1707500000000,
    "updated": 1707500000000
  }
}
```

### Test Connector

**Endpoint:** `POST /v1/orgs/{org}/projects/{project}/connectors/{connector}/test`

```bash
curl -X POST \
  'https://app.harness.io/v1/orgs/{org}/projects/{project}/connectors/{connectorId}/test' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}'
```

### Connector Scopes

Connectors can be created at different scopes:

**Project-level:**
```
POST /v1/orgs/{org}/projects/{project}/connectors
```

**Org-level (shared across projects):**
```
POST /v1/orgs/{org}/connectors
```

**Account-level (shared across orgs):**
```
POST /v1/connectors
```

## Delegate Selectors

Configure which delegates can use the connector:

```yaml
connector:
  identifier: aws_prod
  name: AWS Production
  type: Aws
  spec:
    credential:
      type: ManualConfig
      spec:
        accessKey: AKIAXXXXXXXXXXXXXXXX
        secretKeyRef: aws_secret_key
      region: us-east-1
    delegateSelectors:
      - prod-delegate
      - aws-delegate
    executeOnDelegate: true
```

## Best Practices

### Naming Conventions

| Connector Type | Identifier Pattern | Example |
|---------------|-------------------|---------|
| Git | `{provider}_{purpose}` | `github_app_repo` |
| Cloud | `{cloud}_{env}` | `aws_prod` |
| Registry | `{registry}_{env}` | `ecr_prod` |
| K8s | `k8s_{cluster}` | `k8s_prod_east` |

### Security Best Practices

1. **Use secret references** for all credentials
2. **Scope connectors appropriately** (project > org > account)
3. **Use delegate selectors** to control access
4. **Prefer IAM roles** over static credentials
5. **Rotate credentials** regularly

## Error Handling

### Common Errors

| Error Code | Description | Solution |
|------------|-------------|----------|
| `INVALID_REQUEST` | Malformed YAML or missing fields | Validate YAML structure |
| `DUPLICATE_IDENTIFIER` | Connector with same ID exists | Use unique identifier |
| `SECRET_NOT_FOUND` | Referenced secret doesn't exist | Create secret first |
| `INVALID_CREDENTIALS` | Authentication failed during test | Verify credentials |
| `DELEGATE_NOT_AVAILABLE` | No delegate matches selectors | Check delegate status |

### Validation Errors

```yaml
# Common connector validation issues:

# Wrong authentication type for provider
type: Github
spec:
  authentication:
    type: SSH
    spec:
      type: UsernamePassword  # Wrong - SSH doesn't use UsernamePassword
      type: sshKeyRef  # Correct

# Missing required field for auth type
authentication:
  type: Http
  spec:
    type: UsernameToken
    spec:
      username: my-user
      # Missing: tokenRef

# Invalid secret reference
tokenRef: github_pat  # Just identifier - may need scope prefix
tokenRef: account.github_pat  # With scope prefix
```

## Troubleshooting

### Connector Test Fails

1. **Check credentials:**
   - Verify secret values are correct
   - Check for expired tokens

2. **Network connectivity:**
   - Ensure delegate can reach target
   - Check firewall rules

3. **Permission issues:**
   - Verify API token has required scopes
   - Check service account permissions

### Git Connector Issues

1. **Authentication failures:**
   - For HTTPS: Check PAT has repo access
   - For SSH: Verify SSH key is in correct format

2. **Repository not found:**
   - Check repository URL is correct
   - Verify organization/user name

3. **Clone failures:**
   - Check branch exists
   - Verify depth setting

### Cloud Connector Issues

1. **AWS:**
   - Verify access key and secret
   - Check IAM permissions
   - For IRSA: Verify service account annotation

2. **GCP:**
   - Check service account key is valid JSON
   - Verify required APIs are enabled
   - For workload identity: Check namespace/SA binding

3. **Azure:**
   - Verify tenant ID and client ID
   - Check client secret isn't expired
   - For MSI: Verify managed identity assignment

### Kubernetes Connector Issues

1. **Direct connection:**
   - Verify master URL is reachable
   - Check service account token

2. **Inherit from delegate:**
   - Ensure delegate runs in target cluster
   - Check delegate service account permissions

3. **Certificate issues:**
   - Verify CA certificate is correct
   - Check certificate chain is complete

## Instructions

When creating a connector:

1. **Identify requirements:**
   - What system to connect to?
   - What authentication method?
   - Which delegates should use it?
   - What scope? (Project, Org, Account)

2. **Generate valid YAML:**
   - Use correct identifier patterns
   - Reference secrets properly
   - Configure appropriate auth type

3. **Consider security:**
   - Never hardcode credentials
   - Use secret references
   - Configure delegate selectors

4. **Output the connector YAML** in a code block

5. **Optionally create via API** if the user requests it
