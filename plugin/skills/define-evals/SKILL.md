---
description: Define evals from an approved design spec before writing implementation plans. Use after brainstorming produces a spec and before invoking writing-plans. Triggers on phrases like "define evals", "write evals for this spec", "eval plan", or when transitioning from a completed design to implementation planning. Works for any AI system — coding agents, customer support agents, research agents, RAG systems, content generation, data pipelines, workflow automation, or any LLM-powered application.
---

# Define Evals From Spec

Turn an approved design spec into a concrete eval plan — eval tasks, graders, problem sets, and pass criteria — so the implementation plan is written against measurable success criteria, not just prose.

This skill is domain-agnostic. It works for any AI system: coding agents, conversational agents, customer support bots, research agents, RAG pipelines, content generation, data analysis, workflow automation, or any LLM-powered application. The eval patterns adapt to the domain — what changes is the specific tasks and graders, not the methodology.

This skill sits between brainstorming and planning:

```text
Brainstorming (approved spec)
  → THIS SKILL (eval plan from spec)
    → Writing Plans (implementation plan informed by evals)
```

<HARD-GATE>
Do NOT proceed without an approved design spec. An approved spec means: a document that has been explicitly marked as approved or finalized by the user, OR a spec the user confirms is ready for eval definition in this conversation. If no spec exists, tell the user to complete brainstorming first. Do NOT skip steps or jump to implementation.
</HARD-GATE>

## Checklist

You MUST create a task for each item and complete them in order:

1. **Locate the spec** — Find the approved design doc (ask the user if unclear)
2. **Extract requirements** — Pull every requirement, constraint, and behavior from the spec
3. **Classify each requirement** — Deterministic (code-based grader) vs. subjective (LLM-as-judge) vs. environmental (outcome verification)
4. **Generate eval tasks** — Write concrete eval tasks for each requirement
5. **Build balanced problem sets** — Add negative cases, boundary cases, and edge cases for each task
6. **Select graders** — Choose grader type and define grading logic for each task
7. **Define pass criteria** — Set thresholds, choose pass@k vs. pass^k, define regression vs. capability classification
8. **Present eval plan** — Show the complete plan to the user for approval
9. **Coverage check** — Scan the spec for components not covered by the eval plan, especially agent/LLM components
10. **Write eval plan file** — Save to `docs/eval-plans/YYYY-MM-DD-<topic>-eval-plan.md`
11. **Transition** — Invoke writing-plans skill, passing the eval plan as acceptance criteria

## Phase 1: Locate the Spec

Find the approved design spec. Check in order:

- Was a spec file path mentioned in conversation?
- Does `docs/superpowers/specs/` contain a recent spec?
- Ask the user to point you to the spec.

## Phase 2: Extract Requirements

Read the spec thoroughly. Pull every requirement, constraint, and behavior. Identify:

- **Functional requirements** — What the system must do
- **Non-functional requirements** — Performance, cost, latency constraints
- **Security constraints** — Auth, injection resistance, data handling
- **Integration points** — APIs, tools, external systems
- **User-facing behaviors** — How the system interacts with users

## Phase 3: Classify Each Requirement

For each requirement from the spec, classify it into one of three grader categories:

### Code-Based (Deterministic)

Requirements with objectively verifiable outcomes. Use when:

- There is a single correct answer or a finite set of valid answers
- Success can be checked by inspecting output format, state changes, or tool calls
- The check is binary: it either happened or it didn't

Examples: tool selection correctness, auth gating, format compliance, API response codes, database state after operation, retrieval groundedness checks.

### LLM-as-Judge (Subjective)

Requirements where quality is a spectrum and reasonable people could disagree. Use when:

- Multiple valid outputs exist for the same input
- Quality requires nuanced judgment (tone, helpfulness, completeness)
- The grading criteria are expressible as a rubric but not as code

Examples: response quality, explanation clarity, appropriate level of detail, conversation flow, error message helpfulness.

### Outcome Verification (Environmental)

