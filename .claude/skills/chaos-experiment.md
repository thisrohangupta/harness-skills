---
name: chaos-experiment
description: Create and manage chaos experiments using Harness Chaos Engineering MCP tools. Test system resilience with controlled failure injection.
triggers:
  - chaos experiment
  - chaos engineering
  - resilience test
  - fault injection
  - chaos test
  - litmus chaos
  - reliability testing
---

# Chaos Experiment Skill

Create and manage chaos experiments using Harness Chaos Engineering MCP tools.

## Overview

This skill helps SRE and platform teams:
- List and browse existing chaos experiments
- Create new experiments from templates
- Run chaos experiments
- Analyze experiment results
- Design resilience test strategies

## Required MCP Tools

This skill requires the Harness MCP Server with the `chaos` toolset enabled:

- `chaos_experiments_list` - List all experiments
- `chaos_experiment_describe` - Get experiment details
- `chaos_experiment_run` - Execute an experiment
- `chaos_experiment_run_result` - Get run results
- `chaos_probes_list` - List resilience probes
- `chaos_probe_describe` - Get probe details
- `chaos_create_experiment_from_template` - Create from template
- `chaos_experiment_template_list` - List available templates
- `chaos_experiment_variables_list` - List experiment variables

## Workflow

### Step 1: List Available Templates

See what experiment templates are available:

```
Use MCP tool: chaos_experiment_template_list
Parameters:
  - org_id: <organization>
  - project_id: <project>
```

Common templates:
- Pod Delete
- Pod CPU Hog
- Pod Memory Hog
- Pod Network Loss
- Node Drain
- Node CPU Hog
- AWS EC2 Stop

### Step 2: List Existing Experiments

View experiments in the project:

```
Use MCP tool: chaos_experiments_list
Parameters:
  - org_id: <organization>
  - project_id: <project>
  - page_size: 20
```

### Step 3: Describe an Experiment

Get details of a specific experiment:

```
Use MCP tool: chaos_experiment_describe
Parameters:
  - experiment_id: <experiment identifier>
  - org_id: <organization>
  - project_id: <project>
```

Returns:
- Experiment name and description
- Target infrastructure
- Fault types included
- Probes attached
- Schedule (if any)

### Step 4: Create New Experiment

Create from a template:

```
Use MCP tool: chaos_create_experiment_from_template
Parameters:
  - template_id: <template identifier>
  - experiment_name: <name>
  - description: <description>
  - target_namespace: <kubernetes namespace>
  - target_label_selector: <label selector>
  - org_id: <organization>
  - project_id: <project>
```

### Step 5: Run Experiment

Execute the chaos experiment:

```
Use MCP tool: chaos_experiment_run
Parameters:
  - experiment_id: <experiment identifier>
  - org_id: <organization>
  - project_id: <project>
```

### Step 6: Get Run Results

Check experiment results:

```
Use MCP tool: chaos_experiment_run_result
Parameters:
  - experiment_id: <experiment identifier>
  - run_id: <run identifier>
  - org_id: <organization>
  - project_id: <project>
```

### Step 7: List Probes

View resilience probes:

```
Use MCP tool: chaos_probes_list
Parameters:
  - org_id: <organization>
  - project_id: <project>
```

Describe specific probe:

```
Use MCP tool: chaos_probe_describe
Parameters:
  - probe_id: <probe identifier>
  - org_id: <organization>
  - project_id: <project>
```

## Response Format

### Experiment Overview

```markdown
## Chaos Experiments Overview

**Project:** production-chaos
**Total Experiments:** 12
**Last Run:** 2 hours ago

### Experiments by Category

| Category | Count | Last Success |
|----------|-------|--------------|
| Pod Faults | 5 | 2h ago |
| Network Faults | 3 | 1d ago |
| Node Faults | 2 | 3d ago |
| Resource Faults | 2 | 1w ago |

### Recent Experiments

| Experiment | Type | Last Run | Result |
|------------|------|----------|--------|
| api-pod-delete | Pod Delete | 2h ago | ✅ Pass |
| payment-network-loss | Network Loss | 1d ago | ✅ Pass |
| checkout-cpu-stress | CPU Hog | 3d ago | ⚠️ Partial |
| db-node-drain | Node Drain | 1w ago | ✅ Pass |

### Scheduled Experiments

| Experiment | Schedule | Next Run |
|------------|----------|----------|
| weekly-resilience-suite | Weekly (Mon 2am) | In 3 days |
| daily-pod-chaos | Daily (3am) | In 8 hours |
```

### Experiment Details

```markdown
## Experiment: api-gateway-pod-delete

**ID:** exp_abc123
**Status:** Active
**Last Run:** Passed (2 hours ago)

### Description

Tests the resilience of the API gateway by randomly deleting pods
and verifying the service recovers within acceptable time.

### Target

| Property | Value |
|----------|-------|
| Infrastructure | prod-k8s-cluster |
| Namespace | production |
| Label Selector | app=api-gateway |
| Target Pods | 1 (random) |

### Fault Configuration

| Property | Value |
|----------|-------|
| Fault Type | Pod Delete |
| Duration | 30s |
| Interval | N/A (single delete) |
| Force Delete | false |

### Probes

| Probe | Type | Mode | Timeout |
|-------|------|------|---------|
| api-health-check | HTTP | Continuous | 5s |
| response-time | Prometheus | EOT | 10s |

### Success Criteria

- HTTP probe returns 200 within 30s of pod deletion
- Response time stays under 500ms (P99)
- No error rate spike above 1%

### Schedule

- **Frequency:** Daily at 3:00 AM UTC
- **Next Run:** In 8 hours
```

