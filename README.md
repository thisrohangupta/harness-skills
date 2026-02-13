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

## Schema Reference

All pipelines follow the Harness v0 schema:
https://github.com/harness/harness-schema/tree/main/v0

## Project Structure

```
harness-skills/
├── .claude/
│   └── skills/
│       ├── create-pipeline.md    # Pipeline skill definition
│       ├── create-template.md    # Template skill definition
│       └── create-trigger.md     # Trigger skill definition
├── examples/
│   ├── python-flask-cicd.yaml    # Example CI/CD pipeline
│   ├── templates/                # Example templates
│   │   ├── docker-build-push-step.yaml
│   │   └── k8s-blue-green-stage.yaml
│   └── triggers/                 # Example triggers
│       └── github-cicd-triggers.yaml
├── skills/
│   └── create-pipeline/
│       └── create-pipeline.md    # Detailed documentation
├── CLAUDE.md                     # Claude Code configuration
├── LICENSE
└── README.md
```

## Examples

See the `examples/` directory for complete examples:

**Pipelines:**
- **python-flask-cicd.yaml** - Full CI/CD pipeline with build, test, Docker/ECR, Kubernetes deployment, approval gates, and notifications

**Templates:**
- **templates/docker-build-push-step.yaml** - Reusable Docker build and push step template
- **templates/k8s-blue-green-stage.yaml** - Kubernetes blue-green deployment stage template

**Triggers:**
- **triggers/github-cicd-triggers.yaml** - Complete CI/CD trigger setup with PR, push, and release triggers

## License

Apache 2.0
