# Harness Skills

This repository contains Claude Code skills for working with Harness.io CI/CD platform.

## Available Skills

### /create-pipeline

Generate Harness.io v0 Pipeline YAML files. This skill understands the complete Harness pipeline schema and can create:

- **CI Pipelines** - Build, test, and publish workflows
- **CD Pipelines** - Deployment workflows for Kubernetes, serverless, VMs, and more
- **Combined CI/CD Pipelines** - End-to-end delivery pipelines
- **Approval Workflows** - Manual and automated approval gates

**Usage:** `/create-pipeline` or ask Claude to create a Harness pipeline

**Examples:**
- "Create a CI pipeline for a Node.js app that builds and pushes to Docker Hub"
- "Generate a Kubernetes deployment pipeline with dev, staging, and prod environments"
- "Create a pipeline with parallel test stages and manual approval before production"

### /create-template

Generate Harness.io v0 Template YAML files for reusable pipeline components. This skill can create:

- **Step Templates** - Reusable step definitions (build, deploy, test steps)
- **Stage Templates** - Reusable stage definitions (CI stages, CD stages, approval stages)
- **Pipeline Templates** - Complete reusable pipeline definitions
- **StepGroup Templates** - Groups of related steps bundled together

**Usage:** `/create-template` or ask Claude to create a Harness template

**Examples:**
- "Create a step template for Docker build and push"
- "Generate a stage template for Kubernetes deployment with rollback"
- "Create a reusable test suite step group template"
- "Build a pipeline template for standard CI/CD workflow"

### /create-pipeline-v1

Generate Harness.io v1 Pipeline YAML files using the new simplified syntax. This skill creates:

- **Simplified CI Pipelines** - Clean, concise build and test workflows
- **Simplified CD Pipelines** - Deployment with services and environments
- **GitHub Actions Compatible** - Works with `jobs:` syntax
- **Matrix Builds** - Multi-version, multi-platform testing
- **Caching Intelligence** - Built-in cache support

**Usage:** `/create-pipeline-v1` or ask Claude to create a v1/simplified pipeline

**Examples:**
- "Create a v1 CI pipeline for a Node.js app"
- "Generate a simplified Kubernetes deployment pipeline"
- "Create a GitHub Actions compatible pipeline with Harness extensions"

### /create-trigger

Generate Harness.io v0 Trigger YAML files to automatically start pipelines. This skill can create:

- **Webhook Triggers** - GitHub, GitLab, Bitbucket, Azure Repos events
- **Scheduled Triggers** - Cron-based scheduling for periodic builds
- **Artifact Triggers** - Docker Hub, ECR, GCR, ACR, S3, Nexus updates
- **Custom Triggers** - Custom webhook integrations

**Usage:** `/create-trigger` or ask Claude to create a Harness trigger

**Examples:**
- "Create a GitHub PR trigger for my CI pipeline"
- "Generate a cron trigger for nightly builds at 2 AM"
- "Create an ECR trigger to deploy when new images are pushed"
- "Build a release trigger for production deployments on tags"

### /create-agent-template

Generate Harness Agent Template files for AI-powered automation agents. This skill creates:

- **metadata.json** - Template metadata and versioning
- **pipeline.yaml** - Pipeline definition (v1 syntax)
- **wiki.MD** - User-facing documentation

**Usage:** `/create-agent-template` or ask Claude to create an agent template

**Examples:**
- "Create an agent template for code review"
- "Generate a security scanner agent"
- "Create a documentation generator agent"

## MCP-Powered Skills

These skills leverage the Harness MCP Server for enhanced functionality. Install the MCP server from https://github.com/harness/mcp-server

### /debug-pipeline

Analyze pipeline execution failures and suggest fixes. Uses MCP tools to:

- Fetch recent execution history
- Download and analyze execution logs
- Identify error patterns and root causes
- Provide specific remediation steps

**Usage:** `/debug-pipeline` or ask Claude to debug a pipeline failure

**Examples:**
- "Why did my build-and-deploy pipeline fail?"
- "Debug the last failed execution of ci-pipeline"
- "Analyze the pipeline errors from today"

### /run-pipeline

Trigger and monitor Harness pipeline executions. Uses MCP tools to:

