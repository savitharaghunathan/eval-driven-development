# Eval Plan Template

Use this structure when writing eval plan files in Phase 10.

```markdown
# Eval Plan: <Topic>

**Spec**: <path to the design spec this eval plan covers>
**Date**: YYYY-MM-DD
**Status**: Draft | Approved

## Summary

<1-2 sentences: what system/feature is being evaluated and what success looks like>

## LLM Call Path Inventory

| Path ID | Component | Purpose | Input | Output | Non-Deterministic |
|---------|-----------|---------|-------|--------|-------------------|
| llm-001 | ... | generation / classification / extraction / tool selection / ... | ... | ... | true / false |

## P0 Evals (Must-Pass)

These must pass before any deployment.

| ID | Requirement | Description | Grader Type | Grader Logic |
|----|-------------|-------------|-------------|--------------|
| p0-001 | ... | ... | code / llm-judge / outcome | ... |

### P0 Problem Sets

#### p0-001: <task name>

**Positive cases:**
- ...

**Negative cases:**
- ...

**Boundary cases:**
- ...

## P1 Evals (Core Functionality)

These must pass before the feature is considered complete.

| ID | Requirement | Description | Grader Type | Grader Logic |
|----|-------------|-------------|-------------|--------------|
| p1-001 | ... | ... | ... | ... |

### P1 Problem Sets

#### p1-001: <task name>
...

## P2 Evals (Quality)

Important but not blocking.

| ID | Requirement | Description | Grader Type | Grader Logic |
|----|-------------|-------------|-------------|--------------|
| p2-001 | ... | ... | ... | ... |

### P2 Problem Sets

#### p2-001: <task name>
...

## Pass Criteria

### Per-Task
- **Trials per task**: <number>
- **Metric**: pass@k / pass^k
- **P0 threshold**: <e.g., 95%>
- **P1 threshold**: <e.g., 80%>
- **P2 threshold**: <e.g., 70%>

### Suite-Level
- **Regression floor**: P0 evals must maintain near-100%
- **Graduation rule**: Capability eval at sustained >95% → regression suite
- **Saturation rule**: Regression eval at 100% for N runs → replace with harder tasks

## Non-Functional Metrics

| Metric | Budget | Gate Type |
|--------|--------|-----------|
| Cost per task | <budget> | Tracked / Hard gate |
| Latency per task | <budget> | Tracked / Hard gate |
| Token usage per task | <budget> | Tracked / Hard gate |
| Safety (adversarial pass rate) | <threshold> | Tracked / Hard gate |
| Robustness (edge case pass rate) | <threshold> | Tracked / Hard gate |
| Governance (privacy/deletion compliance) | <threshold> | Tracked / Hard gate |

## Harness Metrics (if applicable)

_Include this section if the system has orchestration infrastructure (routing, context assembly, tool dispatch, multi-agent coordination)._

| Metric | Budget | Gate Type |
|--------|--------|-----------|
| Routing accuracy (correct tool/agent selection) | <threshold> | Tracked / Hard gate |
| Context compaction fidelity (critical instructions survive compression) | <threshold> | Tracked / Hard gate |
| Sensor coverage (% of checks that fire on relevant input) | <threshold> | Tracked / Hard gate |
| Fault recovery rate (recovers vs. loops on injected errors) | <threshold> | Tracked / Hard gate |
| Premature completion rate | <threshold> | Tracked / Hard gate |
| Over-ambition rate (context exhaustion mid-task) | <threshold> | Tracked / Hard gate |
| Model transferability (pass rate delta across different LLMs) | <threshold> | Tracked / Hard gate |

## Memory Metrics (if applicable)

_Include this section if the system maintains persistent state across turns or sessions._

| Metric | Budget | Gate Type |
|--------|--------|-----------|
| Retrieval precision/recall | <threshold> | Tracked / Hard gate |
| Contradiction rate | <threshold> | Tracked / Hard gate |
| Staleness (% of recalled facts outdated) | <threshold> | Tracked / Hard gate |
| Selective forgetting accuracy | <threshold> | Tracked / Hard gate |
| Cross-session coherence | <threshold> | Tracked / Hard gate |
| Consolidation fidelity | <threshold> | Tracked / Hard gate |
| Storage growth rate | <budget> | Tracked / Hard gate |

## Eval Harness Requirements

- **Isolation**: <how trials are isolated — containers, temp dirs, DB snapshots>
- **Providers**: <which LLM providers are tested>
- **Judge model**: <which model is used for LLM-as-judge graders>
- **Human calibration**: <plan for calibrating judges — N examples, cadence>
```
