# Harness Skills

Claude Code skills for generating Harness.io CI/CD pipeline configurations.

## Installation

Clone this repository to use the skills with Claude Code:

```bash
git clone https://github.com/your-org/harness-skills.git
cd harness-skills
```

## Available Skills

### `/create-pipeline`

Generate Harness.io v0 Pipeline YAML files for CI/CD workflows and optionally create them via the Harness API.

**Capabilities:**
- CI Pipelines - Build, test, and publish workflows
- CD Pipelines - Deployment workflows (Kubernetes, ECS, Lambda, etc.)
- Combined CI/CD - End-to-end delivery pipelines
- Approval Workflows - Manual and automated approval gates
- Multi-environment deployments with parallel stages
- API Creation - Create pipelines directly in Harness

**Example Usage:**

```
/create-pipeline

Create a Node.js CI pipeline that:
- Runs on Harness Cloud
- Installs dependencies
- Runs linting and tests in parallel
- Builds a Docker image
- Pushes to Docker Hub
```

**Supported Stage Types:**
- `CI` - Continuous Integration
- `Deployment` - Continuous Deployment
- `Approval` - Approval gates
- `Custom` - Custom workflows
- `Security` - Security scanning
- `IACM` - Infrastructure as Code

**Supported Deployment Types:**
- Kubernetes
- Helm
- ECS
- AWS Lambda / Serverless
- Azure Web Apps / Functions
- Google Cloud Run / Functions
- SSH / WinRM
- And more...

