---
name: create-secret
description: Generate Harness.io Secret definitions and optionally create them via the Harness API. Secrets store sensitive data like passwords, tokens, SSH keys, and certificates.
triggers:
  - harness secret
  - create secret
  - secret text
  - secret file
  - ssh key
  - api key secret
  - create secret api
---

# Create Secret Skill

Generate Harness.io Secret definitions and create them via the API.

## Overview

Secrets in Harness securely store sensitive data like passwords, API tokens, SSH keys, and certificates. This skill helps you:
- Define different types of secrets
- Configure secret managers
- Set up SSH and WinRM credentials
- Create secrets via the Harness API

## Secret Types

### Secret Text

Store text-based secrets (passwords, tokens, API keys):

```yaml
secret:
  identifier: my_api_key
  name: My API Key
  description: API key for external service
  type: SecretText
  spec:
    secretManagerIdentifier: harnessSecretManager
    valueType: Inline
    value: <secret_value>
```

### Secret File

Store file-based secrets (certificates, config files):

```yaml
secret:
  identifier: ssl_cert
  name: SSL Certificate
  description: Production SSL certificate
  type: SecretFile
  spec:
    secretManagerIdentifier: harnessSecretManager
```

### SSH Key (Key Reference)

SSH credentials using key reference:

```yaml
secret:
  identifier: ssh_key
  name: SSH Key
  type: SSHKey
  spec:
    auth:
      type: SSH
      spec:
        credentialType: KeyReference
        spec:
          userName: ec2-user
          key: ssh_private_key
          encryptedPassphrase: ssh_passphrase  # Optional
    port: 22
```

### SSH Key (Key Path)

SSH credentials using key file path on delegate:

```yaml
secret:
  identifier: ssh_key_path
  name: SSH Key Path
  type: SSHKey
  spec:
    auth:
      type: SSH
      spec:
        credentialType: KeyPath
        spec:
          userName: ubuntu
          keyPath: /home/harness/.ssh/id_rsa
    port: 22
```

### SSH Password

SSH credentials using password:

```yaml
secret:
  identifier: ssh_password
  name: SSH Password
  type: SSHKey
  spec:
    auth:
      type: SSH
      spec:
        credentialType: Password
        spec:
          userName: admin
          password: ssh_password_secret
    port: 22
```

### WinRM (NTLM)

Windows Remote Management credentials:

```yaml
secret:
  identifier: winrm_creds
  name: WinRM Credentials
  type: WinRmCredentials
  spec:
    auth:
      type: NTLM
      spec:
        username: Administrator
        password: winrm_password
        domain: MYDOMAIN  # Optional
    port: 5986
    useSSL: true
```

### WinRM (Kerberos)

```yaml
secret:
  identifier: winrm_kerberos
  name: WinRM Kerberos
  type: WinRmCredentials
  spec:
    auth:
      type: Kerberos
      spec:
        principal: admin@MYDOMAIN.COM
        realm: MYDOMAIN.COM
        keyTabFilePath: /etc/krb5.keytab  # Or use tgtGenerationMethod
        tgtGenerationMethod:
          type: KeyTabFilePath
          spec:
            keyTabFilePath: /etc/krb5.keytab
    port: 5986
    useSSL: true
```

## Secret Manager References

### Harness Built-in Secret Manager

```yaml
secret:
  identifier: my_secret
  name: My Secret
  type: SecretText
  spec:
    secretManagerIdentifier: harnessSecretManager
    valueType: Inline
    value: <secret_value>
```

### AWS Secrets Manager

```yaml
secret:
  identifier: aws_secret
  name: AWS Secret
  type: SecretText
  spec:
    secretManagerIdentifier: aws_secrets_manager
    valueType: Reference
    value: my-secret-name  # Secret name in AWS
```

### HashiCorp Vault

```yaml
secret:
  identifier: vault_secret
  name: Vault Secret
  type: SecretText
  spec:
    secretManagerIdentifier: hashicorp_vault
    valueType: Reference
    value: secret/data/myapp/api-key#key  # Vault path
```

### Azure Key Vault

```yaml
secret:
  identifier: azure_secret
  name: Azure Key Vault Secret
  type: SecretText
  spec:
    secretManagerIdentifier: azure_key_vault
    valueType: Reference
    value: my-secret-name
```

### GCP Secret Manager

```yaml
secret:
  identifier: gcp_secret
  name: GCP Secret
  type: SecretText
  spec:
    secretManagerIdentifier: gcp_secret_manager
    valueType: Reference
    value: projects/my-project/secrets/my-secret/versions/latest
```

## Creating Secrets via API

### API Reference

**Endpoint:** `POST /v1/orgs/{org}/projects/{project}/secrets`

**Documentation:** https://apidocs.harness.io/tag/Project-Secret

