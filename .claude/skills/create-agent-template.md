---
name: create-agent-template
description: Generate Harness Agent Template files including metadata.json, pipeline.yaml, and wiki.MD. Use when the user wants to create an AI agent, agent template, automation agent, or Harness agent.
triggers:
  - agent template
  - create agent
  - harness agent
  - ai agent
  - automation agent
  - agent pipeline
---

# Create Agent Template Skill

Generate Harness Agent Template files for AI-powered automation agents.

## Overview

Agent templates are modular pipeline definitions that encapsulate AI-powered automation capabilities. Each agent template consists of:

- **metadata.json** - Template metadata and versioning (required)
- **pipeline.yaml** - Pipeline definition using v1 syntax (required)
- **wiki.MD** - User-facing documentation (optional but recommended)
- **logo.svg** - Template logo/icon (optional)

## Schema Reference

Source: https://github.com/thisrohangupta/agents

## Template Structure

```
templates/
└── <agent-name>/
    ├── metadata.json      # Required: Metadata and versioning
    ├── pipeline.yaml      # Required: Pipeline definition
    ├── wiki.MD            # Optional: Documentation
    └── logo.svg           # Optional: Logo/icon
```

## metadata.json

Required fields for template metadata:

```json
{
  "name": "My Agent",
  "description": "Brief description of what this agent does (1-2 sentences)",
  "version": "1.0.0"
}
```

### Naming Conventions

- **Use Sentence Case**: `My Agent`, `Code Coverage`, `Autofix`
- **Avoid**: `my-agent`, `MyAgent`, `MY_AGENT`

### Version Format

Use semantic versioning: `MAJOR.MINOR.PATCH`
- `1.0.0` - Initial release
- `1.1.0` - New features, backward compatible
- `2.0.0` - Breaking changes

## pipeline.yaml

Pipeline definition using Harness v1 YAML syntax:

### Basic Structure

```yaml
version: 1
pipeline:
  # Clone configuration
  clone:
    depth: 1
    ref:
      name: <+inputs.branch>
      type: branch
    repo: <+inputs.repo>
    connector: "<+inputs.gitConnector != null ? inputs.gitConnector.id : ''>"

  # Stages
  stages:
    - name: my-agent
      steps:
        - name: agent-step
          run:
            container:
              image: myregistry/my-agent:latest
            with:
              param1: value1
            env:
              API_KEY: <+inputs.apiKey>
      platform:
        os: linux
        arch: arm64

  # Input definitions
  inputs:
    apiKey:
      type: secret
    repo:
      type: string
    branch:
      type: string
      default: main
```

### Clone Configuration

```yaml
# Branch clone
clone:
  depth: 1
  ref:
    name: <+inputs.branch>
    type: branch
  repo: <+inputs.repo>
  connector: "<+inputs.gitConnector != null ? inputs.gitConnector.id : ''>"

# Pull request clone
clone:
  depth: 1000
  ref:
    type: pull-request
    number: <+inputs.pullReq>
  repo: <+inputs.repo>
```

### Input Types

```yaml
inputs:
  # String input
  repo:
    type: string
    required: true

  # String with default
  branch:
    type: string
    default: main
    description: The branch to use

  # Secret input
  apiKey:
    type: secret
    default: account.my_secret

  # Connector input
  llmConnector:
    type: connector

  # Git connector
  gitConnector:
    type: connector
```

### Step Types

#### Container Step with Plugin

```yaml
- name: my-step
  run:
    container:
      image: myregistry/my-image:tag
    with:
      plugin_param: value
      another_param: <+inputs.param>
    env:
      ENV_VAR: value
      SECRET_VAR: <+inputs.secret>
```

#### Shell Script Step

```yaml
- name: shell-step
  run:
    shell: bash
    script: |-
      echo "Running script"
      git add -A
      git diff --cached
    env:
      MY_VAR: value
```

### Platform Configuration

```yaml
platform:
  os: linux
  arch: arm64  # or amd64
```

### Expression Syntax

```yaml
# Input references
<+inputs.variableName>

# Connector token
<+inputs.connectorName.token>

# Step outputs
<+pipeline.stages.stage_name.steps.step_name.output.outputVariables.VAR_NAME>

# Environment variables
<+env.HARNESS_ACCOUNT_ID>
<+env.HARNESS_ORG_ID>
<+env.HARNESS_PROJECT_ID>
<+env.DRONE_REPO_SCM>
<+env.DRONE_OUTPUT>

# Alternative syntax for inputs in env
${{inputs.repo}}
```

