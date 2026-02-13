---
name: security-report
description: Generate security compliance reports using Harness SCS and STO MCP tools. Analyze vulnerabilities, review SBOMs, and track security issues.
triggers:
  - security report
  - security scan
  - vulnerabilities
  - security issues
  - sbom
  - compliance report
  - security compliance
  - cve report
---

# Security Report Skill

Generate security compliance reports and analyze vulnerabilities using Harness Supply Chain Security (SCS) and Security Test Orchestration (STO) MCP tools.

## Overview

This skill helps security and development teams:
- View security scan results and vulnerabilities
- Generate compliance reports
- Download and analyze SBOMs (Software Bill of Materials)
- Track security issue remediation
- Manage security exemptions

## Required MCP Tools

This skill requires the Harness MCP Server with these toolsets:
- `scs` - Supply Chain Security tools
- `sto` - Security Test Orchestration tools

### SCS Tools
- `scs_list_artifacts_per_source` - List scanned artifacts
- `scs_get_artifact_overview` - Artifact security summary
- `scs_get_artifact_component_remediation` - Remediation guidance
- `scs_get_artifact_chain_of_custody` - Artifact provenance
- `scs_fetch_compliance_results_for_repo_by_id` - Repository compliance
- `scs_get_code_repository_overview` - Repository security overview
- `scs_download_sbom` - Download SBOM
- `scs_create_opa_policy` - Create compliance policies

### STO Tools
- `get_all_security_issues` - List security findings
- `global_exemptions` - View exemption rules
- `promote_exemption` - Promote exemption scope
- `approve_exemption` - Approve exemption request

## Workflow

### Step 1: Get Security Overview

For artifact-based security:

```
Use MCP tool: scs_get_artifact_overview
Parameters:
  - artifact_id: <artifact identifier>
  - org_id: <organization>
  - project_id: <project>
```

For repository security:

```
Use MCP tool: scs_get_code_repository_overview
Parameters:
  - repo_id: <repository identifier>
  - org_id: <organization>
  - project_id: <project>
```

### Step 2: List Security Issues

Get all security findings:

```
Use MCP tool: get_all_security_issues
Parameters:
  - org_id: <organization>
  - project_id: <project>
  - target: <optional target filter>
  - pipeline: <optional pipeline filter>
  - tool: <optional scanner filter>
  - severity: <optional severity filter>
  - exemption_status: <optional exemption filter>
  - page_size: 50
```

Filter options:
- **severity:** critical, high, medium, low, info
- **tool:** snyk, aqua_trivy, grype, etc.
- **exemption_status:** approved, pending, rejected, none

### Step 3: Get Artifact Details

List artifacts for a source:

```
Use MCP tool: scs_list_artifacts_per_source
Parameters:
  - source_type: "docker" (or git, etc.)
  - source_id: <source identifier>
  - org_id: <organization>
  - project_id: <project>
```

### Step 4: Get Remediation Guidance

For vulnerable components:

```
Use MCP tool: scs_get_artifact_component_remediation
Parameters:
  - artifact_id: <artifact identifier>
  - component_name: <vulnerable component>
  - org_id: <organization>
  - project_id: <project>
```

### Step 5: Check Compliance

Get repository compliance status:

```
Use MCP tool: scs_fetch_compliance_results_for_repo_by_id
Parameters:
  - repo_id: <repository identifier>
  - org_id: <organization>
  - project_id: <project>
```

### Step 6: Download SBOM

Get the Software Bill of Materials:

```
Use MCP tool: scs_download_sbom
Parameters:
  - artifact_id: <artifact identifier>
  - format: "spdx" (or cyclonedx)
  - org_id: <organization>
  - project_id: <project>
```

### Step 7: Review Chain of Custody

Verify artifact provenance:

```
Use MCP tool: scs_get_artifact_chain_of_custody
Parameters:
  - artifact_id: <artifact identifier>
  - org_id: <organization>
  - project_id: <project>
```

### Step 8: Manage Exemptions

View exemptions:

```
Use MCP tool: global_exemptions
Parameters:
  - org_id: <organization>
  - project_id: <project>
```

Approve exemption:

```
Use MCP tool: approve_exemption
Parameters:
  - exemption_id: <exemption identifier>
  - org_id: <organization>
  - project_id: <project>
```

## Response Format

### Security Summary Report

```markdown
## Security Compliance Report

**Generated:** <date>
**Scope:** <artifact/repository name>
**Last Scan:** <timestamp>

### Executive Summary

| Severity | Count | Trend |
|----------|-------|-------|
| Critical | 2 | ⬆️ +1 |
| High | 8 | ⬇️ -3 |
| Medium | 24 | → 0 |
| Low | 45 | ⬇️ -5 |

**Overall Risk Level:** HIGH
**Compliance Status:** FAILING (2 critical unresolved)

### Critical Vulnerabilities

#### CVE-2024-1234 - Remote Code Execution
**Component:** log4j 2.14.1
**CVSS:** 10.0
**Status:** Open
**Fix Available:** Yes - upgrade to 2.17.1

**Affected:**
- backend-service:v2.3.4
- worker-service:v1.2.0

**Remediation:**
```xml
<dependency>
  <groupId>org.apache.logging.log4j</groupId>
  <artifactId>log4j-core</artifactId>
  <version>2.17.1</version>
