# Archetype-Driven Eval Methodology

**Date:** 2026-06-23
**Status:** Draft
**Scope:** Improvements to the `define-evals` skill only. `review-evals` and `evolve-evals` are future work.

## Problem

The current `define-evals` skill has a free-form spec extraction phase ("read the spec and pull requirements") that misses important dimensions depending on the system type. A multi-agent orchestrator needs coordination and delegation evals. A user-facing product needs UX journey and accessibility evals. An ML pipeline needs statistical validation and drift detection. The current skill doesn't systematically extract these because it has no model of what kind of system the spec describes.

Additionally:
- The grader taxonomy is LLM-centric (code-based, LLM-as-judge, outcome verification) and lacks statistical validation for non-LLM systems
- The metric catalog is a flat list rather than domain-aware defaults
- The spec-to-eval mapping is implicit — there's no structured framework for how requirement types translate to eval patterns
- The end-to-end lifecycle (reviewing results, evolving evals) is not covered

## Solution: Spec Archetypes

Introduce **spec archetypes** — recognizable system patterns that each carry a structured extraction checklist and default eval dimensions. The skill identifies which archetype(s) a spec matches, then applies the right extraction patterns and metrics automatically.

A spec can match multiple archetypes. Each matched archetype contributes its checklist and metrics. Merge rules:

- **Primary archetype:** Ask the user which archetype is primary. Lead with that archetype's checklist and metrics, then layer in additions from secondary archetypes.
- **Dedup by intent:** When two archetypes contribute semantically equivalent dimensions (e.g., "tool selection accuracy" from single-agent and "delegation accuracy" from multi-agent), keep the one from the primary archetype.
- **Priority tiebreaking:** When two archetypes suggest the same dimension at different priorities, take the higher priority.
- **Presentation cap:** After merging, present at most 15-20 eval dimensions. Group the rest as "additional dimensions from [archetype]" that the user can opt into.

## Archetypes

### single-agent

**Signature patterns:** One LLM, tool use, single-turn or multi-turn interaction.

**Examples:** Coding assistant, search agent, CLI tool, file editor.