### Run Results

```markdown
## Experiment Run Results

**Experiment:** api-gateway-pod-delete
**Run ID:** run_xyz789
**Status:** ✅ Passed
**Duration:** 2m 15s

### Timeline

| Time | Event |
|------|-------|
| 00:00 | Experiment started |
| 00:05 | Pre-chaos probes passed |
| 00:10 | Fault injection: Deleted pod api-gateway-7d9f8-abc12 |
| 00:15 | Kubernetes creating replacement pod |
| 00:35 | New pod api-gateway-7d9f8-def34 running |
| 00:40 | Health check probe passing |
| 01:30 | Chaos duration complete |
| 02:00 | Post-chaos probes passed |
| 02:15 | Experiment completed |

### Probe Results

| Probe | Status | Details |
|-------|--------|---------|
| api-health-check | ✅ Pass | Recovered in 25s (threshold: 30s) |
| response-time | ✅ Pass | P99: 320ms (threshold: 500ms) |
| error-rate | ✅ Pass | Max: 0.3% (threshold: 1%) |

### Resilience Score

**Score: 95/100** ⭐

| Metric | Score | Weight |
|--------|-------|--------|
| Recovery Time | 25/25 | 25% |
| Error Rate | 25/25 | 25% |
| Probe Success | 25/25 | 25% |
| Overall Stability | 20/25 | 25% |

### Observations

1. **Pod Recovery:** New pod was scheduled within 5s
2. **Service Continuity:** Other pods handled traffic during recovery
3. **Health Checks:** Kubernetes readiness probe worked correctly
4. **No Cascade Failures:** Downstream services unaffected

### Recommendations

✅ System is resilient to single pod failures
📝 Consider testing multi-pod failure scenario
📝 Add chaos experiment for network partition
```

### Create Experiment Report

```markdown
## New Chaos Experiment Created

**Name:** checkout-service-cpu-stress
**ID:** exp_new456
**Template:** pod-cpu-hog

### Configuration

| Property | Value |
|----------|-------|
| Target Namespace | production |
| Target Service | checkout-service |
| Label Selector | app=checkout |
| CPU Cores | 2 |
| Duration | 60s |

### Probes Attached

1. **checkout-health** (HTTP)
   - URL: http://checkout-service/health
   - Interval: 5s
   - Success: status == 200

2. **checkout-latency** (Prometheus)
   - Query: histogram_quantile(0.99, checkout_request_duration)
   - Threshold: < 1000ms

### Next Steps

1. Review experiment configuration
2. Run in dry-run mode first
3. Schedule for low-traffic period
4. Monitor during execution

**Run Command:**
```
/chaos-experiment run checkout-service-cpu-stress
```
```

## Common Fault Types

### Pod Faults

| Fault | Description | Use Case |
|-------|-------------|----------|
| Pod Delete | Delete target pods | Test pod recovery |
| Pod CPU Hog | Stress pod CPU | Test CPU limits |
| Pod Memory Hog | Stress pod memory | Test OOM handling |
| Pod Network Loss | Drop network packets | Test network resilience |
| Pod Network Latency | Add network delay | Test timeout handling |
| Container Kill | Kill container process | Test restart policies |

### Node Faults

| Fault | Description | Use Case |
|-------|-------------|----------|
| Node Drain | Drain node of pods | Test node maintenance |
| Node CPU Hog | Stress node CPU | Test node capacity |
| Node Memory Hog | Stress node memory | Test node limits |
| Node Taint | Add node taint | Test pod scheduling |
| Kubelet Restart | Restart kubelet | Test control plane |

### Network Faults

| Fault | Description | Use Case |
|-------|-------------|----------|
| Network Loss | Drop packets | Test retry logic |
| Network Latency | Add delay | Test timeouts |
| Network Corruption | Corrupt packets | Test error handling |
| DNS Chaos | Disrupt DNS | Test DNS fallback |

### Cloud Faults

| Fault | Description | Use Case |
|-------|-------------|----------|
| EC2 Stop | Stop EC2 instance | Test auto-scaling |
| EBS Detach | Detach EBS volume | Test storage resilience |
| AZ Failure | Simulate AZ outage | Test multi-AZ |
| RDS Failover | Trigger RDS failover | Test DB resilience |

## Common Scenarios

### 1. List Experiments

```
/chaos-experiment

Show me all chaos experiments in the production project
```

### 2. Run Existing Experiment

```
/chaos-experiment

Run the api-pod-delete experiment
```

### 3. Create New Experiment

```
/chaos-experiment

Create a pod-delete chaos experiment for the
payment-service that tests recovery time
```