### Request Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-api-key` | Yes | Harness API key |
| `Harness-Account` | Yes | Account identifier |
| `Content-Type` | Yes | `application/json` |

### Create Secret Text

```bash
curl -X POST \
  'https://app.harness.io/v1/orgs/{org}/projects/{project}/secrets' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}' \
  -H 'Content-Type: application/json' \
  -d '{
    "secret": {
      "identifier": "my_api_key",
      "name": "My API Key",
      "description": "API key for external service",
      "type": "SecretText",
      "spec": {
        "secret_manager_identifier": "harnessSecretManager",
        "value_type": "Inline",
        "value": "my-secret-value-here"
      }
    }
  }'
```

### Create Secret File

```bash
curl -X POST \
  'https://app.harness.io/v1/orgs/{org}/projects/{project}/secrets/files' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}' \
  -F 'file=@/path/to/certificate.pem' \
  -F 'spec={"secret": {"identifier": "ssl_cert", "name": "SSL Certificate", "type": "SecretFile", "spec": {"secret_manager_identifier": "harnessSecretManager"}}}'
```

### Create SSH Key

```bash
curl -X POST \
  'https://app.harness.io/v1/orgs/{org}/projects/{project}/secrets' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}' \
  -H 'Content-Type: application/json' \
  -d '{
    "secret": {
      "identifier": "ssh_deploy_key",
      "name": "SSH Deploy Key",
      "type": "SSHKey",
      "spec": {
        "auth": {
          "type": "SSH",
          "spec": {
            "credential_type": "KeyReference",
            "spec": {
              "user_name": "deploy",
              "key": "ssh_private_key_secret"
            }
          }
        },
        "port": 22
      }
    }
  }'
```

### Response

**Success (201 Created):**

```json
{
  "secret": {
    "identifier": "my_api_key",
    "name": "My API Key",
    "type": "SecretText",
    "org": "default",
    "project": "my_project",
    "created": 1707500000000,
    "updated": 1707500000000
  }
}
```

### Update Secret

**Endpoint:** `PUT /v1/orgs/{org}/projects/{project}/secrets/{secret}`

```bash
curl -X PUT \
  'https://app.harness.io/v1/orgs/{org}/projects/{project}/secrets/{secretId}' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}' \
  -H 'Content-Type: application/json' \
  -d '{
    "secret": {
      "identifier": "my_api_key",
      "name": "My API Key (Updated)",
      "type": "SecretText",
      "spec": {
        "secret_manager_identifier": "harnessSecretManager",
        "value_type": "Inline",
        "value": "new-secret-value"
      }
    }
  }'
```

### Validate Secret Reference

**Endpoint:** `POST /v1/orgs/{org}/projects/{project}/secrets/validate`

```bash
curl -X POST \
  'https://app.harness.io/v1/orgs/{org}/projects/{project}/secrets/validate' \
  -H 'x-api-key: {apiKey}' \
  -H 'Harness-Account: {accountId}' \
  -H 'Content-Type: application/json' \
  -d '{
    "secret_ref": "my_api_key",
    "type": "SecretText"
  }'
```

### Secret Scopes

Secrets can be created at different scopes:

**Project-level:**
```
POST /v1/orgs/{org}/projects/{project}/secrets
```

**Org-level (shared across projects):**
```
POST /v1/orgs/{org}/secrets
```

**Account-level (shared across orgs):**
```
POST /v1/secrets
```

## Referencing Secrets

### In Pipeline YAML

```yaml
# Reference secret value
<+secrets.getValue("my_api_key")>

# Reference org-level secret
<+secrets.getValue("org.my_api_key")>

# Reference account-level secret
<+secrets.getValue("account.my_api_key")>
```

### In Connector Configuration

```yaml
connector:
  identifier: github
  type: Github
  spec:
    authentication:
      type: Http
      spec:
        type: UsernameToken
        spec:
          username: my-user
          tokenRef: github_pat  # Secret identifier
```

### In Service Variables

```yaml
service:
  identifier: my_service
  serviceDefinition:
    spec:
      variables:
        - name: DB_PASSWORD
          type: Secret
          value: <+secrets.getValue("db_password")>
```

## Complete Examples

### GitHub Personal Access Token

```yaml
secret:
  identifier: github_pat
  name: GitHub PAT
  description: Personal Access Token for GitHub
  type: SecretText
  orgIdentifier: default
  projectIdentifier: my_project
  tags:
    provider: github
    type: pat
  spec:
    secretManagerIdentifier: harnessSecretManager
    valueType: Inline
    value: ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

### AWS Credentials

```yaml
# AWS Access Key ID (not really secret, but stored for convenience)
secret:
  identifier: aws_access_key
  name: AWS Access Key ID
  type: SecretText
  spec:
    secretManagerIdentifier: harnessSecretManager
    valueType: Inline
    value: AKIAXXXXXXXXXXXXXXXX