Requirements verified by checking the state of the world after the agent acts. Use when:

- The agent modifies external state (files, databases, APIs)
- Success means the environment is in the correct state, regardless of the path taken
- The outcome matters more than the process

Examples: file was created with correct contents, database record exists, API was called with correct parameters, deployment succeeded.

**Note on memory-bearing systems:** If the spec describes an agent with persistent memory (cross-session state, user profiles, learned preferences, knowledge accumulation), treat memory operations as a distinct eval surface. Classify memory requirements separately: can it retrieve the right information? Can it forget when required? Does it handle contradictions? Does retrieval degrade as the store grows? See `references/eval-guide.md` for the full memory quality framework.

**Note on agent harnesses/scaffolds:** If the spec describes orchestration infrastructure (routing, context assembly, tool dispatch, multi-agent coordination, error recovery), evaluate the harness independently from the agents running inside it. Scaffold differences can dominate outcomes even under fixed base models — the infrastructure is often the bottleneck, not the LLM. Key harness eval surfaces: routing correctness (especially with >15 tools), context compaction fidelity, sensor coverage (do checks actually fire?), fault recovery, and premature completion detection. See `references/eval-guide.md` for the full harness eval checklist.

Read `references/eval-taxonomy.md` for the complete grader taxonomy with methods and tradeoffs.

## Phase 4: Generate Eval Tasks

For each classified requirement, write a concrete eval task. Each task MUST include:

```yaml
- id: <short-descriptive-id>
  requirement: <which spec requirement this tests>
  description: <what this task tests, in one sentence>
  input: <the prompt, context, or scenario given to the agent>
  expected_behavior: <what success looks like — NOT exact output>
  grader_type: code | llm-judge | outcome
  grader_logic: <specific grading approach — what to check and how>
  category: capability | regression
  priority: P0 | P1 | P2
```

### Priority Levels

- **P0**: Non-negotiable. Must pass before any deployment. Security, auth, data integrity.
- **P1**: Core functionality. Must pass before feature is considered complete.
- **P2**: Quality and polish. Important but not blocking.

### Task Writing Rules

Follow these rules from the research (read `references/eval-guide.md` for full context):

1. **Unambiguous tasks**: Two domain experts should independently reach the same pass/fail verdict. Ambiguity in task specs becomes noise in metrics.
2. **Include reference solutions**: Each task should have a known-good solution that proves solvability and verifies grader configuration.
3. **Grade outcomes, not paths**: Don't check exact tool call sequences. Check what the agent produced. For user-facing agents, prefer outcome grading: compare the environment state (database, API, file system) after the agent acts against the expected goal state. The conversation is the means; the environmental state is what matters.
4. **Build in partial credit**: An agent that identifies the problem but fails to fix it is better than one that fails immediately.
5. **Start small**: 20-50 tasks is a strong start. Don't aim for exhaustive coverage in v1.

## Phase 5: Build Balanced Problem Sets

For every eval task, add counterbalancing cases. One-sided evals create one-sided optimization.

For each task, define:

**Positive cases** — Inputs where the behavior SHOULD occur:

- Happy path (standard usage)
- Variations in phrasing, format, or complexity
- Edge cases that should still succeed

**Negative cases** — Inputs where the behavior should NOT occur:

- Inputs that look similar but shouldn't trigger the behavior
- Boundary cases just outside the expected scope
- Adversarial inputs (prompt injection, out-of-scope requests)

**Boundary cases** — Inputs at the exact boundary:

- Ambiguous inputs where the correct behavior is debatable
- Cases requiring judgment calls about escalation vs. handling

Target ratio: roughly 40% positive, 40% negative, 20% boundary for each eval dimension.

## Phase 6: Select Graders and Define Logic

For each task, define the specific grading approach:

### Code-Based Graders

```text
- String matching (exact or fuzzy)
- Regex patterns
- JSON schema validation
- Tool call verification (correct tool, correct params)
- State diff (before/after comparison)
- Static analysis (linters, type checks)
- Binary tests (does it compile, do tests pass)
```

