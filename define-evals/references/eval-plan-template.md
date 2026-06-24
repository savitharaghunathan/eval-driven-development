# Eval Plan Template

Use this structure when writing eval plan files in Phase 10.

```markdown
# Eval Plan: <Topic>

**Spec**: <path to the design spec this eval plan covers>
**Date**: YYYY-MM-DD
**Status**: Draft | Approved

## Summary

<1-2 sentences: what system/feature is being evaluated and what success looks like>

**Archetypes**: <matched archetypes, e.g., single-agent, rag-pipeline>
**Primary archetype**: <the one leading the eval plan>

## LLM Call Path Inventory

| Path ID | Component | Purpose | Input | Output | Non-Deterministic |
|---------|-----------|---------|-------|--------|-------------------|
| llm-001 | ... | generation / classification / extraction / tool selection / ... | ... | ... | true / false |

## Non-LLM Call Path Inventory (if applicable)

_Include this section for ml-pipeline, data-workflow, or api-service archetypes._

| Path ID | Component | Purpose | Input | Output |
|---------|-----------|---------|-------|--------|
| nlm-001 | ... | model inference / data transformation / API integration / ... | ... | ... |

## P0 Evals (Must-Pass)

These must pass before any deployment.

| ID | Requirement | Description | Grader Type | Grader Logic |
|----|-------------|-------------|-------------|--------------|
| p0-001 | ... | ... | code / llm-judge / outcome | ... |

_The table summarizes each task. Input, expected_behavior, reference_solution, and category are detailed in the problem sets below. Priority is determined by section (P0/P1/P2)._

### P0 Problem Sets

#### p0-001: <task name>

**Input:** <the prompt, context, or scenario>

**Expected behavior:** <what success looks like>

**Reference solution:** <a known-good solution proving solvability>

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

## Archetype Metrics

### <Primary Archetype> Metrics

| Metric | Budget | Gate Type |
|--------|--------|-----------|
| <archetype-specific dimension> | <threshold> | Tracked / Hard gate |

### <Secondary Archetype> Metrics (if applicable)

| Metric | Budget | Gate Type |
|--------|--------|-----------|
| <additional dimensions from secondary archetype> | <threshold> | Tracked / Hard gate |

### Universal Metrics

| Metric | Budget | Gate Type |
|--------|--------|-----------|
| Cost per task | <budget> | Tracked / Hard gate |
| Safety (adversarial pass rate) | <threshold> | Tracked / Hard gate |
| Robustness (edge case pass rate) | <threshold> | Tracked / Hard gate |
| Governance (privacy/deletion compliance) | <threshold> | Tracked / Hard gate |
| Observability (trajectory captured) | <threshold> | Tracked / Hard gate |

## Framework Compatibility Notes (if applicable)

_Include this section when any eval task requires capabilities not universally supported across frameworks._

| Task ID | Capability Needed | Framework Support |
|---------|-------------------|-------------------|
| <id> | <e.g., multi-session state> | <e.g., Harbor: native, Inspect AI: custom setup> |

## Eval Harness Requirements

- **Isolation**: <how trials are isolated — containers, temp dirs, DB snapshots>
- **Providers**: <which LLM providers are tested>
- **Judge model**: <which model is used for LLM-as-judge graders>
- **Human calibration**: <plan for calibrating judges — N examples, cadence>
```
