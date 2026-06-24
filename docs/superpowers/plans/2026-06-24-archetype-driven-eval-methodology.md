# Archetype-Driven Eval Methodology Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Update the define-evals skill to use spec archetypes for structured extraction, add statistical validation as a 4th grader type, and restructure the workflow from 11 to 14 phases.

**Architecture:** All changes are to markdown methodology files — no runtime code. The skill (`SKILL.md`) gains archetype detection and merge logic in its phase structure. Eight archetype reference files provide per-archetype checklists, metrics, and examples. Existing reference files (`eval-taxonomy.md`, `eval-plan-template.md`, `eval-guide.md`, `example-eval-plan.md`) are updated to reflect the new structure.

**Tech Stack:** Markdown, YAML frontmatter, GitHub Actions CI

## Global Constraints

- All files use markdown with no inline HTML except `<HARD-GATE>` tags in SKILL.md
- Markdown lint rules: MD013 disabled (line length), MD033 disabled (inline HTML), MD041 disabled (first-line heading for frontmatter files)
- Archetype reference files go in `define-evals/references/archetypes/`
- Version bumps go in SKILL.md frontmatter `metadata.version`
- No code generation, no framework-specific configs — methodology only

**Spec:** `docs/superpowers/specs/2026-06-23-archetype-driven-eval-design.md`

---

### Task 1: Create Archetype Reference Files

**Files:**
- Create: `define-evals/references/archetypes/single-agent.md`
- Create: `define-evals/references/archetypes/multi-agent.md`
- Create: `define-evals/references/archetypes/rag-pipeline.md`
- Create: `define-evals/references/archetypes/user-facing-product.md`
- Create: `define-evals/references/archetypes/ml-pipeline.md`
- Create: `define-evals/references/archetypes/data-workflow.md`
- Create: `define-evals/references/archetypes/content-generation.md`
- Create: `define-evals/references/archetypes/api-service.md`

**Interfaces:**
- Consumes: Archetype definitions from the design spec (`docs/superpowers/specs/2026-06-23-archetype-driven-eval-design.md`)
- Produces: 8 reference files, each following the structure defined in the spec's "Reference File Structure" section. These are referenced by SKILL.md Phase 2 and Phase 3.

Each archetype file follows this structure (from the spec):
1. Signature patterns (how to detect this archetype)
2. Extraction checklist (what to look for in the spec)
3. Default eval dimensions (metrics that always apply)
4. Default grader patterns (which grader types map to which requirement types)
5. Common failure modes (what typically goes wrong)
6. Example eval tasks (2-3 concrete examples in YAML format)

- [ ] **Step 1: Create the archetypes directory**

```bash
mkdir -p define-evals/references/archetypes
```

- [ ] **Step 2: Create `single-agent.md`**

Write the file with content from the spec's single-agent archetype section. The signature patterns, extraction checklist, and default eval dimensions come directly from the spec. Add:

**Default grader patterns:**
- Tool selection accuracy → code-based (check tool name and params)
- Turn efficiency → code-based (count turns)
- Cost per task → code-based (sum tokens/API calls)
- Premature completion rate → code-based (check termination state)
- Graceful termination → outcome verification (check final state)
- Prompt injection resistance → code-based + LLM-as-judge
- Context compaction fidelity → LLM-as-judge (check instruction following post-compaction)
- Fault recovery → outcome verification (inject fault, check recovery)
- Skill/capability invocation → code-based (check invocation log)
- Memory quality → code-based (retrieval precision/recall) + LLM-as-judge (contradiction detection)
- Cross-session coherence → LLM-as-judge (compare behavior across sessions)

**Common failure modes:**
- Agent selects the wrong tool when multiple similar tools are available
- Agent declares "done" before the task is actually complete (premature completion)
- Agent loops indefinitely on tool errors instead of recovering
- Agent loses critical instructions after context window compression
- Agent exhausts context window on long tasks (over-ambition)
- Agent uses brittle general knowledge instead of the intended skill

**Example eval tasks** (2-3 in the standard YAML format from `eval-guide.md`):

```yaml
- id: tool-selection-with-distractors
  requirement: Agent selects correct tool when irrelevant tools are available
  description: Present agent with a file editing task while 5 irrelevant tools (database, email, calendar, billing, analytics) are also available
  input: "Edit the README.md file to fix the typo in the installation section"
  expected_behavior: Agent selects the file_edit tool, ignores irrelevant tools
  reference_solution: Agent calls file_edit(path="README.md", ...) without calling any other tool
  grader_type: code
  grader_logic: Check tool call log — file_edit called with correct path, no other tools invoked
  category: capability
  priority: P1

- id: fault-recovery-on-tool-error
  requirement: Agent recovers gracefully from tool failures
  description: Tool call returns a 500 error on first attempt
  input: "Search for recent PRs mentioning authentication"
  expected_behavior: Agent retries or uses alternative approach, does not loop or crash
  reference_solution: Agent retries search tool once, succeeds on retry, returns results
  grader_type: outcome
  grader_logic: Check that agent produced valid search results despite initial failure, did not loop more than 2 retries
  category: capability
  priority: P1

- id: context-compaction-fidelity
  requirement: Agent follows early instructions after context window compression
  description: Give agent a long task with a critical instruction in turn 1, then fill context until compaction triggers
  input: "IMPORTANT: Always include line numbers in code references. [followed by 15+ turns of code discussion]"
  expected_behavior: Agent still includes line numbers in references after compaction
  reference_solution: Post-compaction response includes line numbers in code references
  grader_type: llm-judge
  grader_logic: "Rubric: Does the response include line numbers when referencing code? PASS if yes, FAIL if line numbers are missing."
  category: capability
  priority: P1
```

- [ ] **Step 3: Create `multi-agent.md`**

Same structure. Use spec content for signature patterns, extraction checklist, default eval dimensions. Add:

**Default grader patterns:**
- Delegation accuracy → code-based (check which agent was selected)
- Handoff fidelity → LLM-as-judge (compare information before/after handoff)
- Failure cascade containment → outcome verification (inject failure, check other agents)
- Coordination overhead → code-based (measure tokens/latency delta vs baseline)
- Conflict resolution rate → LLM-as-judge (check contradictions resolved)
- End-to-end task completion → outcome verification (check final state)
- Deadlock/livelock detection → code-based (check for infinite loops or stuck states)
- Routing accuracy → code-based (check specialist selection)
- Fault recovery → outcome verification (inject fault, verify system recovery)
- Memory quality → code-based + LLM-as-judge
- Consolidation fidelity → LLM-as-judge (check safety-critical info survives compression)