**API:** `POST /v1/orgs/{org}/projects/{project}/pipelines` | [Docs](https://apidocs.harness.io/tag/Pipelines#operation/create-pipeline)

### `/create-pipeline-v1`

Generate Harness.io v1 Pipeline YAML files using the new simplified syntax.

**Capabilities:**
- Simplified, concise YAML syntax
- GitHub Actions compatibility (`jobs:` syntax)
- Native matrix and looping strategies
- Built-in caching intelligence
- Expression syntax: `${{ }}`

**Example Usage:**

```
/create-pipeline-v1

Create a CI pipeline that:
- Builds a Go application
- Runs tests with matrix strategy (Go 1.20, 1.21)
- Pushes Docker image on main branch
```

**Key Differences from v0:**
- Cleaner syntax: `run: npm test` vs nested `spec` objects
- Expressions: `${{ branch }}` vs `<+codebase.branch>`
- GitHub compatible: Can use `jobs:` instead of `pipeline:`

### `/create-template`

Generate Harness.io v0 Template YAML files for reusable pipeline components and optionally create them via the Harness API.

**Capabilities:**
- Step Templates - Reusable step definitions
- Stage Templates - Reusable stage definitions
- Pipeline Templates - Complete reusable pipelines
- StepGroup Templates - Related steps bundled together
- API Creation - Create templates directly in Harness

**Example Usage:**

```
/create-template

Create a step template for:
- Building and pushing Docker images to ECR
- Configurable image name and tags
- Resource limits
- Then create it in Harness via API
```

**Supported Template Types:**
- `Step` - Reusable steps (Run, Build, Deploy, etc.)
- `Stage` - Reusable stages (CI, CD, Approval, Custom)
- `Pipeline` - Complete pipeline definitions
- `StepGroup` - Groups of related steps

**API:** `POST /template/api/templates` | [Docs](https://apidocs.harness.io/templates/createtemplate)

### `/create-trigger`

Generate Harness.io v0 Trigger YAML files to automatically start pipelines.

**Capabilities:**
- Webhook Triggers - Git events (push, PR, tags, releases)
- Scheduled Triggers - Cron-based scheduling
- Artifact Triggers - Container registry and artifact updates

**Example Usage:**

```
/create-trigger

Create a GitHub webhook trigger that:
- Fires on PR open and sync
- Only for PRs targeting main branch
- Passes PR number to the pipeline
```

**Supported Webhook Types:**
- `Github` - Push, PR, Issue Comment, Release, Tag
- `Gitlab` - Push, Merge Request, Tag, Pipeline Hook
- `Bitbucket` - Push, Pull Request
- `AzureRepo` - Push, Pull Request
- `Custom` - Custom webhook payloads

**Supported Artifact Types:**
- `DockerRegistry` - Docker Hub
- `Ecr` - Amazon ECR
- `Gcr` - Google Container Registry
- `GoogleArtifactRegistry` - Google Artifact Registry
- `Acr` - Azure Container Registry
- `AmazonS3` - S3 bucket artifacts
- `Nexus3Registry` - Nexus Repository

### `/create-agent-template`

Generate Harness Agent Template files for AI-powered automation agents.

**Capabilities:**
- Code Review Agents - AI-powered PR review and commenting
- Test Generator Agents - Automated unit test generation
- Security Scanner Agents - Vulnerability detection and reporting
- Documentation Agents - Auto-generated documentation

**Example Usage:**

```
/create-agent-template

Create a code review agent that:
- Reviews pull request changes
- Comments on code quality issues
- Suggests improvements
- Supports GitHub, GitLab, and Bitbucket
```

**Generated Files:**
- `metadata.json` - Template metadata and versioning
- `pipeline.yaml` - Pipeline definition (v1 syntax)
- `wiki.MD` - User-facing documentation

**Common Patterns:**
- SCM provider detection (multi-provider support)
- PR creation with branch management
- AI coding agent integration
- Secret and connector handling

### `/template-usage`

Get template reference entities and usage information using the Harness API.

**Capabilities:**
- Find which pipelines use a specific template
- Analyze impact before updating templates
- Track template adoption across projects
- Identify unused templates for cleanup

**Example Usage:**

```
/template-usage

Which pipelines are using the docker-build-push template?
Show me the impact analysis for updating it.
```

**API:** `GET /template/api/templates/entitySetupUsage/{templateIdentifier}`
**Docs:** https://apidocs.harness.io/templates/listtemplateusage

### `/create-service`

Generate Harness.io Service YAML definitions and optionally create them via the API.

**Capabilities:**
- Kubernetes, Helm, ECS, Lambda service definitions
- Artifact sources (Docker, ECR, GCR, ACR, S3)
- Manifest configurations
- Service variables
- API Creation - Create services directly in Harness

**Example Usage:**

```
/create-service

Create a Kubernetes service with:
- Docker image from ECR
- K8s manifests from GitHub
- Environment-specific values files
```

**API:** `POST /v1/orgs/{org}/projects/{project}/services` | [Docs](https://apidocs.harness.io/tag/Project-Services)

### `/create-environment`

Generate Harness.io Environment YAML definitions and optionally create them via the API.

**Capabilities:**
- PreProduction and Production environment types
- Environment variables and overrides
- Environment grouping
- API Creation - Create environments directly in Harness

**Example Usage:**

```
/create-environment

Create staging and production environments with:
- Environment-specific variables
- Database connection configs
- Replica count settings
```

**API:** `POST /v1/orgs/{org}/projects/{project}/environments` | [Docs](https://apidocs.harness.io/tag/Project-Environments)

### `/create-infrastructure`

Generate Harness.io Infrastructure Definition YAML and optionally create them via the API.

**Capabilities:**
- Kubernetes (Direct, GKE, EKS, AKS)
- ECS, Lambda, Azure Web Apps, Cloud Run
- SSH/WinRM for traditional deployments
- API Creation - Create infrastructure directly in Harness

**Example Usage:**

```
/create-infrastructure

Create Kubernetes infrastructure definitions for:
- Dev cluster with namespace per service
- Prod cluster with HA configuration
```

**API:** `POST /v1/orgs/{org}/projects/{project}/environments/{env}/infrastructures` | [Docs](https://apidocs.harness.io/tag/Project-Infrastructures)

### `/create-connector`

Generate Harness.io Connector YAML definitions and optionally create them via the API.

**Capabilities:**
- Git connectors (GitHub, GitLab, Bitbucket, Azure Repos)
- Cloud providers (AWS, GCP, Azure)
- Container registries (Docker Hub, ECR, GCR, ACR)
- Kubernetes clusters
- API Creation - Create connectors directly in Harness

**Example Usage:**

```
/create-connector

Create a GitHub connector with:
- SSH authentication
- API access for PR status
- Delegate selector for private network
```

**API:** `POST /v1/orgs/{org}/projects/{project}/connectors` | [Docs](https://apidocs.harness.io/tag/Project-Connector)

### `/create-secret`

Generate Harness.io Secret definitions and optionally create them via the API.

**Capabilities:**
- Secret text (passwords, tokens, API keys)
- Secret files (certificates, config files)
- SSH keys and WinRM credentials
- External secret manager references
- API Creation - Create secrets directly in Harness

**Example Usage:**

```
/create-secret

Create secrets for:
- GitHub PAT for repository access
- AWS credentials for deployments
- SSH key for server access
```

**API:** `POST /v1/orgs/{org}/projects/{project}/secrets` | [Docs](https://apidocs.harness.io/tag/Project-Secret)

### `/create-input-set`

Generate Harness.io Input Set YAML definitions and optionally create them via the API.

**Capabilities:**
- Reusable runtime inputs for pipelines
- Environment-specific configurations
- Overlay input sets (combining multiple)
- API Creation - Create input sets directly in Harness

**Example Usage:**

```
/create-input-set

Create input sets for the deploy pipeline:
- Development inputs (1 replica, debug logging)
- Production inputs (5 replicas, warn logging)
```

**API:** `POST /v1/orgs/{org}/projects/{project}/input-sets` | [Docs](https://apidocs.harness.io/tag/Input-Sets)

### `/create-freeze`

Generate Harness.io Deployment Freeze YAML and optionally create them via the API.

**Capabilities:**
- Time-based deployment freezes
- Service and environment filtering
- Recurring freeze windows
- Global and scoped freezes
- API Creation - Create freezes directly in Harness

**Example Usage:**

```
/create-freeze

Create a holiday freeze that:
- Blocks all production deployments
- Runs from Dec 23-26
- Allows emergency overrides
```

**API:** `POST /ng/api/freeze` | [Docs](https://apidocs.harness.io/tag/Freeze-CRUD)

### `/manage-roles`

Manage Harness.io Role Assignments via the API for access control.

**Capabilities:**
- Assign built-in and custom roles
- User, group, and service account permissions
- Account, org, and project scopes
- Custom roles and resource groups

**Example Usage:**

```
/manage-roles

Assign the pipeline executor role to the dev team
for the staging project
```

**API:** `POST /v1/orgs/{org}/projects/{project}/role-assignments` | [Docs](https://apidocs.harness.io/tag/Account-Role-Assignments)

### `/webhook-manager`

Manage Harness.io GitX Webhooks via the API for Git integration.

**Capabilities:**
- GitX webhook configuration
- Git repository sync setup
- Webhook event monitoring
- Multi-folder sync

**Example Usage:**

```
/webhook-manager

Create a GitX webhook to sync:
- Pipelines from .harness/pipelines/
- Templates from .harness/templates/
```

**API:** `POST /v1/gitx-webhooks` | [Docs](https://apidocs.harness.io/tag/GitX-Webhooks)

## MCP-Powered Skills

These skills leverage the [Harness MCP Server](https://github.com/harness/mcp-server) for enhanced functionality. Install and configure the MCP server to enable these skills.

### `/debug-pipeline`

Analyze pipeline execution failures and suggest fixes.

**Capabilities:**
- Fetch recent execution history and logs
- Identify error patterns and root causes
- Provide specific remediation steps
- Detect common failure patterns

**Example Usage:**

```
/debug-pipeline

Analyze why the build-and-deploy pipeline failed
and suggest how to fix it
```

**MCP Tools Used:** `list_executions`, `get_execution`, `download_execution_logs`, `get_pipeline`

### `/run-pipeline`

Trigger and monitor Harness pipeline executions.

**Capabilities:**
- List available pipelines and input sets
- Execute pipelines with custom inputs
- Monitor execution progress
- Report results and outputs

**Example Usage:**

```
/run-pipeline

Deploy version 2.0.0 of the api-service to staging
```

**MCP Tools Used:** `list_pipelines`, `get_pipeline`, `list_input_sets`, `get_execution`

### `/analyze-costs`

Analyze cloud costs and optimization opportunities using Harness CCM.

**Capabilities:**
- Generate cost overview reports
- Identify optimization recommendations
- Detect cost anomalies
- Create Jira/ServiceNow tickets for action items

**Example Usage:**

```
/analyze-costs

Find me $5,000 in monthly savings from our
cloud infrastructure
```

**MCP Tools Used:** `get_ccm_overview`, `list_ccm_recommendations`, `list_ccm_anomalies`, `create_jira_ticket_for_ccm_recommendation`

### `/security-report`

Generate security compliance reports using Harness SCS and STO.

**Capabilities:**
- List vulnerabilities by severity
- Download and analyze SBOMs
- Check compliance status
- Manage security exemptions

**Example Usage:**

```
/security-report

Generate a security report for backend-service:v2.3.4
with remediation guidance for critical CVEs
```

**MCP Tools Used:** `get_all_security_issues`, `scs_get_artifact_overview`, `scs_download_sbom`, `scs_get_artifact_component_remediation`

### `/dora-metrics`

Generate DORA metrics and engineering performance reports using Harness SEI.

**Capabilities:**
- Track deployment frequency, lead time, CFR, and MTTR
- Compare team performance
- Identify improvement opportunities
- Generate executive summaries

**Example Usage:**

```
/dora-metrics

Compare DORA metrics across all teams
and identify who needs support
```

**MCP Tools Used:** `sei_deployment_frequency`, `sei_efficiency_lead_time`, `sei_change_failure_rate`, `sei_mttr`

### `/gitops-status`

Check GitOps application status and health using Harness GitOps.

**Capabilities:**
- Monitor ArgoCD application health
- Check sync status across environments
- View resource trees and pod status
- Get pod logs for debugging

**Example Usage:**

```
/gitops-status

Show me the status of all production GitOps applications
and highlight any sync issues
```

**MCP Tools Used:** `gitops_list_applications`, `gitops_get_application`, `gitops_get_app_resource_tree`, `gitops_get_pod_logs`

### `/migrate-pipeline`

Migrate pipelines from v0 to v1 format.

**Capabilities:**
- Read existing v0 pipeline definitions
- Convert to simplified v1 syntax
- Preserve all functionality
- Show side-by-side comparison

**Example Usage:**

```
/migrate-pipeline

Migrate the build-and-deploy pipeline to v1 format
and show me the differences
```

**MCP Tools Used:** `get_pipeline`, `list_pipelines`

### `/chaos-experiment`

Create and manage chaos experiments using Harness Chaos Engineering.

**Capabilities:**
- List and browse existing experiments
- Create experiments from templates
- Run chaos experiments
- Analyze experiment results

**Example Usage:**

```
/chaos-experiment

Create a pod-delete chaos experiment for the
checkout-service in the staging environment
```

**MCP Tools Used:** `chaos_experiments_list`, `chaos_experiment_describe`, `chaos_create_experiment_from_template`, `chaos_experiment_run`

### `/scorecard-review`

Review IDP scorecards and service maturity.

**Capabilities:**
- View service scorecards and scores
- Check compliance with engineering standards
- Identify improvement areas
- Get remediation steps for failing checks

**Example Usage:**

```
/scorecard-review

How is the api-gateway doing on the production
readiness scorecard?
```

**MCP Tools Used:** `get_scorecard`, `list_scorecards`, `get_score_summary`, `get_scores`

### `/audit-report`

Generate audit reports and compliance trails.

**Capabilities:**
- Track user actions and changes
- Generate compliance reports
- Investigate security incidents
- Monitor access patterns

**Example Usage:**

```
/audit-report

Generate an audit report for all production
pipeline changes in the last 30 days
```

**MCP Tools Used:** `list_user_audits`

## Schema Reference

- **v0 Pipelines/Templates/Triggers:** https://github.com/harness/harness-schema/tree/main/v0
- **v1 Pipelines:** https://github.com/thisrohangupta/spec
- **Agent Templates:** https://github.com/thisrohangupta/agents
- **MCP Server:** https://github.com/harness/mcp-server

## Project Structure

```
harness-skills/
├── .claude/
│   └── skills/
│       ├── create-pipeline.md       # v0 Pipeline skill + API
│       ├── create-pipeline-v1.md    # v1 Pipeline skill
│       ├── create-template.md       # Template skill + API
│       ├── create-trigger.md        # Trigger skill
│       ├── create-agent-template.md # Agent Template skill
│       ├── template-usage.md        # Template references API
│       ├── create-service.md        # Service definitions + API
│       ├── create-environment.md    # Environment definitions + API
│       ├── create-infrastructure.md # Infrastructure definitions + API
│       ├── create-connector.md      # Connector definitions + API
│       ├── create-secret.md         # Secret definitions + API
│       ├── create-input-set.md      # Input Set definitions + API
│       ├── create-freeze.md         # Deployment Freeze + API
│       ├── manage-roles.md          # Role Assignments API
│       ├── webhook-manager.md       # GitX Webhooks API
│       ├── debug-pipeline.md        # MCP: Pipeline debugging
│       ├── run-pipeline.md          # MCP: Pipeline execution
│       ├── analyze-costs.md         # MCP: Cost analysis
│       ├── security-report.md       # MCP: Security reports
│       ├── dora-metrics.md          # MCP: DORA metrics
│       ├── gitops-status.md         # MCP: GitOps monitoring
│       ├── migrate-pipeline.md      # MCP: v0 to v1 migration
│       ├── chaos-experiment.md      # MCP: Chaos engineering
│       ├── scorecard-review.md      # MCP: IDP scorecards
│       └── audit-report.md          # MCP: Audit & compliance
├── examples/
│   ├── v0/                          # v0 format examples
│   │   ├── python-flask-cicd.yaml
│   │   └── microservices-cicd.yaml
│   ├── v1/                          # v1 format examples
│   │   ├── nodejs-cicd.yaml
│   │   └── go-microservice-cicd.yaml
│   ├── templates/
│   │   ├── docker-build-push-step.yaml
│   │   └── k8s-blue-green-stage.yaml
│   ├── triggers/
│   │   └── github-cicd-triggers.yaml
│   ├── agents/                      # Agent template examples
│   │   └── code-review-agent/
│   ├── services/                    # Service definition examples
│   │   ├── kubernetes-backend-service.yaml
│   │   ├── helm-microservice.yaml
│   │   └── serverless-lambda.yaml
│   ├── environments/                # Environment examples
│   │   └── multi-environment-setup.yaml
│   ├── infrastructures/             # Infrastructure examples
│   │   ├── kubernetes-multi-cluster.yaml
│   │   └── ecs-fargate.yaml
│   ├── connectors/                  # Connector examples
│   │   ├── git-connectors.yaml
│   │   ├── cloud-connectors.yaml
│   │   ├── registry-connectors.yaml
│   │   └── kubernetes-connectors.yaml
│   ├── secrets/                     # Secret examples
│   │   └── common-secrets.yaml
│   ├── input-sets/                  # Input set examples
│   │   └── deployment-input-sets.yaml
│   └── freezes/                     # Deployment freeze examples
│       └── deployment-freezes.yaml
├── CLAUDE.md
├── LICENSE
└── README.md
```

## Examples

See the `examples/` directory for complete examples:

**v0 Pipelines:**
- **v0/python-flask-cicd.yaml** - Full CI/CD pipeline with ECR, K8s, approvals
- **v0/microservices-cicd.yaml** - Complete microservices pipeline with security scanning and canary deployments

**v1 Pipelines (Simplified Format):**
- **v1/nodejs-cicd.yaml** - CI/CD with caching, matrix testing, K8s deployment
- **v1/go-microservice-cicd.yaml** - Go service pipeline with canary deployments

**Templates:**
- **templates/docker-build-push-step.yaml** - Reusable Docker build step template
- **templates/k8s-blue-green-stage.yaml** - Blue-green deployment stage template

**Triggers:**
- **triggers/github-cicd-triggers.yaml** - PR, push, release, and scheduled triggers

**Services:**
- **services/kubernetes-backend-service.yaml** - K8s service with Docker artifacts
- **services/helm-microservice.yaml** - Helm-based service with ECR
- **services/serverless-lambda.yaml** - AWS Lambda serverless service

**Environments:**
- **environments/multi-environment-setup.yaml** - Dev, staging, prod environments with variables

**Infrastructures:**
- **infrastructures/kubernetes-multi-cluster.yaml** - Multi-cluster K8s (EKS, GKE)
- **infrastructures/ecs-fargate.yaml** - ECS Fargate infrastructure

**Connectors:**
- **connectors/git-connectors.yaml** - GitHub, GitLab, Bitbucket, Azure Repos
- **connectors/cloud-connectors.yaml** - AWS, GCP, Azure cloud providers
- **connectors/registry-connectors.yaml** - Docker Hub, ECR, GCR, ACR, Artifactory
- **connectors/kubernetes-connectors.yaml** - K8s cluster connections

**Secrets:**
- **secrets/common-secrets.yaml** - API keys, passwords, SSH keys, certificates

**Input Sets:**
- **input-sets/deployment-input-sets.yaml** - Environment-specific deployment inputs

**Freezes:**
- **freezes/deployment-freezes.yaml** - Holiday, maintenance, and event freezes

**Agent Templates:**
- **agents/code-review-agent/** - AI-powered code review agent with PR commenting

## License

Apache 2.0