</dependency>
```

---

#### CVE-2024-5678 - SQL Injection
**Component:** mysql-connector 8.0.25
**CVSS:** 9.8
**Status:** Open
**Fix Available:** Yes - upgrade to 8.0.33

---

### High Vulnerabilities (Top 5)

| CVE | Component | CVSS | Fix Available |
|-----|-----------|------|---------------|
| CVE-2024-2345 | openssl 1.1.1k | 8.1 | Yes |
| CVE-2024-3456 | jackson-databind 2.12.3 | 7.5 | Yes |
| CVE-2024-4567 | spring-core 5.3.20 | 7.4 | Yes |
| CVE-2024-5678 | netty 4.1.65 | 7.2 | Yes |
| CVE-2024-6789 | guava 30.1 | 7.0 | Yes |

### Compliance Status

| Policy | Status | Details |
|--------|--------|---------|
| No Critical CVEs | ❌ Fail | 2 critical found |
| SBOM Generated | ✅ Pass | SPDX 2.3 |
| Signed Artifacts | ✅ Pass | Sigstore verified |
| License Compliance | ✅ Pass | No GPL violations |
| Approved Base Images | ✅ Pass | Using allowed images |
```

### SBOM Analysis Report

```markdown
## Software Bill of Materials Analysis

**Artifact:** backend-service:v2.3.4
**Format:** SPDX 2.3
**Generated:** <date>

### Component Summary

| Category | Count |
|----------|-------|
| Direct Dependencies | 45 |
| Transitive Dependencies | 312 |
| Total Components | 357 |

### License Distribution

| License | Count | Risk |
|---------|-------|------|
| Apache-2.0 | 180 | Low |
| MIT | 95 | Low |
| BSD-3-Clause | 42 | Low |
| GPL-3.0 | 3 | High |
| Unknown | 12 | Medium |

### High-Risk Licenses

⚠️ **GPL-3.0 Detected:**
- libgmp6 (transitive via libcrypto)
- Consider replacing or isolating

### Supply Chain Details

**Build Information:**
- Builder: Harness CI
- Build Time: 2024-01-15T10:30:00Z
- Source Commit: abc123def

**Verification:**
- SLSA Level: 3
- Signature: Valid (Sigstore)
- Attestation: Present
```

### Vulnerability Trend Report

```markdown
## Security Trend Analysis

**Period:** Last 30 Days
**Scope:** Production Artifacts

### Vulnerability Trend

```
Critical: ██░░░░░░░░ 2 (was 5)
High:     ████░░░░░░ 8 (was 15)
Medium:   ██████░░░░ 24 (was 28)
Low:      █████████░ 45 (was 52)
```

### Key Improvements

1. **Resolved:** CVE-2024-0001 (Critical) - Log4Shell variant
2. **Resolved:** CVE-2024-0002 (High) - Spring4Shell
3. **Mitigated:** 5 high-severity via WAF rules

### Remaining Priorities

1. **Upgrade jackson-databind** - Affects 12 services
2. **Replace mysql-connector** - Critical vulnerability
3. **Update base images** - 3 services on outdated alpine

### Metrics

| Metric | Current | Target | Status |
|--------|---------|--------|--------|
| MTTR (Critical) | 2.3 days | <3 days | ✅ |
| MTTR (High) | 8.5 days | <7 days | ⚠️ |
| Scan Coverage | 94% | 100% | ⚠️ |
| Fix Rate | 78% | 85% | ⚠️ |
```

## Common Scenarios

### 1. Pre-Release Security Check

```
/security-report

Generate a security report for backend-service:v2.3.4
before we release to production
```

### 2. Vulnerability Summary

```
/security-report

Show me all critical and high vulnerabilities
in the payments project
```

### 3. SBOM Generation

```
/security-report

Download the SBOM for our main API service
in CycloneDX format
```

### 4. Compliance Check

```
/security-report

Check if the checkout-service repository
meets our security compliance policies
```

### 5. Exemption Review

```
/security-report

Show me pending security exemptions
that need approval
```

### 6. Remediation Guidance

```
/security-report

How do I fix the log4j vulnerability
in the order-service?
```

## Security Issue Categories

### By Scanner Type
- **SAST:** Static Application Security Testing
- **DAST:** Dynamic Application Security Testing
- **SCA:** Software Composition Analysis
- **Container:** Container image scanning
- **IaC:** Infrastructure as Code scanning
- **Secrets:** Secret detection

### By Severity (CVSS)
- **Critical (9.0-10.0):** Immediate action required
- **High (7.0-8.9):** Fix within 7 days
- **Medium (4.0-6.9):** Fix within 30 days
- **Low (0.1-3.9):** Fix within 90 days
- **Info:** No fix required, informational