## wiki.MD

Documentation template:

```markdown
# Agent Name

**Version:** 1.0.0
**Name:** Agent Name

## Overview

Brief description of what the agent does and why it's useful.
Focus on the value proposition and primary use case.

---

## Key Capabilities

- 🔍 **Capability 1**: Description
- 🛠 **Capability 2**: Description
- 🔁 **Capability 3**: Description
- 🧠 **Capability 4**: Description

---

## How It Works

1. **Step 1**: Description of first step
2. **Step 2**: Description of second step
3. **Step 3**: Description of third step

## Required Inputs

| Input | Type | Description | Default |
|-------|------|-------------|---------|
| `inputName` | type | Description | value |

## Configuration

- **Setting 1:** Value
- **Setting 2:** Value

## Usage Example

\`\`\`yaml
inputs:
  repo: "my-organization/my-repository"
  branch: "main"
\`\`\`

## Technical Details

### Container Images
- **Image Name**: `registry/image:tag`
- **Purpose**: What it does

### Security Considerations

- Point 1
- Point 2

## Troubleshooting

### Common Issue 1
Solution description

### Common Issue 2
Solution description

---

**Last Updated:** Month Year
```

## Complete Examples

### Code Review Agent

**metadata.json:**
```json
{
  "name": "Code Review",
  "description": "AI-powered agent that reviews code changes and comments on pull requests",
  "version": "1.0.0"
}
```

**pipeline.yaml:**
```yaml
version: 1
pipeline:
  clone:
    depth: 1000
    ref:
      type: pull-request
      number: <+inputs.pullReq>
    repo: <+inputs.repo>

  stages:
    - name: review
      steps:
        - name: review_prompt_generation
          run:
            container:
              image: myregistry/ai-review:latest
            with:
              output_file: /harness/review/task.txt
              review_output_file: /harness/review/review.json
              working_directory: /harness

        - name: coding_agent
          run:
            container:
              image: myregistry/coding-agent:latest
            with:
              detailed_logging: "true"
              max_iterations: "50"
              task_file_path: /harness/review/task.txt
              working_directory: /harness
            env:
              ANTHROPIC_API_KEY: <+inputs.llmConnector.token>

        - name: post_comments
          run:
            container:
              image: myregistry/comment-plugin:latest
            with:
              comments_file: /harness/review/review.json
              repo: <+inputs.repo>
              pr_number: <+inputs.pullReq>
            env:
              TOKEN: <+inputs.harnessKey>

      platform:
        os: linux
        arch: arm64

  inputs:
    llmConnector:
      type: connector
    harnessKey:
      type: secret
      default: account.harness_api_key
    repo:
      type: string
      required: true
    pullReq:
      type: string
      required: true
```

### Test Generator Agent

**metadata.json:**
```json
{
  "name": "Test Generator",
  "description": "AI-powered agent that analyzes code and generates comprehensive unit tests",
  "version": "1.0.0"
}
```