### 4. Check Results

```
/chaos-experiment

What were the results of the last chaos experiment run?
```

### 5. Design Resilience Test

```
/chaos-experiment

Design a comprehensive resilience test suite for
our checkout microservice
```

### 6. Troubleshoot Failure

```
/chaos-experiment

The last chaos experiment failed - help me understand why
```

## Probe Types

### HTTP Probe

Tests HTTP endpoint availability:

```yaml
type: httpProbe
httpProbe:
  url: http://service/health
  method: GET
  expectedResponseCode: "200"
  interval: 5s
  timeout: 3s
```

### Command Probe

Runs a command and checks exit code:

```yaml
type: cmdProbe
cmdProbe:
  command: "kubectl get pods -l app=myapp"
  expectedResult: "Running"
  interval: 10s
```

### Prometheus Probe

Queries Prometheus metrics:

```yaml
type: promProbe
promProbe:
  endpoint: http://prometheus:9090
  query: "sum(rate(http_errors[5m]))"
  comparator:
    type: "<"
    value: "0.01"
```

### Kubernetes Probe

Checks Kubernetes resource state:

```yaml
type: k8sProbe
k8sProbe:
  group: apps
  version: v1
  resource: deployments
  operation: present
```

## Example Usage

### Quick Status

```
/chaos-experiment

What chaos experiments do we have?
```

### Run Experiment

```
/chaos-experiment

Run the weekly resilience test
```

### Create Experiment

```
/chaos-experiment

Create a network latency experiment for the api-gateway
that adds 200ms delay for 60 seconds
```

### Analyze Results

```
/chaos-experiment

Analyze the results of today's chaos runs
and identify any services that need improvement
```

## Error Handling

### Common Errors

| Error Code | Description | Solution |
|------------|-------------|----------|
| `EXPERIMENT_NOT_FOUND` | Experiment doesn't exist | Verify experiment ID and project |
| `INFRASTRUCTURE_OFFLINE` | Chaos infrastructure down | Check chaos agent status |
| `TARGET_NOT_FOUND` | Target pods/nodes not found | Verify label selectors |
| `PROBE_FAILED` | Resilience probe failed | Check probe configuration |
| `PERMISSION_DENIED` | Cannot inject fault | Verify RBAC permissions |

### MCP Tool Errors

```
# Common MCP tool issues:

# Chaos tools not available
Error: Tool 'chaos_experiments_list' not found
→ Ensure 'chaos' toolset is enabled in MCP server config

# Infrastructure not connected
Error: No chaos infrastructure available
→ Install and configure chaos infrastructure

# Template not found
Error: Template 'pod-delete' not found
→ Verify template ID and project scope
```

## Troubleshooting

### Experiment Won't Start

1. **Check infrastructure:**
   - Verify chaos infrastructure installed
   - Check chaos agent pod running
   - Ensure agent connected to Harness

2. **Target validation:**
   - Verify target pods/nodes exist
   - Check label selector matches
   - Ensure namespace accessible

3. **Permissions:**
   - Verify chaos RBAC configured
   - Check service account permissions
   - Review pod security policies

### Fault Injection Failures

1. **Pod faults:**
   - Verify pod exists and running
   - Check container runtime compatible
   - Ensure sidecar injection works

2. **Network faults:**
   - Check network policy allows traffic
   - Verify CNI plugin compatible
   - Ensure iptables/tc available

3. **Node faults:**
   - Verify node-level access
   - Check privileged permissions
   - Ensure host access available

### Probe Issues

1. **HTTP probe failing:**
   - Verify endpoint accessible
   - Check URL and port correct
   - Review expected response code

2. **Command probe failing:**
   - Verify command syntax
   - Check execution context
   - Review expected output

3. **Prometheus probe failing:**
   - Verify Prometheus endpoint
   - Check query syntax
   - Ensure metrics available

### Experiment Results Unclear

1. **Analyze probe data:**
   - Review all probe results
   - Check timing of failures
   - Correlate with fault injection

2. **Verify steady state:**
   - Ensure pre-chaos checks passed
   - Verify baseline established
   - Check post-chaos recovery

3. **Insufficient data:**
   - Increase experiment duration
   - Add more probes
   - Run multiple iterations

### Infrastructure Connection Issues

1. **Agent not connecting:**
   - Check network connectivity
   - Verify agent token valid
   - Review firewall rules

2. **Agent unhealthy:**
   - Check agent pod logs
   - Verify resource availability
   - Review cluster health

## Instructions

When working with chaos experiments:

1. **Understand the goal:**
   - What resilience property to test?
   - What's the hypothesis?
   - What's acceptable behavior?

2. **Select appropriate fault:**
   - Match fault to scenario
   - Start with lighter faults
   - Progress to more severe

3. **Configure probes:**
   - Define success criteria
   - Set appropriate thresholds
   - Include steady-state checks

4. **Run safely:**
   - Start in non-prod
   - Use blast radius limits
   - Have rollback plan

5. **Analyze results:**
   - Review all probe results
   - Identify failure patterns
   - Document learnings
   - Plan improvements
