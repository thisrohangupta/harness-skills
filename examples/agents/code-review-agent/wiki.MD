# Code Review Agent

**Version:** 1.0.0
**Name:** Code Review

## Overview

The Code Review Agent is an AI-powered automation that analyzes pull request changes and provides comprehensive code review feedback. It identifies code quality issues, suggests improvements, and posts review comments directly on your pull requests.

This agent helps development teams maintain code quality by providing consistent, thorough reviews that complement human reviewers.

---

## Key Capabilities

- **Code Quality Analysis**: Identifies bugs, anti-patterns, and code smells
- **Security Review**: Flags potential security vulnerabilities
- **Performance Insights**: Suggests performance optimizations
- **Style Consistency**: Checks adherence to coding standards
- **Multi-SCM Support**: Works with GitHub, GitLab, Bitbucket, and Harness Code

---

## How It Works

1. **Clone PR**: The agent clones the pull request with full history for context
2. **Generate Review Task**: Analyzes the diff and creates a review task
3. **AI Analysis**: Uses an AI coding agent for deep code analysis
4. **Detect SCM**: Identifies the source control provider for proper API calls
5. **Post Comments**: Posts review comments directly on the pull request

## Required Inputs

| Input | Type | Description | Default |
|-------|------|-------------|---------|
| `llmConnector` | connector | LLM connector (Anthropic, OpenAI) | - |
| `harnessKey` | secret | Harness API key | `account.harness_api_key` |
| `gitConnector` | connector | Git connector for repo access | - |
| `repo` | string | Repository (org/repo format) | Required |
| `pullReq` | string | Pull request number | Required |
| `reviewFocus` | string | Review focus area | `general` |

## Configuration

- **Review Focus Options:**
  - `general` - Comprehensive review covering all aspects
  - `security` - Focus on security vulnerabilities
  - `performance` - Focus on performance optimizations
  - `style` - Focus on code style and conventions

## Usage Example

```yaml
inputs:
  repo: "my-organization/my-repository"
  pullReq: "123"
  reviewFocus: "security"
```

## Technical Details

### Container Images

- **harness/ai-review-generator:latest** - Generates review tasks from PR diffs
- **harness/coding-agent:latest** - AI coding agent for deep analysis
- **harness/pr-comment-plugin:latest** - Posts comments to PRs

### Platform Requirements

- **OS**: Linux
- **Architecture**: arm64

### Security Considerations

- LLM API keys are passed as secrets
- Git tokens are handled securely via connectors
- No credentials are logged or exposed

## Troubleshooting

### Comments not appearing on PR

- Verify the `gitConnector` has write access to the repository
- Check that `harnessKey` is valid for Harness Code repos
- Ensure `pullReq` is a valid, open pull request number

### Review takes too long

- The agent may take longer for large PRs
- Consider setting a timeout on the pipeline
- Large diffs (1000+ lines) may require additional processing time

### Authentication errors

- Verify connector tokens are not expired
- Ensure the connector has appropriate repository permissions
- For Harness Code, verify the API key is active

---

**Last Updated:** February 2026