**pipeline.yaml:**
```yaml
version: 1
pipeline:
  clone:
    depth: 1
    ref:
      name: <+inputs.branch>
      type: branch
    repo: <+inputs.repo>
    connector: "<+inputs.gitConnector != null ? inputs.gitConnector.id : ''>"

  stages:
    - name: test-generator
      steps:
        - name: analyze_and_generate
          run:
            container:
              image: myregistry/test-generator:latest
            with:
              detailed_logging: "true"
              max_iterations: "100"
              working_directory: /harness
              target_coverage: "80"
              prompt: "Analyze the codebase and generate unit tests to achieve target coverage"
            env:
              ANTHROPIC_API_KEY: <+inputs.llmConnector.token>

        - name: show_changes
          run:
            shell: bash
            script: |-
              git add -A
              git diff --cached

        - name: detect_scm_provider
          run:
            shell: bash
            script: |-
              SCM_PROVIDER="${DRONE_REPO_SCM}"
              if [ "$SCM_PROVIDER" = "Git" ]; then
                SCM_PROVIDER="harness"
              fi
              SCM_PROVIDER=$(echo "$SCM_PROVIDER" | tr '[:upper:]' '[:lower:]')

              if [ "$SCM_PROVIDER" = "harness" ]; then
                if [ -n "$HARNESS_API_KEY" ]; then
                  TOKEN="${HARNESS_API_KEY}"
                else
                  TOKEN="${SCM_TOKEN}"
                fi
                NETRC_USERNAME="${DRONE_NETRC_USERNAME}"
                NETRC_PASSWORD="${DRONE_NETRC_PASSWORD}"
              else
                if [ -n "$DRONE_REPO_LINK" ]; then
                  NETRC_USERNAME=$(echo "$DRONE_REPO_LINK" | sed -E 's|https?://[^/]+/([^/]+)/.*|\1|')
                else
                  NETRC_USERNAME="${DRONE_NETRC_USERNAME}"
                fi
                TOKEN="${SCM_TOKEN}"
                NETRC_PASSWORD="${SCM_TOKEN}"
              fi

              BASE_URL="${DRONE_SYSTEM_PROTO}://${DRONE_SYSTEM_HOST}"

              echo "SCM_PROVIDER=$SCM_PROVIDER" >> $DRONE_OUTPUT
              echo "TOKEN=$TOKEN" >> $HARNESS_OUTPUT_SECRET_FILE
              echo "NETRC_USERNAME=$NETRC_USERNAME" >> $DRONE_OUTPUT
              echo "NETRC_PASSWORD=$NETRC_PASSWORD" >> $HARNESS_OUTPUT_SECRET_FILE
              echo "BASE_URL=$BASE_URL" >> $DRONE_OUTPUT
            env:
              HARNESS_API_KEY: <+inputs.harnessKey>
              SCM_TOKEN: <+inputs.gitConnector.token>

        - name: create_pr
          run:
            container:
              image: myregistry/create-pr-plugin:latest
            env:
              PLUGIN_SCM_PROVIDER: <+pipeline.stages.testgenerator_1.steps.detect_scm_provider_1.output.outputVariables.SCM_PROVIDER>
              PLUGIN_TOKEN: <+pipeline.stages.testgenerator_1.steps.detect_scm_provider_1.output.outputVariables.TOKEN>
              PLUGIN_REPO: ${{inputs.repo}}
              PLUGIN_SOURCE_BRANCH: ${{inputs.branch}}
              PLUGIN_NETRC_MACHINE: <+env.DRONE_NETRC_MACHINE>
              PLUGIN_NETRC_USERNAME: <+pipeline.stages.testgenerator_1.steps.detect_scm_provider_1.output.outputVariables.NETRC_USERNAME>
              PLUGIN_NETRC_PASSWORD: <+pipeline.stages.testgenerator_1.steps.detect_scm_provider_1.output.outputVariables.NETRC_PASSWORD>
              PLUGIN_BRANCH_SUFFIX: ai-tests
              PLUGIN_COMMIT_MESSAGE: "AI Generated Tests by Harness"
              PLUGIN_PUSH_CHANGES: "true"
              PLUGIN_CREATE_PR: "true"
              PLUGIN_PR_TITLE: "AI Generated Unit Tests"
              PLUGIN_PR_DESCRIPTION: "Automated unit test generation. Please review before merging."
              PLUGIN_HARNESS_ACCOUNT_ID: <+env.HARNESS_ACCOUNT_ID>
              PLUGIN_HARNESS_ORG_ID: <+env.HARNESS_ORG_ID>
              PLUGIN_HARNESS_PROJECT_ID: <+env.HARNESS_PROJECT_ID>
              PLUGIN_HARNESS_BASE_URL: <+pipeline.stages.testgenerator_1.steps.detect_scm_provider_1.output.outputVariables.BASE_URL>

      platform:
        os: linux
        arch: arm64

  inputs:
    llmConnector:
      type: connector
      description: LLM connector for AI operations
    harnessKey:
      type: secret
      default: harness_api_key
    gitConnector:
      type: connector
      description: Git connector for repository access
    repo:
      type: string
      required: true
    branch:
      type: string
      default: main
```

### Security Scanner Agent

**metadata.json:**
```json
{
  "name": "Security Scanner",
  "description": "AI-powered agent that scans code for security vulnerabilities and suggests fixes",
  "version": "1.0.0"
}
```