### LLM-as-Judge Graders

```text
- Rubric: Define 3-5 specific criteria the judge evaluates
- Default quality dimensions: coherence, accuracy, clarity, relevance, efficiency
  — adapt these to your domain; they cover the most common real-world LLM usage patterns
- Scoring: PASS/FAIL (not Likert scales — binary forces clarity)
- Judge model: Use the most powerful model affordable
- Isolation: One judge per dimension, not one judge for everything
- Escape hatch: Include instruction to return "Unknown" when uncertain
- Calibration plan: Note that human calibration is needed (25-50 examples)
```

### Outcome Verification Graders

```text
- Environment state checks (file exists, DB record, API state)
- Diff-based (expected state vs. actual state)
- Idempotency checks (running twice produces same outcome)
```

## Phase 7: Define Pass Criteria

For the overall eval plan, define:

### Per-Task Criteria

- **Threshold**: What pass rate is acceptable? (e.g., 90% for P0, 80% for P1)
- **Trials**: How many trials per task? (minimum 3 for statistical meaning, 5-10 for agents)
- **Metric**: pass@k (at least one success) or pass^k (all must succeed)?
  - **Default to pass^k for user-facing agents.** Research shows even state-of-the-art agents pass less than half of customer service tasks on any single try, and pass^8 drops below 25%. An agent that looks decent on one try is unreliable over multiple interactions. Users expect consistent behavior, not coin flips.
  - Use pass@k only for capability exploration or internal tooling where retries are acceptable.

### Suite-Level Criteria

- **Regression floor**: P0 evals must maintain near-100% pass rate
- **Capability target**: P1/P2 evals start low, define the hill to climb
- **Graduation rule**: When a capability eval hits sustained >95%, it graduates to regression
- **Saturation rule**: When a regression eval hits 100% for N consecutive runs, flag for replacement with harder tasks

### Non-Functional Metrics (The Underexplored Gaps)

Research consistently finds that these dimensions are the most under-evaluated across all agent types. They MUST be part of every eval plan, not afterthoughts:

- **Cost-efficiency**: Token usage, latency, cost per task, API call count. Set budgets early.
- **Safety**: Harmful output detection, policy compliance, data leakage, PII handling. Include adversarial test cases.
- **Robustness**: Behavior under malformed input, edge cases, adversarial prompts, partial failures. Does the agent degrade gracefully or catastrophically?
- **Governance**: Privacy leakage rate, deletion compliance (can the system forget when required?), access-scope violations. Critical for any agent handling user data or operating under regulatory constraints.

Track these alongside correctness. Start as tracked metrics, promote to pass/fail gates as baselines stabilize.

### Harness-Specific Metrics (for systems with orchestration infrastructure)

If the system has routing, context assembly, tool dispatch, or multi-agent coordination, add these dimensions:

- **Routing accuracy**: Does the right tool or specialist get selected? Accuracy degrades past 15-20 tools — test with increasing tool counts.
- **Context compaction fidelity**: Do critical instructions survive window compression? Test by checking whether the agent follows early instructions after compaction occurs.
- **Sensor coverage**: Do your checks (linters, validators, type checkers) actually fire on relevant input? Silence may mean blind spots, not quality.
- **Fault recovery**: Inject faults (500s, malformed JSON, timeouts) and verify the agent recovers rather than looping indefinitely.
- **Premature completion**: Does the agent declare "done" before the task is actually finished?
- **Over-ambition**: Does the agent exhaust its context window mid-task? Test with tasks that require sustained reasoning across many steps.
- **Model transferability**: Does the harness work across different LLMs? Infrastructure improvements tend to generalize — test this.

Scaffold differences can dominate outcomes even under fixed base models. Evaluate the harness independently from the agents running inside it.

### Memory-Specific Metrics (for agents with persistent state)

If the system maintains memory across turns or sessions, add these dimensions:

