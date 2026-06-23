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

A spec can match multiple archetypes. Each matched archetype contributes its checklist and metrics, which are merged and deduplicated.

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
- Confidence interval comparison
- Statistical hypothesis testing (t-test, chi-squared)
- Distribution comparison (KS test, Jensen-Shannon divergence)
- Regression detection (metric drops beyond N standard deviations)

## Revised Phase Structure

1. **Locate the spec**
2. **Detect archetypes** — Scan spec for signature patterns, classify into 1+ archetypes, present to user for confirmation
3. **Extract requirements via archetype checklists** — For each matched archetype, run its extraction checklist. Merge results across archetypes. Deduplicate.
4. **Inventory call paths** — LLM call paths (current) PLUS non-LLM call paths for ml-pipeline and data-workflow archetypes (model inference, API integrations, data transformations)
5. **Classify each requirement** — Now 4 grader categories: code-based, LLM-as-judge, outcome verification, statistical validation
6. **Generate eval tasks** — Same as current, plus archetype-specific task templates
7. **Build balanced problem sets** — Same as current
8. **Select graders and define logic** — Same as current, plus statistical validation graders
9. **Define pass criteria with archetype-specific default metrics** — Each matched archetype contributes its default metrics. Merge and deduplicate. User approves which metrics are tracked vs. hard-gated.
10. **Present eval plan**
11. **Coverage check** — Same as current, plus check that all matched archetypes have coverage
12. **Framework compatibility check** — Verify the eval plan is implementable. Reference agent-eval-harness and Harbor as recommended execution environments. Flag if any task requires capabilities not commonly available (multi-session, container isolation, pairwise comparison).
13. **Write eval plan file**
14. **Transition**

## Eval Plan Template Changes

The eval plan template gains:

- **Archetypes matched** field in the summary section
- **Per-archetype metric tables** replacing the flat non-functional metrics list
- **Statistical validation graders** section (for ml-pipeline/data-workflow)
- **Framework compatibility notes** section

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
    │   └── content-generation.md
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

All sources from `references/sources.md` remain applicable. Additional research needed for:
- Multi-agent evaluation patterns (coordination metrics, failure cascade measurement)
- Non-LLM ML evaluation best practices (MLOps evaluation literature)
- UX evaluation for AI products (user journey testing, accessibility compliance)
- Statistical validation methods for ML pipelines