**pipeline.yaml:**
```yaml
version: 1
pipeline:
  clone:
    depth: 1
    ref:
      name: <+inputs.branch>
      type: branch
    repo: <+inputs.repo>
    connector: "<+inputs.gitConnector != null ? inputs.gitConnector.id : ''>"

  stages:
    - name: security-scan
      steps:
        - name: vulnerability_scanner
          run:
            container:
              image: myregistry/security-scanner:latest
            with:
              scan_type: "full"
              severity_threshold: "medium"
              output_file: /harness/security/findings.json
              working_directory: /harness
            env:
              ANTHROPIC_API_KEY: <+inputs.llmConnector.token>

        - name: generate_report
          run:
            container:
              image: myregistry/report-generator:latest
            with:
              findings_file: /harness/security/findings.json
              report_file: /harness/security/SECURITY_REPORT.md
              format: markdown

        - name: post_findings
          run:
            container:
              image: myregistry/pr-comment:latest
            with:
              report_file: /harness/security/SECURITY_REPORT.md
              repo: <+inputs.repo>
              pr_number: <+inputs.pullReq>
            env:
              TOKEN: <+inputs.harnessKey>
          if: <+inputs.pullReq> != ""

      platform:
        os: linux
        arch: arm64

  inputs:
    llmConnector:
      type: connector
    harnessKey:
      type: secret
      default: harness_api_key
    gitConnector:
      type: connector
    repo:
      type: string
      required: true
    branch:
      type: string
      default: main
    pullReq:
      type: string
      default: ""
      description: Pull request number (optional, for PR comments)
```

### Documentation Generator Agent

**metadata.json:**
```json
{
  "name": "Documentation Generator",
  "description": "AI-powered agent that analyzes code and generates comprehensive documentation",
  "version": "1.0.0"
}
```

**pipeline.yaml:**
```yaml
version: 1
pipeline:
  clone:
    depth: 1
    ref:
      name: <+inputs.branch>
      type: branch
    repo: <+inputs.repo>
    connector: "<+inputs.gitConnector != null ? inputs.gitConnector.id : ''>"

  stages:
    - name: doc-generator
      steps:
        - name: analyze_code
          run:
            container:
              image: myregistry/doc-analyzer:latest
            with:
              working_directory: /harness
              output_dir: /harness/docs
              doc_type: <+inputs.docType>
            env:
              ANTHROPIC_API_KEY: <+inputs.llmConnector.token>

        - name: generate_docs
          run:
            container:
              image: myregistry/coding-agent:latest
            with:
              detailed_logging: "true"
              max_iterations: "50"
              working_directory: /harness
              prompt: |
                Generate comprehensive documentation for this codebase:
                - README.md with project overview
                - API documentation
                - Usage examples
                - Architecture diagrams (mermaid)
            env:
              ANTHROPIC_API_KEY: <+inputs.llmConnector.token>

        - name: commit_and_pr
          run:
            shell: bash
            script: |-
              git add -A
              if git diff --cached --quiet; then
                echo "No changes to commit"
                exit 0
              fi
              git commit -m "docs: AI-generated documentation"
              # PR creation logic here

      platform:
        os: linux
        arch: arm64

  inputs:
    llmConnector:
      type: connector
    gitConnector:
      type: connector
    repo:
      type: string
      required: true
    branch:
      type: string
      default: main
    docType:
      type: string
      default: full
      description: Documentation type (full, api, readme)
```

## Common Patterns

### SCM Provider Detection

Standard pattern for multi-SCM support:

```yaml
- name: detect_scm_provider
  run:
    shell: bash
    script: |-
      SCM_PROVIDER="${DRONE_REPO_SCM}"
      if [ "$SCM_PROVIDER" = "Git" ]; then
        SCM_PROVIDER="harness"
      fi
      SCM_PROVIDER=$(echo "$SCM_PROVIDER" | tr '[:upper:]' '[:lower:]')

      if [ "$SCM_PROVIDER" = "harness" ]; then
        TOKEN="${HARNESS_API_KEY:-$SCM_TOKEN}"
        NETRC_USERNAME="${DRONE_NETRC_USERNAME}"
        NETRC_PASSWORD="${DRONE_NETRC_PASSWORD}"
      else
        NETRC_USERNAME=$(echo "$DRONE_REPO_LINK" | sed -E 's|https?://[^/]+/([^/]+)/.*|\1|')
        TOKEN="${SCM_TOKEN}"
        NETRC_PASSWORD="${SCM_TOKEN}"
      fi

      BASE_URL="${DRONE_SYSTEM_PROTO}://${DRONE_SYSTEM_HOST}"

      echo "SCM_PROVIDER=$SCM_PROVIDER" >> $DRONE_OUTPUT
      echo "TOKEN=$TOKEN" >> $HARNESS_OUTPUT_SECRET_FILE
      echo "NETRC_USERNAME=$NETRC_USERNAME" >> $DRONE_OUTPUT
      echo "NETRC_PASSWORD=$NETRC_PASSWORD" >> $HARNESS_OUTPUT_SECRET_FILE
      echo "BASE_URL=$BASE_URL" >> $DRONE_OUTPUT
    env:
      HARNESS_API_KEY: <+inputs.harnessKey>
      SCM_TOKEN: <+inputs.gitConnector.token>
```