# AWS Secret Access Key
secret:
  identifier: aws_secret_key
  name: AWS Secret Access Key
  type: SecretText
  spec:
    secretManagerIdentifier: harnessSecretManager
    valueType: Inline
    value: <secret_value>
```

### Database Credentials

```yaml
secret:
  identifier: prod_db_password
  name: Production DB Password
  description: PostgreSQL password for production database
  type: SecretText
  tags:
    environment: production
    database: postgresql
  spec:
    secretManagerIdentifier: hashicorp_vault
    valueType: Reference
    value: secret/data/production/database#password
```

### SSH Deploy Key

```yaml
secret:
  identifier: deploy_ssh_key
  name: Deploy SSH Key
  description: SSH key for deployment servers
  type: SSHKey
  spec:
    auth:
      type: SSH
      spec:
        credentialType: KeyReference
        spec:
          userName: deploy
          key: ssh_private_key_ref
    port: 22
```

## Best Practices

### Naming Conventions

| Secret Type | Identifier Pattern | Example |
|-------------|-------------------|---------|
| API Keys | `{service}_api_key` | `github_api_key` |
| Passwords | `{system}_password` | `prod_db_password` |
| Tokens | `{provider}_token` | `slack_token` |
| SSH Keys | `ssh_{purpose}` | `ssh_deploy_key` |
| Certificates | `{service}_cert` | `ssl_prod_cert` |

### Security Best Practices

1. **Use external secret managers** for production (Vault, AWS SM, etc.)
2. **Scope secrets appropriately** (project > org > account)
3. **Rotate secrets regularly** with automation
4. **Audit secret access** via audit logs
5. **Use private secrets** for highly sensitive data
6. **Never log secret values** in pipelines

### Private Secrets

Mark secrets as private to prevent value viewing:

```bash
curl -X POST \
  'https://app.harness.io/v1/orgs/{org}/projects/{project}/secrets?private_secret=true' \
  ...
```

## Error Handling

### Common Errors

| Error Code | Description | Solution |
|------------|-------------|----------|
| `INVALID_REQUEST` | Malformed request or missing fields | Validate request structure |
| `DUPLICATE_IDENTIFIER` | Secret with same ID exists | Use unique identifier |
| `SECRET_MANAGER_NOT_FOUND` | Invalid secret manager reference | Verify secret manager exists |
| `ENCRYPTION_FAILED` | Unable to encrypt secret | Check secret manager connectivity |
| `INVALID_SECRET_TYPE` | Unsupported secret type | Use SecretText, SecretFile, SSHKey, or WinRmCredentials |

### Validation Errors

```yaml
# Common secret validation issues:

# Invalid valueType
spec:
  valueType: inline  # Wrong (case-sensitive)
  valueType: Inline  # Correct

# Missing SSH credential spec
type: SSHKey
spec:
  auth:
    type: SSH
    spec:
      credentialType: KeyReference
      # Missing: spec: { userName, key }

# Invalid secret manager reference
secretManagerIdentifier: vault  # May not exist
secretManagerIdentifier: harnessSecretManager  # Built-in
```

## Troubleshooting

### Secret Not Accessible

1. **Check scope:**
   - Project secrets: Use identifier directly
   - Org secrets: Use `org.secret_id`
   - Account secrets: Use `account.secret_id`

2. **Verify permissions:**
   - Check user/service account has secret access
   - Verify RBAC role includes secret view

### Secret Manager Connection Issues

1. **External secret manager:**
   - Verify connector is valid
   - Check network connectivity from delegate

2. **HashiCorp Vault:**
   - Verify path is correct
   - Check token permissions

3. **AWS Secrets Manager:**
   - Verify IAM permissions
   - Check secret name/ARN

### SSH Key Issues

1. **Key format:**
   - Ensure PEM format for private key
   - Check key isn't password-protected (or provide passphrase)

2. **Connection failures:**
   - Verify username is correct
   - Check target server accepts key auth

### Secret Reference Errors

```yaml
# Debug secret references
steps:
  - step:
      type: Run
      spec:
        command: |
          # This will fail - secrets are masked
          echo $MY_SECRET
          # Use secrets for actual operations
          curl -H "Authorization: Bearer $MY_SECRET" ...
        envVariables:
          MY_SECRET: <+secrets.getValue("my_api_key")>
```

## Instructions

When creating a secret:

1. **Identify requirements:**
   - What type of secret? (Text, File, SSH, WinRM)
   - Which secret manager to use?
   - What scope? (Project, Org, Account)
   - Should it be private?

2. **Generate valid configuration:**
   - Use appropriate type
   - Reference correct secret manager
   - Set proper auth methods for credentials

3. **Consider security:**
   - Use external secret managers for production
   - Apply appropriate scoping
   - Enable private mode when needed

4. **Output the secret configuration** (without actual values)

5. **Optionally create via API** if the user requests it