- **Memory quality**: Retrieval precision/recall, contradiction rate, staleness distribution of recalled facts.
- **Selective forgetting**: Can the agent discard outdated information without losing critical facts?
- **Cross-session coherence**: Does the agent maintain consistent behavior across sessions separated by time?
- **Consolidation fidelity**: When memories are compressed or summarized, does safety-critical information survive?

Memory architecture often matters more than model choice — the gap between "has memory" and "no memory" frequently exceeds the gap between different model versions. Eval accordingly.

## Phase 8: Present Eval Plan

Present the complete eval plan to the user organized by priority:

1. **P0 evals** (must-pass) — show first, get explicit approval
2. **P1 evals** (core functionality) — show next
3. **P2 evals** (quality) — show last
4. **Suite-level criteria** — thresholds, metrics, graduation rules

Wait for user approval before proceeding.

## Phase 9: Coverage Check

After presenting the eval plan, scan the original spec for components that the eval plan does NOT cover. This is a gap-detection step — the most common failure mode is scoping down to the easy deterministic pieces while quietly omitting the agent/LLM components that actually need evaluation.

Check for:

1. **Uncovered agent components**: Does the spec describe agents, skills, or sub-agents that make LLM calls? If so, does the eval plan include LLM-as-judge graders and pass^k metrics for those components, or did it only cover the deterministic infrastructure around them?

2. **Uncovered LLM call paths**: Trace every path in the spec where an LLM is invoked (generation, extraction, classification, summarization, tool selection). Each path needs at least one eval task. If any path has zero coverage, flag it.

3. **Missing grader diversity**: If the spec describes a system with both deterministic and non-deterministic components but the eval plan only has code-based graders, the non-deterministic parts are unevaluated. Flag this explicitly.

4. **"Evaluated separately" without a plan**: If the eval plan scopes out any component with language like "evaluated separately" or "tested in a different plan," verify that a separate eval plan exists or is planned. If not, either include it in this plan or surface it to the user as a gap that needs its own eval plan.

Present any gaps to the user:

> "The eval plan covers [X, Y, Z] but the spec also describes [A, B] which are not covered. These components make LLM calls and need their own eval tasks with [LLM-as-judge / pass^k / outcome verification]. Want me to:
>
> 1. Add eval tasks for these components to this plan?
> 2. Create a separate eval plan for them?
> 3. Consciously skip them (document why)?"

Do NOT proceed to writing the eval plan file until all gaps are resolved — either covered or explicitly skipped with the user's approval.

## Phase 10: Write Eval Plan File

After user approval, write the eval plan. Default path:
`docs/eval-plans/YYYY-MM-DD-<topic>-eval-plan.md`

If the project already has an eval-related directory (e.g., `evals/`, `tests/evals/`, `evaluation/`), use that instead. Ask the user if multiple candidates exist.

Use the structure from `references/eval-plan-template.md` as the output format.

Commit the eval plan.

## Phase 11: Transition to Implementation Planning

After the eval plan is approved and committed, invoke the **writing-plans** skill. When framing the implementation plan:

- Reference the eval plan file as acceptance criteria
- Each implementation step should note which eval tasks it must pass
- The plan should include an "eval harness setup" phase early in implementation
- CI/CD integration for running evals should be part of the plan

The implementation is not done until the evals pass. The evals are the definition of done.

## Key Principles

- **Evals before code**: The eval plan defines what "done" means. Implementation follows.
- **Balanced coverage**: Always test both positive and negative cases.
- **Grade outcomes, not paths**: Don't over-specify how the agent should solve the problem.
- **Start small, iterate**: 20-50 tasks beats 0 tasks. Expand from real failures later.
- **Binary judgments**: PASS/FAIL, not scales. A fail is actionable; a "3" is ambiguous.
- **No single layer catches everything**: Combine code-based, LLM-as-judge, and outcome verification.
- **Evals are living artifacts**: They graduate, saturate, and get replaced. Plan for this.