### PR Creation Plugin

Standard pattern for creating PRs:

```yaml
- name: create_pr
  run:
    container:
      image: myregistry/create-pr-plugin:latest
    env:
      PLUGIN_SCM_PROVIDER: <+pipeline.stages.STAGE.steps.detect_scm_provider.output.outputVariables.SCM_PROVIDER>
      PLUGIN_TOKEN: <+pipeline.stages.STAGE.steps.detect_scm_provider.output.outputVariables.TOKEN>
      PLUGIN_REPO: ${{inputs.repo}}
      PLUGIN_SOURCE_BRANCH: ${{inputs.branch}}
      PLUGIN_NETRC_MACHINE: <+env.DRONE_NETRC_MACHINE>
      PLUGIN_NETRC_USERNAME: <+pipeline.stages.STAGE.steps.detect_scm_provider.output.outputVariables.NETRC_USERNAME>
      PLUGIN_NETRC_PASSWORD: <+pipeline.stages.STAGE.steps.detect_scm_provider.output.outputVariables.NETRC_PASSWORD>
      PLUGIN_BRANCH_SUFFIX: ai-changes
      PLUGIN_UNIQUE_PER_EXECUTION: "false"
      PLUGIN_FORCE_PUSH: "true"
      PLUGIN_COMMIT_MESSAGE: "AI: Changes by Harness Agent"
      PLUGIN_PUSH_CHANGES: "true"
      PLUGIN_CREATE_PR: "true"
      PLUGIN_PR_TITLE: "AI: Automated changes"
      PLUGIN_PR_DESCRIPTION: "Automated changes by Harness AI Agent"
      PLUGIN_IS_DRAFT: "false"
      PLUGIN_BYPASS_RULES: "false"
      PLUGIN_HARNESS_ACCOUNT_ID: <+env.HARNESS_ACCOUNT_ID>
      PLUGIN_HARNESS_ORG_ID: <+env.HARNESS_ORG_ID>
      PLUGIN_HARNESS_PROJECT_ID: <+env.HARNESS_PROJECT_ID>
      PLUGIN_HARNESS_BASE_URL: <+pipeline.stages.STAGE.steps.detect_scm_provider.output.outputVariables.BASE_URL>
```

### Coding Agent Step

Standard AI coding agent pattern:

```yaml
- name: coding_agent
  run:
    container:
      image: myregistry/coding-agent:latest
    with:
      detailed_logging: "true"
      max_iterations: "50"
      task_file_path: /harness/task.txt
      working_directory: /harness
      show_diff: "false"
    env:
      ANTHROPIC_API_KEY: <+inputs.llmConnector.token>
```

## Best Practices

### Security

- Never hardcode secrets or credentials
- Use `type: secret` for sensitive inputs
- Use `type: connector` for authentication
- Store outputs securely with `$HARNESS_OUTPUT_SECRET_FILE`

### Documentation

- Always include a comprehensive wiki.MD
- Document all inputs with types and defaults
- Provide usage examples
- Include troubleshooting section

### Naming

- Use Sentence Case for agent names: `Code Review`, `Test Generator`
- Use lowercase, hyphen-separated directory names: `code-review`, `test-generator`

### Pipeline Design

- Keep stages focused on single responsibilities
- Use meaningful step names
- Include proper error handling
- Set appropriate timeouts

## Instructions

When a user requests an agent template:

1. **Clarify requirements:**
   - What should the agent do?
   - What inputs does it need?
   - What SCM providers should it support?
   - Should it create PRs?

2. **Generate all required files:**
   - `metadata.json` with name, description, version
   - `pipeline.yaml` with inputs, stages, steps
   - `wiki.MD` with documentation

3. **Follow patterns:**
   - Use standard SCM detection for multi-provider support
   - Use standard PR creation plugin pattern
   - Include proper secret handling

4. **Output each file** in separate code blocks with filenames.
