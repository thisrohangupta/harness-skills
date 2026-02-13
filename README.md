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

Generate Harness.io v0 Pipeline YAML files for CI/CD workflows.

**Capabilities:**
- CI Pipelines - Build, test, and publish workflows
- CD Pipelines - Deployment workflows (Kubernetes, ECS, Lambda, etc.)
- Combined CI/CD - End-to-end delivery pipelines
- Approval Workflows - Manual and automated approval gates
- Multi-environment deployments with parallel stages

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

Generate Harness.io v0 Template YAML files for reusable pipeline components.

**Capabilities:**
- Step Templates - Reusable step definitions
- Stage Templates - Reusable stage definitions
- Pipeline Templates - Complete reusable pipelines
- StepGroup Templates - Related steps bundled together

**Example Usage:**

```
/create-template

Create a step template for:
- Building and pushing Docker images to ECR
- Configurable image name and tags
- Resource limits
```

**Supported Template Types:**
- `Step` - Reusable steps (Run, Build, Deploy, etc.)
- `Stage` - Reusable stages (CI, CD, Approval, Custom)
- `Pipeline` - Complete pipeline definitions
- `StepGroup` - Groups of related steps

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
│       ├── create-pipeline.md       # v0 Pipeline skill
│       ├── create-pipeline-v1.md    # v1 Pipeline skill
│       ├── create-template.md       # Template skill
│       ├── create-trigger.md        # Trigger skill
│       ├── create-agent-template.md # Agent Template skill
│       ├── debug-pipeline.md        # MCP: Pipeline debugging
│       ├── run-pipeline.md          # MCP: Pipeline execution
│       ├── analyze-costs.md         # MCP: Cost analysis
│       ├── security-report.md       # MCP: Security reports
│       └── dora-metrics.md          # MCP: DORA metrics
├── examples/
│   ├── v0/                          # v0 format examples
│   │   └── python-flask-cicd.yaml
│   ├── v1/                          # v1 format examples
│   │   └── nodejs-cicd.yaml
│   ├── templates/
│   │   ├── docker-build-push-step.yaml
│   │   └── k8s-blue-green-stage.yaml
│   ├── triggers/
│   │   └── github-cicd-triggers.yaml
│   └── agents/                      # Agent template examples
│       └── code-review-agent/
│           ├── metadata.json
│           ├── pipeline.yaml
│           └── wiki.MD
├── CLAUDE.md
├── LICENSE
└── README.md
```

## Examples

See the `examples/` directory for complete examples:

**v0 Pipelines (Current):**
- **v0/python-flask-cicd.yaml** - Full CI/CD pipeline with ECR, K8s, approvals

**v1 Pipelines (New Simplified Format):**
- **v1/nodejs-cicd.yaml** - CI/CD with caching, matrix testing, K8s deployment

**Templates:**
- **templates/docker-build-push-step.yaml** - Reusable Docker build step template
- **templates/k8s-blue-green-stage.yaml** - Blue-green deployment stage template

**Triggers:**
- **triggers/github-cicd-triggers.yaml** - PR, push, release, and scheduled triggers

**Agent Templates:**
- **agents/code-review-agent/** - AI-powered code review agent with PR commenting

## License

Apache 2.0