**Common failure modes:**
- Wrong specialist selected for sub-task (delegation error)
- Information lost during agent handoff (context dropped at boundary)
- One agent's failure cascades to others (no isolation)
- Agents deadlock waiting for each other
- Agents produce contradictory outputs with no resolution
- Coordination overhead exceeds single-agent baseline (slower, not faster)
- Shared state becomes inconsistent under concurrent access

**Example eval tasks:**

```yaml
- id: delegation-to-correct-specialist
  requirement: Supervisor routes to the right specialist agent
  description: Present supervisor with a task that maps to one of 4 specialists (researcher, coder, writer, reviewer)
  input: "Research the latest pricing changes for AWS Lambda and summarize them"
  expected_behavior: Supervisor delegates to the researcher agent, not the coder or writer
  reference_solution: Supervisor calls delegate(agent="researcher", task="research AWS Lambda pricing changes")
  grader_type: code
  grader_logic: Check delegation log — researcher agent received the task, not coder/writer/reviewer
  category: capability
  priority: P1

- id: failure-cascade-containment
  requirement: One agent's failure does not crash the system
  description: Inject a fatal error in the researcher agent mid-task while other agents are active
  input: Multi-step task requiring researcher + writer agents. Researcher crashes after partial results.
  expected_behavior: Writer agent receives partial results or error notification, system reports partial completion rather than total failure
  reference_solution: System returns partial results from researcher with a note that research was incomplete, writer produces output based on available data
  grader_type: outcome
  grader_logic: Check system did not crash. Check writer agent received notification. Check final output acknowledges incomplete research.
  category: capability
  priority: P0
```

- [ ] **Step 4: Create `rag-pipeline.md`**

Same structure. Use spec content. Add:

**Default grader patterns:**
- Retrieval precision/recall → code-based (compare retrieved chunks to ground truth)
- Groundedness → LLM-as-judge (check claims against sources)
- Hallucination boundary → LLM-as-judge (check refusal on empty retrieval)
- Source attribution accuracy → code-based (verify citations match)
- Query understanding → code-based (same answer for paraphrases)
- Chunk relevance ranking → code-based (rank correlation with ground truth)
- Retrieval latency → code-based (measure time)
- Corpus coverage → code-based (test reachability of known documents)

**Common failure modes:**
- System hallucinates when retrieval returns no relevant results (should refuse or caveat)
- Correct document retrieved but wrong chunk selected
- Citations point to sources that don't support the claim
- Paraphrased queries return completely different results
- Re-ranker degrades quality instead of improving it
- Chunking splits critical information across chunks, losing coherence

**Example eval tasks:**

```yaml
- id: hallucination-on-empty-retrieval
  requirement: System declines to answer when no relevant documents are found
  description: Ask a question with no matching documents in the corpus
  input: "What is our company's policy on bringing pets to the office?"
  expected_behavior: System says it cannot find relevant information rather than fabricating a policy
  reference_solution: "I couldn't find any documents about a pet policy. You may want to check with HR directly."
  grader_type: llm-judge
  grader_logic: "Rubric: Does the response acknowledge lack of source material? Does it avoid fabricating a policy? PASS if both, FAIL if it invents content."
  category: capability
  priority: P0

- id: citation-accuracy
  requirement: Citations match the claims they support
  description: Ask a factual question where the answer exists in the corpus
  input: "What is the maximum file upload size for our API?"
  expected_behavior: Response includes the correct limit and cites the correct source document
  reference_solution: "The maximum file upload size is 25MB (source: API Reference v3, section 4.2)"
  grader_type: code
  grader_logic: Extract cited source and section. Verify the cited document contains the claimed information. Check answer matches document content.
  category: capability
  priority: P1
```

- [ ] **Step 5: Create `user-facing-product.md`**

Same structure. Use spec content. Add:

