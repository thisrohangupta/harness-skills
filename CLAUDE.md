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

## Schema References

- **v0 Pipelines/Templates/Triggers**: https://github.com/harness/harness-schema/tree/main/v0
- **v1 Pipelines**: https://github.com/thisrohangupta/spec
- **Agent Templates**: https://github.com/thisrohangupta/agents