### By Status
- **Open:** Unresolved vulnerability
- **In Progress:** Remediation underway
- **Fixed:** Resolved in latest version
- **Exempted:** Accepted risk with approval
- **False Positive:** Not actually vulnerable

## Exemption Management

### When to Exempt

Valid exemption reasons:
- False positive confirmed
- Mitigated by other controls (WAF, network isolation)
- Not exploitable in context
- Fix breaks functionality, compensating controls in place

### Exemption Workflow

1. **Request:** Developer requests exemption with justification
2. **Review:** Security team reviews
3. **Approve/Reject:** Decision with conditions
4. **Monitor:** Track exemption expiration

### Exemption Report

```markdown
## Active Exemptions

| CVE | Component | Reason | Expires | Owner |
|-----|-----------|--------|---------|-------|
| CVE-2024-1234 | libxml2 | Mitigated by WAF | 2024-03-01 | @security |
| CVE-2024-2345 | openssl | Not exploitable | 2024-02-15 | @platform |

### Pending Approval

| CVE | Component | Requested By | Submitted |
|-----|-----------|--------------|-----------|
| CVE-2024-3456 | jackson | @dev-team | 2024-01-10 |
```

## Creating OPA Policies

For custom compliance rules:

```
Use MCP tool: scs_create_opa_policy
Parameters:
  - policy_name: "no-critical-cves"
  - policy_content: <rego policy>
  - org_id: <organization>
  - project_id: <project>
```

Example policy:

```rego
package harness.security

deny[msg] {
  input.vulnerabilities[_].severity == "CRITICAL"
  msg := "Critical vulnerabilities not allowed"
}
```

## Example Usage

### Quick Scan Summary

```
/security-report

Security summary for the api-gateway service
```

### Detailed CVE Report

```
/security-report

List all CVEs affecting production services
sorted by severity and show remediation steps
```

### Compliance Audit

```
/security-report

Generate a compliance report for SOC2 audit
covering all production artifacts
```

### Supply Chain Verification

```
/security-report

Verify the chain of custody and signatures
for the release candidate artifact
```

## Error Handling

### Common Errors

| Error Code | Description | Solution |
|------------|-------------|----------|
| `ARTIFACT_NOT_FOUND` | Artifact doesn't exist | Verify artifact ID and project scope |
| `SCAN_NOT_COMPLETE` | Security scan still running | Wait for scan completion |
| `NO_SCAN_DATA` | No security scans performed | Run security scan first |
| `SBOM_NOT_AVAILABLE` | SBOM not generated | Enable SBOM generation in pipeline |
| `EXEMPTION_NOT_FOUND` | Exemption ID invalid | Check exemption identifier |

### MCP Tool Errors

```
# Common MCP tool issues:

# SCS tools not available
Error: Tool 'scs_get_artifact_overview' not found
→ Ensure 'scs' toolset is enabled in MCP server config

# STO tools not available
Error: Tool 'get_all_security_issues' not found
→ Ensure 'sto' toolset is enabled in MCP server config

# No security data
Error: No scan results found for artifact
→ Verify artifact has been scanned
```

## Troubleshooting

### No Security Data Available

1. **Check scan configuration:**
   - Verify security scans enabled in pipeline
   - Check scanner step configuration
   - Ensure scans completed successfully

2. **Verify artifact ingestion:**
   - Check artifact was properly tagged
   - Verify SCS connector configuration
   - Review scan logs for errors

3. **Scanner connectivity:**
   - Verify scanner service accessible
   - Check scanner credentials
   - Review network policies

### SBOM Not Generated

1. **Enable SBOM generation:**
   - Add SBOM step to pipeline
   - Configure output format (SPDX/CycloneDX)
   - Verify generator tool works

2. **Check dependencies:**
   - Ensure package manager files present
   - Verify lockfiles committed
   - Check for unsupported languages

### Exemption Workflow Issues

1. **Cannot create exemption:**
   - Verify user has exemption permissions
   - Check vulnerability still exists
   - Ensure proper scope selected

2. **Exemption not applying:**
   - Verify exemption approved
   - Check exemption scope matches
   - Review exemption expiration date

3. **Approval workflow stuck:**
   - Check approvers configured
   - Verify notification delivery
   - Review approval policies

### Compliance Check Failures

1. **Policy evaluation errors:**
   - Verify OPA policy syntax
   - Check policy references valid
   - Review policy inputs

2. **Missing compliance data:**
   - Ensure all checks configured
   - Verify data sources connected
   - Check compliance rules active

## Instructions

When generating security reports:

1. **Understand the scope:**
   - Specific artifact or repository?
   - Project-wide or specific service?
   - Time period for trends?

2. **Gather security data:**
   - Fetch vulnerability information
   - Get compliance status
   - Check exemptions

3. **Prioritize findings:**
   - Lead with critical issues
   - Highlight what's actionable
   - Note what's already being addressed

4. **Provide remediation:**
   - Include fix versions
   - Show code/config changes
   - Suggest alternatives if no fix available

5. **Recommend next steps:**
   - Immediate actions for critical
   - Timeline for other severities
   - Process improvements