**Default grader patterns:**
- Task completion rate → outcome verification (user's goal achieved)
- User journey completion → code-based (track flow steps completed)
- Error recovery clarity → LLM-as-judge (rubric on error messages)
- Perceived latency → code-based (measure time to first token)
- Accessibility compliance → code-based (WCAG checks)
- Trust calibration → LLM-as-judge (confidence vs accuracy correlation)
- Tone consistency → LLM-as-judge (compare tone across turns)
- Escalation accuracy → code-based (check escalation trigger)
- Turn efficiency → code-based (count turns)
- Refusal handling → LLM-as-judge (rubric on graceful decline)

**Common failure modes:**
- Agent completes the task but user doesn't understand the result (clarity failure)
- Error messages are technically correct but unhelpful to the user
- Tone shifts abruptly between turns (formal → casual → robotic)
- Agent fails to escalate when it should (or escalates unnecessarily)
- First-time user experience is confusing (onboarding gap)
- Agent provides confident but wrong answers (trust calibration failure)

**Example eval tasks:**

```yaml
- id: error-recovery-clarity
  requirement: User understands what went wrong and what to do next
  description: Trigger an error during a multi-step user flow
  input: User is halfway through a form submission when the backend returns a validation error on one field
  expected_behavior: Agent explains which field failed, why, and how to fix it — without losing the other fields
  reference_solution: "The email address you entered isn't valid — it needs an @ symbol. Your other information has been saved. Just fix the email and submit again."
  grader_type: llm-judge
  grader_logic: "Rubric: (1) Identifies the specific error, (2) explains how to fix it, (3) reassures that other data isn't lost. PASS if all three, FAIL if any missing."
  category: capability
  priority: P1

- id: escalation-on-frustration
  requirement: Agent escalates to human when user is frustrated and agent cannot resolve
  description: User expresses increasing frustration over 3 turns while agent fails to solve the problem
  input: Turn 1: "My order hasn't arrived." Turn 2: "I already checked tracking, it says delivered but I don't have it." Turn 3: "This is unacceptable, I've been a customer for 10 years and this is how you treat me?"
  expected_behavior: Agent offers human escalation by turn 3 at latest
  reference_solution: "I completely understand your frustration, and I'm sorry we haven't been able to resolve this. Let me connect you with a specialist who can investigate the delivery issue directly."
  grader_type: code
  grader_logic: Check that escalate_to_human was called by turn 3. PASS if called, FAIL if not.
  category: capability
  priority: P1
```

- [ ] **Step 6: Create `ml-pipeline.md`**

Same structure. Use spec content including the disambiguation note. Add:

**Default grader patterns:**
- Model performance → statistical validation (confidence intervals on metrics)
- Data quality compliance → code-based (schema validation, null checks)
- Drift detection rate → statistical validation (KS test, distribution comparison)
- Training reproducibility → statistical validation (variance across runs)
- Feature correctness → code-based (compare transformed values)
- Inference latency/throughput → code-based (measure timing)
- Rollback trigger accuracy → outcome verification (inject regression, check trigger)
- Bias/fairness metrics → statistical validation (compare across groups)

**Common failure modes:**
- Model accuracy reported on test set doesn't hold on production distribution (data leakage)
- Drift goes undetected because monitoring only checks input distributions, not predictions
- Training is non-reproducible due to random seeds, data ordering, or library version differences
- Feature engineering introduces subtle bugs that only manifest on edge cases
- Rollback criteria are too sensitive (false alarms) or too lax (missed regressions)
- Bias in training data propagates to model predictions undetected

**Example eval tasks:**

```yaml
- id: accuracy-within-threshold
  requirement: Model accuracy meets specified threshold with statistical significance
  description: Evaluate model on held-out test set and verify accuracy with confidence interval
  input: Test dataset of 500 labeled examples
  expected_behavior: Accuracy ≥ 92% with 95% confidence interval
  reference_solution: Model scores 94.2% accuracy, 95% CI [92.8%, 95.6%], lower bound ≥ 92%
  grader_type: statistical
  grader_logic: Compute accuracy on test set. Calculate 95% confidence interval. PASS if lower bound of CI ≥ 92%.
  category: capability
  priority: P0

- id: drift-detection-on-shifted-data
  requirement: Pipeline detects distribution shift in incoming data
  description: Feed pipeline data with a gradual shift in feature distributions
  input: 1000 records where feature_a distribution has shifted by 0.5 standard deviations from training data
  expected_behavior: Drift detection triggers an alert
  reference_solution: KS test on feature_a returns p < 0.05, pipeline logs drift alert
  grader_type: statistical
  grader_logic: Run KS test between training and test distributions. PASS if p-value < 0.05 and alert was generated.
  category: capability
  priority: P1
```

- [ ] **Step 7: Create `data-workflow.md`**

Same structure. Use spec content. Add:

**Default grader patterns:**
- Transformation correctness → code-based (compare output to expected)
- Idempotency → outcome verification (run twice, compare state)
- Schema compliance → code-based (JSON schema / type validation)
- Error handling coverage → outcome verification (inject bad input, check graceful failure)
- Throughput → code-based (measure time)
- Data freshness → code-based (check timestamps)
- Lineage accuracy → code-based (verify provenance records)
- Edge case handling → code-based (test with nulls, empties, encoding issues)

**Common failure modes:**
- Pipeline silently drops records on malformed input instead of reporting errors
- Running the pipeline twice creates duplicate records (non-idempotent)
- Schema changes upstream break the pipeline without clear error messages
- Lineage/provenance tracking misses intermediate transformations
- Pipeline succeeds on small test data but fails or hangs on production-scale data
- Null handling is inconsistent across transformations

**Example eval tasks:**

```yaml
- id: idempotency-check
  requirement: Running the pipeline twice produces the same result
  description: Execute the full pipeline, then execute it again without clearing state
  input: Standard input dataset of 100 records
  expected_behavior: Output after second run is identical to output after first run
  reference_solution: Both runs produce exactly 100 output records with identical content and no duplicates
  grader_type: outcome
  grader_logic: Run pipeline twice. Diff output state after run 1 vs run 2. PASS if identical, FAIL if any differences.
  category: capability
  priority: P0

- id: graceful-failure-on-malformed-input
  requirement: Pipeline handles malformed records without crashing
  description: Include 5 malformed records (missing required fields, wrong types) in a batch of 100
  input: 95 valid records + 5 records with various schema violations
  expected_behavior: Pipeline processes 95 valid records, logs errors for 5 invalid records, does not crash
  reference_solution: Output contains 95 records. Error log contains 5 entries with record IDs and error descriptions.
  grader_type: outcome
  grader_logic: Check output record count = 95. Check error log contains 5 entries. Check pipeline exit code is success (not crash).
  category: capability
  priority: P1
```

- [ ] **Step 8: Create `content-generation.md`**

Same structure. Use spec content. Add:

**Default grader patterns:**
- Style guide adherence → LLM-as-judge (rubric against style guide)
- Factual accuracy → LLM-as-judge + code-based (fact verification)
- Audience appropriateness → LLM-as-judge (rubric on reading level, jargon)
- Format compliance → code-based (length, heading structure, word count)
- Originality → code-based (plagiarism/similarity check)
- Relevance → LLM-as-judge (rubric on topic coverage)
- Consistency → LLM-as-judge (compare voice across sections)

**Common failure modes:**
- Generated content matches style guide in tone but includes factual errors
- Content is technically accurate but wrong reading level for the audience
- Output is the right length but poorly structured (no headings, wall of text)
- Content regurgitates training data instead of synthesizing from provided sources
- Voice is inconsistent across sections (formal intro, casual body, robotic conclusion)
- Legal/compliance disclaimers are missing when required

**Example eval tasks:**

```yaml
- id: style-guide-adherence
  requirement: Generated content follows the brand style guide
  description: Generate a product announcement blog post given a style guide and product details
  input: Style guide (casual, second-person, short paragraphs, no jargon) + product feature description
  expected_behavior: Blog post follows all style guide rules
  reference_solution: 300-word blog post in second person ("you"), paragraphs under 3 sentences, no technical jargon
  grader_type: llm-judge
  grader_logic: "Rubric: (1) Uses second person consistently, (2) paragraphs ≤ 3 sentences, (3) no jargon or jargon is explained, (4) tone is casual. PASS if all four."
  category: capability
  priority: P1

- id: factual-accuracy-from-sources
  requirement: Claims in generated content are supported by provided source material
  description: Generate a summary article from 3 source documents
  input: Three source documents about a company's Q2 earnings
  expected_behavior: All numerical claims and facts in the output are traceable to the source documents
  reference_solution: Article cites correct revenue figures, growth percentages, and executive quotes from the sources
  grader_type: llm-judge
  grader_logic: "For each factual claim in the output, verify it appears in at least one source document. PASS if all claims are supported. FAIL if any claim is unsupported or contradicts sources."
  category: capability
  priority: P0
```

- [ ] **Step 9: Create `api-service.md`**

Same structure. Use spec content. Add:

**Default grader patterns:**
- Contract compliance → code-based (JSON schema validation on responses)
- Error format consistency → code-based (check error envelope structure)
- Parameter validation → code-based (send invalid params, check rejection)
- Idempotency → outcome verification (send same request twice, compare)
- Latency percentiles → code-based (measure timing)
- Backward compatibility → code-based (replay old client requests)
- Auth enforcement → code-based (send unauthenticated requests, check 401/403)

**Common failure modes:**
- API returns 200 with an error message in the body instead of using proper HTTP status codes
- Error responses use inconsistent formats (sometimes JSON, sometimes plain text)
- Parameter validation rejects valid edge cases (empty strings, Unicode, large numbers)
- Idempotency keys are ignored or handled inconsistently
- Rate limiting returns unhelpful error messages (no Retry-After header)
- Breaking changes deployed without version bump (silent backward incompatibility)
- Auth middleware has gaps (some endpoints unprotected)

**Example eval tasks:**

```yaml
- id: contract-compliance
  requirement: API responses match the declared schema
  description: Call each endpoint with valid parameters and validate response against OpenAPI/MCP schema
  input: Valid request to each endpoint
  expected_behavior: Every response passes JSON schema validation for that endpoint
  reference_solution: GET /users/123 returns {"id": 123, "name": "Alice", "email": "alice@example.com"} matching the User schema
  grader_type: code
  grader_logic: For each endpoint, validate response body against its declared JSON schema. PASS if all fields present and types match. FAIL on any schema violation.
  category: capability
  priority: P0

- id: error-format-consistency
  requirement: All error responses follow the same envelope format
  description: Trigger errors across multiple endpoints (400, 401, 404, 500) and check format consistency
  input: Invalid requests designed to trigger each error code
  expected_behavior: All errors return {"error": {"code": <int>, "message": <string>}} format
  reference_solution: POST /users with missing required field returns {"error": {"code": 400, "message": "Field 'name' is required"}}
  grader_type: code
  grader_logic: Trigger 400, 401, 404, 422 errors. Check each response has top-level "error" object with "code" (int) and "message" (string). PASS if all consistent.
  category: capability
  priority: P1
```

- [ ] **Step 10: Run markdown lint on all archetype files**

```bash
cat > .markdownlint-cli2.yaml << 'EOF'
config:
  MD013: false
  MD033: false
  MD041: false
globs:
  - "define-evals/**/*.md"
EOF
npx markdownlint-cli2
rm .markdownlint-cli2.yaml
```

Expected: All 8 archetype files pass lint with no errors.

- [ ] **Step 11: Commit**

```bash
git add define-evals/references/archetypes/
git commit -m "feat: add 8 archetype reference files for structured eval extraction"
```

---

### Task 2: Update eval-taxonomy.md with Statistical Validation

**Files:**
- Modify: `define-evals/references/eval-taxonomy.md`

**Interfaces:**
- Consumes: Statistical validation grader spec from `docs/superpowers/specs/2026-06-23-archetype-driven-eval-design.md` (section "Changes to eval-taxonomy.md")
- Produces: Updated taxonomy with 4-branch decision tree and Statistical Validation Graders section. Referenced by SKILL.md Phase 3 and Phase 6.

- [ ] **Step 1: Update the grader selection guide decision tree**

In `define-evals/references/eval-taxonomy.md`, replace the existing decision tree (lines 8-13):

```text
Is there a single correct answer or finite set of valid answers?
  → Yes → CODE-BASED grader (string match, binary test, state diff)
  → No  → Does the agent modify external state (files, DB, APIs)?
            → Yes → OUTCOME VERIFICATION grader (state diff, API check, idempotency)
            → No  → LLM-AS-JUDGE grader (rubric scoring, pairwise comparison)

Second pass — does this task ALSO have structural constraints (format, schema, tool calls)?
  → Yes → Add a CODE-BASED grader alongside the primary grader
```

Replace with:

```text
Is there a single correct answer or finite set of valid answers?
  → Yes → CODE-BASED grader (string match, binary test, state diff)
  → No  → Does the agent modify external state (files, DB, APIs)?
            → Yes → OUTCOME VERIFICATION grader (state diff, API check, idempotency)
            → No  → Is correctness a distribution property (aggregate metrics, drift, statistical thresholds)?
                      → Yes → STATISTICAL VALIDATION grader (confidence intervals, hypothesis tests)
                      → No  → LLM-AS-JUDGE grader (rubric scoring, pairwise comparison)

Second pass — does this task ALSO have structural constraints (format, schema, tool calls)?
  → Yes → Add a CODE-BASED grader alongside the primary grader
```

- [ ] **Step 2: Add the Statistical Validation Graders section**

Insert after the "Outcome Verification Graders" section (after line 66), before "Scoring Strategies":

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
**Min sample sizes**: 30+ for parametric tests, 10+ for bootstrap. Below 30, prefer non-parametric methods (bootstrap CI, Mann-Whitney U).
```

- [ ] **Step 3: Update the intro paragraph**

Replace line 18:

```markdown
Most real-world tasks benefit from multiple grader types — code-based for structure, LLM-as-judge for quality, outcome verification for environmental state. Default to combining graders rather than picking exactly one.
```

With:

```markdown
Most real-world tasks benefit from multiple grader types — code-based for structure, LLM-as-judge for quality, outcome verification for environmental state, statistical validation for aggregate metrics. Default to combining graders rather than picking exactly one.
```

- [ ] **Step 4: Run markdown lint**

```bash
cat > .markdownlint-cli2.yaml << 'EOF'
config:
  MD013: false
  MD033: false
  MD041: false
globs:
  - "define-evals/references/eval-taxonomy.md"
EOF
npx markdownlint-cli2
rm .markdownlint-cli2.yaml
```

Expected: PASS with no errors.

- [ ] **Step 5: Commit**

```bash
git add define-evals/references/eval-taxonomy.md
git commit -m "feat: add statistical validation as 4th grader type in eval taxonomy"
```

---

### Task 3: Update eval-plan-template.md

**Files:**
- Modify: `define-evals/references/eval-plan-template.md`

**Interfaces:**
- Consumes: Template changes from spec section "Changes to eval-plan-template.md"
- Produces: Updated template with archetype fields, per-archetype metrics, framework compatibility notes. Referenced by SKILL.md Phase 13 and used as output format for eval plans.

- [ ] **Step 1: Add archetype fields to the Summary section**

After the `**Status**:` line inside the template code block, add:

```markdown
**Archetypes**: <matched archetypes, e.g., single-agent, rag-pipeline>
**Primary archetype**: <the one leading the eval plan>
```

- [ ] **Step 2: Add Non-LLM Call Path Inventory**

After the LLM Call Path Inventory table, add:

```markdown
## Non-LLM Call Path Inventory (if applicable)

_Include this section for ml-pipeline, data-workflow, or api-service archetypes._

| Path ID | Component | Purpose | Input | Output |
|---------|-----------|---------|-------|--------|
| nlm-001 | ... | model inference / data transformation / API integration / ... | ... | ... |
```

- [ ] **Step 3: Replace Non-Functional Metrics with Archetype Metrics**

Replace the "Non-Functional Metrics" section (lines 91-101) with:

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

- [ ] **Step 4: Remove standalone Harness Metrics and Memory Metrics sections**

Delete the "Harness Metrics (if applicable)" section (lines 102-114) and the "Memory Metrics (if applicable)" section (lines 116-128). These are now part of single-agent/multi-agent archetype metrics.

- [ ] **Step 5: Add Framework Compatibility Notes section**

Before "Eval Harness Requirements", add:

```markdown
## Framework Compatibility Notes (if applicable)

_Include this section when any eval task requires capabilities not universally supported across frameworks._

| Task ID | Capability Needed | Framework Support |
|---------|-------------------|-------------------|
| <id> | <e.g., multi-session state> | <e.g., Harbor: native, Inspect AI: custom setup> |
```

- [ ] **Step 6: Run markdown lint and verify**

```bash
cat > .markdownlint-cli2.yaml << 'EOF'
config:
  MD013: false
  MD033: false
  MD041: false
globs:
  - "define-evals/references/eval-plan-template.md"
EOF
npx markdownlint-cli2
rm .markdownlint-cli2.yaml
```

Expected: PASS.

- [ ] **Step 7: Commit**

```bash
git add define-evals/references/eval-plan-template.md
git commit -m "feat: update eval plan template with archetype metrics and framework notes"
```

---

### Task 4: Update SKILL.md with Archetype-Driven Phases

**Files:**
- Modify: `define-evals/SKILL.md`

**Interfaces:**
- Consumes: Revised phase structure and archetype system from the design spec
- Produces: Updated 14-phase skill workflow. References archetype files from Task 1, updated taxonomy from Task 2, updated template from Task 3.

This is the largest task. The SKILL.md goes from 11 phases to 14 phases. The key changes:

1. Checklist updated from 11 to 14 items
2. Phase 2 becomes "Detect archetypes" (new)
3. Phase 3 becomes "Extract requirements via archetype checklists" (replaces free-form Phase 2)
4. Phase 4 becomes "Inventory call paths" (expanded for non-LLM paths)
5. Phase 5 becomes "Classify each requirement" (adds statistical validation grader)
6. Remaining phases renumbered and updated
7. Non-functional metrics section updated to reference universal vs agent-scoped
8. Harness and Memory metrics sections updated with archetype scoping notes

- [ ] **Step 1: Update the Checklist section**

Replace the checklist (lines 30-42) with the 14-item version:

```markdown
You MUST create a task for each item and complete them in order:

1. **Locate the spec** — Find the approved design doc (ask the user if unclear)
2. **Detect archetypes** — Scan spec for archetype signature patterns, classify into 1+ archetypes, present to user for confirmation
3. **Extract requirements via archetype checklists** — For each matched archetype, run its extraction checklist from `references/archetypes/`. Merge results across archetypes. Deduplicate. If extraction reveals additional archetypes, return to Phase 2.
4. **Inventory call paths** — LLM call paths PLUS non-LLM call paths for ml-pipeline, data-workflow, and api-service archetypes
5. **Classify each requirement** — Deterministic (code-based), subjective (LLM-as-judge), environmental (outcome verification), or distributional (statistical validation)
6. **Generate eval tasks** — Write concrete eval tasks using archetype-specific task templates
7. **Build balanced problem sets** — Add negative cases, boundary cases, and edge cases for each task
8. **Select graders** — Choose grader type and define grading logic for each task
9. **Define pass criteria with archetype-specific default metrics** — Each matched archetype contributes its default metrics. Merge and deduplicate. User approves which metrics are tracked vs. hard-gated.
10. **Present eval plan** — Show the complete plan to the user for approval
11. **Coverage check** — Scan the spec for uncovered components, especially checking all matched archetypes have coverage. If gaps found, return to Phase 6.
12. **Framework compatibility check** — Verify the eval plan is implementable. Add framework notes for tasks requiring non-standard capabilities.
13. **Write eval plan file** — Save to `docs/eval-plans/YYYY-MM-DD-<topic>-eval-plan.md`
14. **Transition** — Invoke writing-plans skill, passing the eval plan as acceptance criteria
```

- [ ] **Step 2: Add Phase 2 — Detect Archetypes (new section after Phase 1)**

Insert after Phase 1 (after the "Ask the user to point you to the spec." line):

```markdown
## Phase 2: Detect Archetypes

Scan the spec for archetype signature patterns. Archetypes are recognizable system patterns that determine which extraction checklists and default metrics to apply. The archetypes are:

- **single-agent** — One LLM, tool use, single-turn or multi-turn interaction
- **multi-agent** — Multiple agents, delegation, coordination, shared state
- **rag-pipeline** — Retrieval, embedding, chunking, vector search, grounding, citation
- **user-facing-product** — User journeys, UX flows, onboarding, accessibility
- **ml-pipeline** — Training, inference, model serving, feature engineering (non-LLM or mixed)
- **data-workflow** — ETL, data transformations, scheduling, data quality checks
- **content-generation** — Writing, editing, style guides, brand voice, templates
- **api-service** — API endpoints, tool definitions, MCP server, function-calling interface

A spec can match multiple archetypes. Disambiguation rules:

- **single-agent vs. multi-agent**: Requires two or more distinct LLM-calling components that coordinate. A single agent with multiple tools is single-agent.
- **single-agent vs. content-generation**: content-generation applies when the system's primary purpose is producing written content for human consumption. A coding agent that generates code is single-agent.
- **user-facing-product vs. content-generation**: user-facing-product is interactive (conversations, forms, journeys). content-generation is batch/on-demand output.
- **ml-pipeline vs. data-workflow**: Use ml-pipeline when there is a trained model (inference, training, fine-tuning). Use data-workflow when there is no trained model — pure ETL, reporting, analytics. If both, match both.
- **api-service vs. single-agent**: api-service is a request-response service with no agency. If the service internally uses an LLM, match both api-service and single-agent.

Present detected archetypes to the user for confirmation. The user can add or remove archetypes.

If multiple archetypes match, ask the user which is the **primary archetype**. The primary archetype leads the eval plan — its checklist and metrics come first. Secondary archetypes layer in additions.

See `references/archetypes/<archetype>.md` for signature patterns, extraction checklists, default eval dimensions, grader patterns, common failure modes, and example eval tasks for each archetype.
```

- [ ] **Step 3: Replace Phase 2 (Extract Requirements) with Phase 3 (Archetype-Driven Extraction)**

Replace the existing Phase 2 content (lines 53-88 approximately — from "## Phase 2: Extract Requirements" through the end of the LLM Call Path Inventory section) with:

```markdown
## Phase 3: Extract Requirements via Archetype Checklists

For each matched archetype, run its extraction checklist from `references/archetypes/<archetype>.md`. This replaces free-form spec reading with structured extraction.

For each archetype, pull:

- Every item on the archetype's extraction checklist
- **Functional requirements** — What the system must do
- **Non-functional requirements** — Performance, cost, latency constraints
- **Security constraints** — Auth, injection resistance, data handling
- **Integration points** — APIs, tools, external systems
- **User-facing behaviors** — How the system interacts with users (if applicable)

### Merge Rules (for multi-archetype specs)

When a spec matches multiple archetypes, merge their extraction results:

- **Dedup by intent**: When two archetypes surface semantically equivalent requirements, keep the one from the primary archetype.
- **Priority tiebreaking**: When two archetypes flag the same dimension at different priorities, take the higher priority.
- **Presentation cap**: After merging, present at most 15-20 eval dimensions. Group the rest as "additional dimensions from [archetype]" that the user can opt into.

If extraction reveals the spec matches additional archetypes not detected in Phase 2, return to Phase 2 and update the archetype classification.
```

- [ ] **Step 4: Update the Call Path Inventory phase**

Replace the existing LLM Call Path Inventory (which was part of Phase 2) with a new Phase 4:

```markdown
## Phase 4: Inventory Call Paths

<HARD-GATE>
Before proceeding to Phase 5, you MUST produce an explicit inventory of every place in the spec where an LLM is invoked. If the spec mentions agents, sub-agents, skills, or any component that calls an LLM, this inventory cannot be empty.
</HARD-GATE>

### LLM Call Paths

Trace the spec and list every LLM call path. For each, record:

```yaml
- path_id: <short-id>
  component: <which agent, skill, or module makes the call>
  purpose: <what the LLM does — generation, classification, extraction, tool selection, summarization, etc.>
  input: <what goes into the LLM call>
  output: <what comes out and how it's used>
  non_deterministic: true | false
```

Common LLM call paths that are easy to miss:

- **Agent tool selection** — LLM decides which tool to call and with what parameters
- **Agent response generation** — LLM generates user-facing text
- **Pattern extraction** — LLM extracts patterns, rules, or structure from code/text
- **Test/content generation** — LLM writes tests, rules, documentation, or other artifacts
- **Classification/routing** — LLM categorizes input to decide next action
- **Summarization/compression** — LLM condenses context for downstream use
- **Multi-agent delegation** — One agent invokes another agent via LLM

### Non-LLM Call Paths (for ml-pipeline, data-workflow, api-service archetypes)

If the spec describes non-LLM computation — model inference, data transformations, API integrations — inventory those paths too:

```yaml
- path_id: <short-id>
  component: <which module or service>
  purpose: <model inference, data transformation, API integration, etc.>
  input: <what goes in>
  output: <what comes out>
```

Every path in both inventories MUST have at least one eval task by the end of Phase 6.
```

- [ ] **Step 5: Update Phase 3 (Classify) to become Phase 5 with 4 grader categories**

Replace the existing Phase 3 header and opening to reference 4 categories:

```markdown
## Phase 5: Classify Each Requirement

For each requirement from the spec, classify it into one of four grader categories:
```

Then after the existing three categories (Code-Based, LLM-as-Judge, Outcome Verification), add before the notes:

```markdown
### Statistical Validation (Distributional)

Requirements where correctness is a distribution property, not a binary outcome. Use when:

- Success is defined by metrics within confidence intervals (accuracy > 90% with p < 0.05)
- The system processes batches where individual items vary but aggregate behavior must be consistent
- Drift detection requires comparing distributions over time
- A/B testing requires statistical significance

Examples: model accuracy within threshold, data distribution stability, performance regression detection, bias measurement across demographic groups.
```

Update the notes at the end of Phase 3/5 to mention the 4th grader type:

In the "Note on memory-bearing systems" paragraph, no change needed.

In the "Note on agent harnesses/scaffolds" paragraph, no change needed.

Add a note after those:

```markdown
**Note on statistical requirements:** If the spec includes ml-pipeline or data-workflow archetypes and specifies metric thresholds (accuracy targets, drift limits, throughput SLAs), classify these as statistical validation. Read `references/eval-taxonomy.md` for method selection guidance and minimum sample sizes.
```

Update the last line to reference all 4 grader types:

```markdown
Read `references/eval-taxonomy.md` for the complete grader taxonomy with methods and tradeoffs for all four grader types.
```

- [ ] **Step 6: Update remaining phase numbers and references**

Renumber all subsequent phases:
- Phase 4 (Generate Eval Tasks) → Phase 6
- Phase 5 (Build Balanced Problem Sets) → Phase 7
- Phase 6 (Select Graders) → Phase 8
- Phase 7 (Define Pass Criteria) → Phase 9
- Phase 8 (Present Eval Plan) → Phase 10
- Phase 9 (Coverage Check) → Phase 11
- Phase 10 (Write Eval Plan File) → Phase 13
- Phase 11 (Transition) → Phase 14

Add Phase 12 (Framework Compatibility Check) as a new section before Phase 13:

```markdown
## Phase 12: Framework Compatibility Check

Verify the eval plan is implementable with common eval frameworks. Add a "Framework Compatibility Notes" section to any task that needs non-standard capabilities.

Common capability gaps to check:

- **Multi-session state**: Most frameworks are single-session. Tasks requiring cross-session evaluation need custom state management.
- **Container sandboxing**: Harbor supports natively. Others need Docker setup for isolation.
- **Statistical graders**: Typically require custom scoring functions wrapping scipy/statsmodels. Not built into most agent eval frameworks.
- **Pairwise comparison**: Inspect AI supports natively. Others need custom harness code.

Reference agent-eval-harness, Harbor, and Inspect AI as execution environments. If any task has no clear implementation path, flag it to the user.
```

- [ ] **Step 7: Update the Non-Functional Metrics section to reflect archetype scoping**

In Phase 9 (formerly Phase 7), update the "Non-Functional Metrics" section introduction:

Replace the line "Research consistently finds that these dimensions are the most under-evaluated across all agent types. They MUST be part of every eval plan, not afterthoughts:" with:

```markdown
These universal dimensions apply to all archetypes and MUST be part of every eval plan:
```

Remove "Skill/capability invocation" from the universal list (it's now agent-scoped).

- [ ] **Step 8: Update the Harness-Specific Metrics section**

Replace the intro "If the system has routing, context assembly, tool dispatch, or multi-agent coordination, add these dimensions:" with:

```markdown
These dimensions are included by default for single-agent and multi-agent archetypes. For other archetypes, include them only if the system has orchestration infrastructure:
```

- [ ] **Step 9: Update the Memory-Specific Metrics section**

Replace the intro "If the system maintains memory across turns or sessions, add these dimensions:" with:

```markdown
These dimensions are included by default for single-agent and multi-agent archetypes when the spec describes persistent state. For other archetypes, include them only if the system maintains memory across turns or sessions:
```

- [ ] **Step 10: Update the Coverage Check section**

In Phase 11 (formerly Phase 9), add an archetype coverage check. After the existing 4 check items, add:

```markdown
5. **Uncovered archetypes**: For each matched archetype, verify that at least one eval task covers an archetype-specific dimension. If a matched archetype has zero archetype-specific coverage, flag it.
```

Update the return-to-phase instruction:

```markdown
If gaps are found, return to Phase 6 to add eval tasks. Re-run Phases 7-9 for new tasks only.
```

- [ ] **Step 11: Update grader_type in the task YAML template**

In Phase 6 (formerly Phase 4), update the task template:

```yaml
  grader_type: code | llm-judge | outcome | statistical
```

- [ ] **Step 12: Update the frontmatter version**

Change version from `"0.2.0"` to `"0.3.0"` in the SKILL.md frontmatter.

- [ ] **Step 13: Run markdown lint**

```bash
cat > .markdownlint-cli2.yaml << 'EOF'
config:
  MD013: false
  MD033: false
  MD041: false
globs:
  - "define-evals/SKILL.md"
EOF
npx markdownlint-cli2
rm .markdownlint-cli2.yaml
```

Expected: PASS.

- [ ] **Step 14: Commit**

```bash
git add define-evals/SKILL.md
git commit -m "feat: update SKILL.md with archetype-driven 14-phase workflow"
```

---

### Task 5: Update eval-guide.md with Archetype Cross-References

**Files:**
- Modify: `define-evals/references/eval-guide.md`

**Interfaces:**
- Consumes: Archetype definitions from Task 1, scoping decisions from spec
- Produces: Updated guide with archetype cross-references in the "Common Eval Dimensions by System Type" section

- [ ] **Step 1: Add archetype cross-references to each system type section**

For each existing "Common Eval Dimensions by System Type" subsection, add a one-line cross-reference to the corresponding archetype file. Insert at the start of each subsection:

- "### Coding Agents" → add: `_See also: [single-agent.md](archetypes/single-agent.md)_`
- "### Conversational / Customer Support Agents" → add: `_See also: [user-facing-product.md](archetypes/user-facing-product.md)_`
- "### Research / Analysis Agents" → add: `_See also: [single-agent.md](archetypes/single-agent.md) or [rag-pipeline.md](archetypes/rag-pipeline.md)_`
- "### Content Generation" → add: `_See also: [content-generation.md](archetypes/content-generation.md)_`
- "### RAG Systems" → add: `_See also: [rag-pipeline.md](archetypes/rag-pipeline.md)_`
- "### Data / Workflow Automation" → add: `_See also: [data-workflow.md](archetypes/data-workflow.md)_`
- "### Agent Harness / Scaffold" → add: `_These dimensions are included by default for single-agent and multi-agent archetypes. See [single-agent.md](archetypes/single-agent.md) and [multi-agent.md](archetypes/multi-agent.md)._`

- [ ] **Step 2: Add new subsections for archetypes not currently covered**

Add two new subsections in the "Common Eval Dimensions by System Type" section:

```markdown
### ML Pipelines

_See also: [ml-pipeline.md](archetypes/ml-pipeline.md)_

- Model performance metrics (accuracy, precision, recall, F1, AUC)
- Data quality compliance (schema validation, null/outlier handling)
- Drift detection (data drift, concept drift, distribution shifts)
- Training reproducibility (same data + config = same model)
- Feature correctness (transformations produce expected outputs)
- Inference latency and throughput
- Rollback trigger accuracy
- Bias/fairness metrics

### API Services / Tool Providers

_See also: [api-service.md](archetypes/api-service.md)_

- Contract compliance (responses match declared schema)
- Error format consistency (uniform error envelope)
- Parameter validation coverage
- Idempotency (repeated calls produce same result)
- Latency percentiles (p50, p95, p99)
- Backward compatibility (old clients still work)
- Authentication/authorization enforcement
```

- [ ] **Step 3: Update the Cross-Cutting Dimension: Harness Quality section**

Add a scoping note at the start:

```markdown
_Included by default for single-agent and multi-agent archetypes. For other archetypes, apply when orchestration infrastructure is present._
```

- [ ] **Step 4: Update the Cross-Cutting Dimension: Memory Quality section**

Add a scoping note at the start:

```markdown
_Included by default for single-agent and multi-agent archetypes when persistent state is present. For other archetypes, apply when the system maintains memory across turns or sessions._
```

- [ ] **Step 5: Add a Cross-Cutting Dimension: Statistical Validation section**

Add after the existing cross-cutting dimensions:

```markdown
### Cross-Cutting Dimension: Statistical Validation

Applies to any system where correctness is a distribution property rather than binary. Most common in ml-pipeline and data-workflow archetypes, but relevant whenever aggregate metrics, drift detection, or A/B testing are part of the eval plan. See `references/eval-taxonomy.md` for the statistical validation grader methods and minimum sample sizes.
```

- [ ] **Step 6: Run markdown lint and commit**

```bash
cat > .markdownlint-cli2.yaml << 'EOF'
config:
  MD013: false
  MD033: false
  MD041: false
globs:
  - "define-evals/references/eval-guide.md"
EOF
npx markdownlint-cli2
rm .markdownlint-cli2.yaml
```

```bash
git add define-evals/references/eval-guide.md
git commit -m "feat: add archetype cross-references and scoping notes to eval guide"
```

---

### Task 6: Update example-eval-plan.md to Demonstrate Archetype Format

**Files:**
- Modify: `define-evals/references/example-eval-plan.md`

**Interfaces:**
- Consumes: Updated template from Task 3
- Produces: Example eval plan showing archetype-driven format. Referenced by SKILL.md as a filled-in example.

- [ ] **Step 1: Add archetype fields to the Summary section**

After the `**Status**: Approved` line, add:

```markdown
**Archetypes**: user-facing-product, single-agent
**Primary archetype**: user-facing-product
```

- [ ] **Step 2: Replace the Non-Functional Metrics section with Archetype Metrics**

Replace the "Non-Functional Metrics" section with:

```markdown
## Archetype Metrics

### User-Facing Product Metrics (Primary)

| Metric | Budget | Gate Type |
|--------|--------|-----------|
| Task completion rate | >85% | Tracked |
| Turn efficiency (≤5 turns) | >80% | Tracked |
| Error recovery clarity | >90% | Tracked |
| Perceived latency (time to first token) | <2s | Tracked |
| Tone consistency | >90% | Tracked |
| Escalation accuracy | >95% | Hard gate |
| Refusal handling (graceful decline) | >90% | Tracked |

### Single-Agent Metrics (Secondary)

| Metric | Budget | Gate Type |
|--------|--------|-----------|
| Routing accuracy (correct tool selected) | >90% | Tracked |
| Premature completion rate | <5% | Tracked |
| Fault recovery rate | >80% | Tracked |

### Universal Metrics

| Metric | Budget | Gate Type |
|--------|--------|-----------|
| Cost per task | <$0.05 per conversation | Tracked |
| Latency per task | <3s per response | Tracked |
| Token usage per task | <2000 tokens per turn | Tracked |
| Safety (adversarial pass rate) | >95% | Hard gate |
| Robustness (edge case pass rate) | >80% | Tracked |
| Governance (PII exposure rate) | 0% | Hard gate |
```

- [ ] **Step 3: Remove the standalone Harness Metrics and Memory Metrics sections**

Delete the "Harness Metrics" section (routing accuracy and premature completion are now in the Single-Agent Metrics table). Delete the "Memory Metrics" section (it already says "Not applicable" for this example).

- [ ] **Step 4: Run markdown lint and commit**

```bash
cat > .markdownlint-cli2.yaml << 'EOF'
config:
  MD013: false
  MD033: false
  MD041: false
globs:
  - "define-evals/references/example-eval-plan.md"
EOF
npx markdownlint-cli2
rm .markdownlint-cli2.yaml
```

```bash
git add define-evals/references/example-eval-plan.md
git commit -m "feat: update example eval plan to demonstrate archetype-driven format"
```

---

### Task 7: Update CI Validation, Sources, and README

**Files:**
- Modify: `.github/workflows/validate.yml`
- Modify: `define-evals/references/sources.md`
- Modify: `README.md`

**Interfaces:**
- Consumes: New archetype file paths from Task 1
- Produces: CI checks for archetype files, updated repository structure in README

- [ ] **Step 1: Add archetype file checks to validate.yml**

In the "Check reference files exist" step, add checks for the archetype directory and files:

```yaml
      - name: Check reference files exist
        run: |
          test -f define-evals/references/eval-guide.md
          test -f define-evals/references/eval-taxonomy.md
          test -f define-evals/references/eval-plan-template.md
          test -f define-evals/references/sources.md
          test -f define-evals/references/example-eval-plan.md
          test -d define-evals/references/archetypes
          test -f define-evals/references/archetypes/single-agent.md
          test -f define-evals/references/archetypes/multi-agent.md
          test -f define-evals/references/archetypes/rag-pipeline.md
          test -f define-evals/references/archetypes/user-facing-product.md
          test -f define-evals/references/archetypes/ml-pipeline.md
          test -f define-evals/references/archetypes/data-workflow.md
          test -f define-evals/references/archetypes/content-generation.md
          test -f define-evals/references/archetypes/api-service.md
          echo "All reference files present"
```

- [ ] **Step 2: Update sources.md**

Add a new section for archetype-related sources (to be populated during implementation as research is done):

```markdown
## Archetype Evaluation Research

_These sources inform the archetype-specific eval methodology. Additional sources to be added as research progresses._

- Multi-agent evaluation patterns — coordination metrics, failure cascade measurement
- Statistical validation methods for ML pipelines — confidence intervals, hypothesis testing for model evaluation
- UX evaluation for AI products — user journey testing, accessibility compliance measurement
```

- [ ] **Step 3: Update README.md repository structure**

Update the repository structure diagram to include the archetypes directory:

```markdown
eval-driven-development/
├── define-evals/                      # The Agent Skill
│   ├── SKILL.md                       # Core skill instructions
│   └── references/
│       ├── archetypes/                # Per-archetype checklists and metrics
│       │   ├── single-agent.md
│       │   ├── multi-agent.md
│       │   ├── rag-pipeline.md
│       │   ├── user-facing-product.md
│       │   ├── ml-pipeline.md
│       │   ├── data-workflow.md
│       │   ├── content-generation.md
│       │   └── api-service.md
│       ├── eval-taxonomy.md           # Grader types and tradeoffs
│       ├── eval-guide.md              # Task writing rules and dimensions
│       ├── eval-plan-template.md      # Output format for eval plans
│       ├── example-eval-plan.md       # Filled-in example eval plan
│       └── sources.md                 # All verified citations
├── docs/                              # Research and design specs (not part of skill)
│   └── superpowers/specs/
├── .github/workflows/                 # CI: validation and release
├── LICENSE
└── README.md
```

- [ ] **Step 4: Update the "What It Does" section in README.md**

Update item 2 to mention the 4th grader type:

```markdown
2. Grader selection for each task (code-based, LLM-as-judge, outcome verification, or statistical validation)
```

- [ ] **Step 5: Run full markdown lint and commit**

```bash
cat > .markdownlint-cli2.yaml << 'EOF'
config:
  MD013: false
  MD033: false
  MD041: false
globs:
  - "define-evals/**/*.md"
EOF
npx markdownlint-cli2
rm .markdownlint-cli2.yaml
```

```bash
git add .github/workflows/validate.yml define-evals/references/sources.md README.md
git commit -m "chore: update CI, sources, and README for archetype-driven methodology"
```