**Extraction checklist:**
- Tool roster (what tools are available, what do they do?)
- Decision points (where does the agent choose between actions?)
- Context window management (how does it handle long contexts?)
- Termination conditions (how does it know it's done?)
- Error handling (what happens when a tool call fails?)
- Retry/recovery strategy

**Default eval dimensions:**
- Tool selection accuracy
- Turn efficiency (resolved in under N turns)
- Cost per task (tokens, API calls)
- Premature completion rate
- Graceful termination (stops when done, not mid-task)
- Prompt injection resistance
- Context compaction fidelity (critical instructions survive window compression)
- Fault recovery (recovers from tool errors, timeouts, malformed responses)
- Skill/capability invocation accuracy (uses intended skill, avoids irrelevant ones)
- Memory quality (retrieval precision/recall, contradiction rate) — if persistent state
- Cross-session coherence — if multi-session

### multi-agent

**Signature patterns:** Multiple agents, delegation, coordination, shared state, supervisor/worker patterns.

**Examples:** Research swarm, pipeline of specialists, supervisor pattern, debate architecture.

**Extraction checklist:**
- Agent roster (who are all the agents, what are their roles?)
- Delegation patterns (who calls whom, under what conditions?)
- Shared state (what state is shared between agents, how is it synchronized?)
- Handoff protocols (how does work transfer between agents?)
- Failure cascades (if agent A fails, what happens to agents B and C?)
- Conflict resolution (what happens when two agents produce contradictory outputs?)
- Coordination overhead (how much extra cost/latency does coordination add?)
- Authority hierarchy (who has final say?)

**Default eval dimensions:**
- Delegation accuracy (correct specialist selected for sub-task)
- Handoff fidelity (information preserved across agent boundaries)
- Failure cascade containment (one agent's failure doesn't cascade)
- Coordination overhead (extra tokens/latency vs. single-agent baseline)
- Conflict resolution rate (contradictions resolved correctly)
- End-to-end task completion (the whole system achieves the goal)
- Deadlock/livelock detection (agents don't get stuck in loops)
- Routing accuracy (right specialist selected, degrades past 15-20 tools)
- Fault recovery (agent-level and system-level error handling)
- Memory quality and cross-session coherence — if shared/persistent state
- Consolidation fidelity (safety-critical info survives memory compression)

### rag-pipeline

**Signature patterns:** Retrieval, embedding, chunking, vector search, grounding, citation.

**Examples:** Knowledge base Q&A, document search, enterprise search, research assistant.

**Extraction checklist:**
- Corpus definition (what documents, what formats, how large?)
- Chunking strategy (how are documents split?)
- Embedding model and dimensions
- Retrieval method (vector search, hybrid, keyword)
- Re-ranking strategy (if any)
- Context assembly (how are retrieved chunks combined into a prompt?)
- Citation requirements (must the system attribute sources?)
- Freshness requirements (how current must the corpus be?)

**Default eval dimensions:**
- Retrieval precision/recall (right chunks retrieved)
- Groundedness (claims supported by retrieved sources)
- Hallucination boundary (declines when retrieval is empty or irrelevant)
- Source attribution accuracy (correct citations)
- Query understanding across phrasings (paraphrase robustness)
- Chunk relevance ranking (best chunks ranked highest)
- Retrieval latency
- Corpus coverage (what % of the corpus is reachable?)

### user-facing-product

**Signature patterns:** User journeys, UX flows, onboarding, accessibility, error messages, trust signals.

**Examples:** Chatbot, customer support agent, interactive assistant, form-filling agent.

**Extraction checklist:**
- User journeys (what are the key flows a user goes through?)
- Persona diversity (who are the different user types?)
- Error recovery UX (what does the user see when something fails?)
- Accessibility requirements (screen readers, keyboard nav, contrast, language)
- Onboarding flow (first-time user experience)
- Latency expectations (perceived responsiveness, not just p99)
- Trust signals (how does the user know the output is reliable?)
- Escalation paths (when and how does the user reach a human?)
- Tone and voice requirements (formal, casual, empathetic)
- Localization/i18n requirements

**Default eval dimensions:**
- Task completion rate (user's goal achieved)
- User journey completion rate (full flow completed without abandonment)
- Error recovery clarity (user understands what went wrong and what to do)
- Perceived latency (time to first useful token, streaming behavior)
- Accessibility compliance (WCAG level if specified)
- Trust calibration (does user confidence match actual accuracy?)
- Tone consistency (voice stays consistent across turns)
- Escalation accuracy (knows when to hand off to human)
- Turn efficiency (resolved without unnecessary back-and-forth)
- Refusal handling (declines out-of-scope gracefully)

### ml-pipeline

**Signature patterns:** Training, inference, model serving, feature engineering, data preprocessing, model versioning. Explicitly non-LLM or mixed LLM/traditional ML.

**Disambiguation:** Use ml-pipeline when there is a trained model (inference, training, fine-tuning). Use data-workflow when there is no trained model — pure ETL, reporting, analytics. If a system has both, match both archetypes; model-specific items come from ml-pipeline, data plumbing items from data-workflow.

**Examples:** Classification service, recommendation engine, anomaly detection, forecasting, computer vision pipeline.

**Extraction checklist:**
- Model type and architecture (what kind of model?)
- Training data requirements (source, size, labeling, splits)
- Feature engineering steps (transformations, derived features)
- Inference interface (API, batch, streaming?)
- Performance metric requirements (accuracy, precision, recall, F1, AUC, etc.)
- Data quality requirements (schema validation, missing values, outliers)
- Drift detection requirements (data drift, concept drift)
- Model versioning and rollback criteria
- Training reproducibility requirements
- Bias and fairness requirements

**Default eval dimensions:**
- Model performance (accuracy/precision/recall/F1/AUC as specified)
- Data quality compliance (schema valid, no unexpected nulls/outliers)
- Drift detection rate (catches distribution shifts)
- Training reproducibility (same data + config = same model within tolerance)
- Feature correctness (transformations produce expected outputs)
- Inference latency and throughput
- Rollback trigger accuracy (correctly identifies when to revert)
- Bias/fairness metrics (if specified in requirements)

### data-workflow

**Signature patterns:** ETL, data transformations, scheduling, data quality checks, reporting, analytics.

**Examples:** Data pipeline, report generator, analytics automation, data migration.

**Extraction checklist:**
- Data sources and sinks (where does data come from and go?)
- Transformation logic (what computations or restructuring?)
- Scheduling and orchestration (what triggers runs?)
- Data quality contracts (what guarantees does the pipeline make?)
- Error handling (what happens on partial failure, bad input?)
- Idempotency requirements (running twice = same result?)
- Lineage/provenance requirements (can you trace where data came from?)

**Default eval dimensions:**
- Transformation correctness (output matches expected)
- Idempotency (running twice produces same result)
- Schema compliance (output conforms to expected schema)
- Error handling coverage (graceful failure on malformed input)
- Throughput (processes within SLA)
- Data freshness (output available within required window)
- Lineage accuracy (provenance tracked correctly)
- Edge case handling (nulls, empty sets, large inputs, encoding issues)

### content-generation

**Signature patterns:** Writing, editing, style guides, brand voice, templates, formatting requirements.

**Examples:** Marketing copy, documentation generator, translation, email drafting, summarization.

**Extraction checklist:**
- Style guide or brand voice document
- Target audience definition
- Format requirements (length, structure, tone)
- Factual accuracy requirements (can it hallucinate?)
- Source material (what does it draw from?)
- Approval/review workflow (who signs off?)
- Localization requirements
- Legal/compliance constraints (disclaimers, required language)

**Default eval dimensions:**
- Style guide adherence (matches specified voice/tone)
- Factual accuracy (claims are verifiable)
- Audience appropriateness (matches target audience level)
- Format compliance (length, structure, heading usage)
- Originality (not regurgitating training data verbatim)
- Relevance (content addresses the specified topic)
- Consistency (maintains voice across sections/pieces)

### api-service

**Signature patterns:** API endpoints, tool definitions, MCP server, function-calling interface, request-response service consumed by other agents or systems.

**Examples:** MCP tool server, REST API, function-calling endpoint, webhook handler, data access layer.

**Extraction checklist:**
- API contract (OpenAPI spec, MCP tool definitions, schema)
- Parameter validation rules
- Error response format and codes
- Authentication/authorization model
- Rate limiting and throttling behavior
- Idempotency guarantees
- Backward compatibility constraints
- Timeout and retry expectations

**Default eval dimensions:**
- Contract compliance (responses match declared schema)
- Error format consistency (all errors follow the same envelope)
- Parameter validation coverage (rejects bad input, accepts good input)
- Idempotency (repeated calls produce same result)
- Latency percentiles (p50, p95, p99)
- Backward compatibility (old clients still work after changes)
- Auth enforcement (unauthorized requests rejected)

### Archetype Detection Notes

Signature patterns are starting points, not definitive classifiers. Key disambiguation rules:

- **single-agent vs. multi-agent:** Requires two or more distinct LLM-calling components that coordinate. A single agent with multiple tools is single-agent.
- **single-agent vs. content-generation:** content-generation applies when the system's primary purpose is producing written content for human consumption. A coding agent that generates code is single-agent.
- **user-facing-product vs. content-generation:** user-facing-product is interactive (conversations, forms, journeys). content-generation is batch/on-demand output.
- **ml-pipeline vs. data-workflow:** See disambiguation note under ml-pipeline.
- **api-service vs. single-agent:** api-service is a request-response service with no agency. If the service internally uses an LLM, match both api-service and single-agent.

Present detected archetypes to the user for confirmation before proceeding. The user can add or remove archetypes.

## Grader Taxonomy: Fourth Category

Add **statistical validation** as a fourth grader type alongside code-based, LLM-as-judge, and outcome verification:

### Statistical Validation

For requirements where correctness is a distribution property, not a binary outcome. Use when:

- Success is defined by metrics within confidence intervals (accuracy > 90% with p < 0.05)
- The system processes batches where individual items vary but aggregate behavior must be consistent
- Drift detection requires comparing distributions over time
- A/B testing requires statistical significance

Examples: model accuracy within threshold, data distribution stability, performance regression detection, bias measurement.

Methods:

| Method | Best For | Min Sample Size |
|--------|----------|-----------------|
| Confidence interval | Accuracy/F1 threshold validation | 30+ |
| Two-sample t-test | A/B testing, regression detection | 30+ per group |
| Chi-squared test | Classification distribution shifts | 5+ per category |
| KS test | Feature/data drift detection | 50+ |
| Bootstrap CI | Small samples, non-normal distributions | 10+ |
| Regression detection (N-sigma) | CI/CD gates, monitoring | 10+ historical runs |

For sample sizes below 30, prefer non-parametric methods (bootstrap CI, Mann-Whitney U) over parametric tests.

### Changes to eval-taxonomy.md

Update the grader selection guide decision tree to add a fourth branch:

```text
Is there a single correct answer or finite set of valid answers?
  → Yes → CODE-BASED grader (string match, binary test, state diff)
  → No  → Does the agent modify external state (files, DB, APIs)?
            → Yes → OUTCOME VERIFICATION grader (state diff, API check, idempotency)
            → No  → Is correctness a distribution property (aggregate metrics, drift, statistical thresholds)?
                      → Yes → STATISTICAL VALIDATION grader (confidence intervals, hypothesis tests)
                      → No  → LLM-AS-JUDGE grader (rubric scoring, pairwise comparison)
```

Add a new section after "Outcome Verification Graders":

```markdown
## Statistical Validation Graders

| Method | Description | Best For |
| -------- | ------------- | ---------- |
| Confidence interval | Metric within [lower, upper] at significance level | Accuracy/F1 threshold validation |
| Two-sample t-test | Compare means of two distributions | A/B testing, regression detection |
| Chi-squared test | Compare categorical distributions | Classification distribution shifts |
| KS test | Compare continuous distributions | Feature/data drift detection |
| Bootstrap CI | Resample-based confidence interval | Small samples, non-normal data |
| Regression detection | Metric drops > N sigma from baseline | CI/CD gates, monitoring |

**Use when**: Correctness is a distribution property — aggregate metrics, drift detection, A/B comparisons, or threshold validation over batches.
**Strengths**: Quantifies uncertainty, handles natural variance, supports CI/CD gating.
**Weaknesses**: Requires sufficient sample sizes, assumptions about distributions, slower feedback loops.
**Min sample sizes**: 30+ for parametric tests, 10+ for bootstrap. Below 30, prefer non-parametric methods.
```

## Revised Phase Structure

1. **Locate the spec**
2. **Detect archetypes** — Scan spec for signature patterns, classify into 1+ archetypes, present to user for confirmation
3. **Extract requirements via archetype checklists** — For each matched archetype, run its extraction checklist. Merge results across archetypes. Deduplicate. If extraction reveals the spec matches additional archetypes, return to Phase 2.
4. **Inventory call paths** — LLM call paths (current) PLUS non-LLM call paths for ml-pipeline and data-workflow archetypes (model inference, API integrations, data transformations)
5. **Classify each requirement** — Now 4 grader categories: code-based, LLM-as-judge, outcome verification, statistical validation
6. **Generate eval tasks** — Same as current, plus archetype-specific task templates
7. **Build balanced problem sets** — Same as current
8. **Select graders and define logic** — Same as current, plus statistical validation graders
9. **Define pass criteria with archetype-specific default metrics** — Each matched archetype contributes its default metrics. Merge and deduplicate. User approves which metrics are tracked vs. hard-gated.
10. **Present eval plan**
11. **Coverage check** — Same as current, plus check that all matched archetypes have coverage. If gaps found, return to Phase 6 to add eval tasks (re-run Phases 7-9 for new tasks only).
12. **Framework compatibility check** — Verify the eval plan is implementable. Add a "Framework Notes" section to any task that needs non-standard capabilities. Common capability gaps: multi-session state (most frameworks are single-session), container sandboxing (Harbor supports natively, others need Docker setup), statistical graders (typically require custom scoring functions wrapping scipy/statsmodels), pairwise comparison (Inspect AI supports natively, others need custom harness). Reference agent-eval-harness, Harbor, and Inspect AI as execution environments.
13. **Write eval plan file**
14. **Transition**

## Eval Plan Template Changes

### Changes to eval-plan-template.md

**1. Add to Summary section:**

```markdown
**Archetypes**: <matched archetypes, e.g., single-agent, rag-pipeline>
**Primary archetype**: <the one leading the eval plan>
```

**2. Add after LLM Call Path Inventory:**

```markdown
## Non-LLM Call Path Inventory (if applicable)

| Path ID | Component | Purpose | Input | Output |
|---------|-----------|---------|-------|--------|
| nlm-001 | ... | model inference / data transformation / API integration / ... | ... | ... |
```

**3. Replace flat Non-Functional Metrics with per-archetype structure:**

```markdown
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
```

**4. Remove standalone Harness Metrics and Memory Metrics sections.** These are now part of single-agent/multi-agent archetype metrics.

**5. Add before Eval Harness Requirements:**

```markdown
## Framework Compatibility Notes

| Task ID | Capability Needed | Framework Support |
|---------|-------------------|-------------------|
| <id> | <e.g., multi-session state> | <e.g., Harbor: native, Inspect AI: custom setup> |
```

Archetype defaults are domain-specific. The existing skill's cross-cutting dimensions change scope:

- **Universal** (apply to all archetypes): cost-efficiency, safety, robustness, governance, observability
- **Agent-scoped** (move into single-agent and multi-agent archetype defaults): memory quality, selective forgetting, cross-session coherence, consolidation fidelity, harness metrics (routing accuracy, context compaction fidelity, fault recovery, premature completion, over-ambition, model transferability), skill/capability invocation

This keeps non-agent eval plans (ml-pipeline, data-workflow, api-service) clean while ensuring agent eval plans still cover these dimensions automatically.

## Reference File Structure

```
define-evals/
├── SKILL.md
└── references/
    ├── archetypes/
    │   ├── single-agent.md
    │   ├── multi-agent.md
    │   ├── rag-pipeline.md
    │   ├── user-facing-product.md
    │   ├── ml-pipeline.md
    │   ├── data-workflow.md
    │   ├── content-generation.md
    │   └── api-service.md
    ├── eval-guide.md              # Updated with archetype cross-references
    ├── eval-taxonomy.md           # Updated with statistical validation
    ├── eval-plan-template.md      # Updated template
    ├── example-eval-plan.md       # Updated example
    └── sources.md                 # Updated with new research
```

Each archetype reference file contains:
1. Signature patterns (how to detect this archetype)
2. Extraction checklist (what to look for in the spec)
3. Default eval dimensions (metrics that always apply)
4. Default grader patterns (which grader types map to which requirement types)
5. Common failure modes (what typically goes wrong)
6. Example eval tasks (2-3 concrete examples)

## Future Work

### review-evals skill
- Triggers after evals have been executed
- Guides result interpretation, failure triage, judge calibration
- Produces review reports with recommended actions
- Methodology-only (no harness-specific code)

### evolve-evals skill
- Triggers when eval suites mature
- Guides graduation (capability → regression), saturation detection, replacement
- Manages baseline comparisons for new components
- Tracks eval health over time

## Sources

All sources from `references/sources.md` remain applicable. The following topics should be researched during implementation and added to `sources.md`:
- Multi-agent evaluation patterns
- Statistical validation methods for ML pipelines
- UX evaluation for AI products
