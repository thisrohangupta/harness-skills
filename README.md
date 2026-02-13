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

## Schema Reference

All pipelines follow the Harness v0 schema:
https://github.com/harness/harness-schema/tree/main/v0

## Project Structure

```
harness-skills/
├── .claude/
│   └── skills/
│       ├── create-pipeline.md       # v0 Pipeline skill
│       ├── create-pipeline-v1.md    # v1 Pipeline skill
│       ├── create-template.md       # Template skill
│       ├── create-trigger.md        # Trigger skill
│       └── create-agent-template.md # Agent Template skill
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