- List available pipelines and input sets
- Execute pipelines with custom inputs
- Monitor execution progress
- Report results and outputs

**Usage:** `/run-pipeline` or ask Claude to run a pipeline

**Examples:**
- "Run the deploy pipeline with version 2.0.0"
- "Deploy the api-service to staging"
- "Execute the CI pipeline for the main branch"

### /analyze-costs

Analyze cloud costs and optimization opportunities using Harness CCM. Uses MCP tools to:

- Generate cost overview reports
- Identify optimization recommendations
- Detect cost anomalies
- Create Jira/ServiceNow tickets for action items

**Usage:** `/analyze-costs` or ask Claude about cloud costs

**Examples:**
- "How much are we spending on cloud this month?"
- "Find me $5,000 in monthly savings"
- "Why did our AWS bill spike last week?"

### /security-report

Generate security compliance reports using Harness SCS and STO. Uses MCP tools to:

- List vulnerabilities by severity
- Download and analyze SBOMs
- Check compliance status
- Manage security exemptions

**Usage:** `/security-report` or ask Claude for a security report

**Examples:**
- "Generate a security report for backend-service:v2.3.4"
- "Show me all critical vulnerabilities in the payments project"
- "Download the SBOM for our API service"

### /dora-metrics

Generate DORA metrics and engineering performance reports using Harness SEI. Uses MCP tools to:

- Track deployment frequency, lead time, CFR, and MTTR
- Compare team performance
- Identify improvement opportunities
- Generate executive summaries

**Usage:** `/dora-metrics` or ask Claude for DORA metrics

**Examples:**
- "How are we doing on DORA metrics?"
- "Compare DORA metrics across all teams"
- "What's our deployment frequency trend?"

### /gitops-status

Check GitOps application status and health using Harness GitOps. Uses MCP tools to:

- Monitor ArgoCD application health
- Check sync status across environments
- View resource trees and pod status
- Get pod logs for debugging

**Usage:** `/gitops-status` or ask Claude about GitOps applications

**Examples:**
- "Show me the status of all GitOps applications"
- "Is the api-gateway application in sync?"
- "Get pod logs for the failing payment-service"

### /migrate-pipeline

Migrate pipelines from v0 to v1 format. Uses MCP tools to:

- Read existing v0 pipeline definitions
- Convert to simplified v1 syntax
- Preserve all functionality
- Validate the migration

**Usage:** `/migrate-pipeline` or ask Claude to migrate a pipeline

**Examples:**
- "Migrate the build-and-deploy pipeline to v1"
- "Show me what ci-pipeline would look like in v1 format"
- "Convert all my pipelines to the new syntax"

### /chaos-experiment

Create and manage chaos experiments using Harness Chaos Engineering. Uses MCP tools to:

- List and browse existing experiments
- Create experiments from templates
- Run chaos experiments
- Analyze experiment results

**Usage:** `/chaos-experiment` or ask Claude about chaos engineering

**Examples:**
- "Show me all chaos experiments"
- "Create a pod-delete experiment for the checkout-service"
- "Run the weekly resilience test"

### /scorecard-review

Review IDP scorecards and service maturity. Uses MCP tools to:

- View service scorecards and scores
- Check compliance with engineering standards
- Identify improvement areas
- Get remediation steps

**Usage:** `/scorecard-review` or ask Claude about service scorecards

**Examples:**
- "How is the api-gateway doing on scorecards?"
- "Which services are failing production readiness?"
- "Help me improve the checkout-service score"

### /audit-report

Generate audit reports and compliance trails. Uses MCP tools to:

- Track user actions and changes
- Generate compliance reports
- Investigate security incidents
- Monitor access patterns

**Usage:** `/audit-report` or ask Claude for an audit report

**Examples:**
- "Generate an audit report for the last 30 days"
- "What did john.doe do last week?"
- "Show all production pipeline changes this month"

## Schema References

- **v0 Pipelines/Templates/Triggers**: https://github.com/harness/harness-schema/tree/main/v0
- **v1 Pipelines**: https://github.com/thisrohangupta/spec
- **Agent Templates**: https://github.com/thisrohangupta/agents
- **MCP Server**: https://github.com/harness/mcp-server
