# Eval Task Writing Guide

Practical rules for writing high-quality eval tasks, distilled from research on agent evaluation. See `sources.md` in this directory for the full reference list.

## Task Structure

Every eval task must include:

```yaml
- id: short-descriptive-id
  requirement: which spec requirement this tests
  description: what this task tests, in one sentence
  input: the prompt, context, or scenario
  expected_behavior: what success looks like (NOT exact output)
  grader_type: code | llm-judge | outcome
  grader_logic: what to check and how
  category: capability | regression
  priority: P0 | P1 | P2
```

## Task Writing Rules

### 1. Make Tasks Unambiguous
Two domain experts should independently reach the same pass/fail verdict on any given output. Ambiguity in task specs becomes noise in metrics.
### 2. Include Reference Solutions
Each task should have a known-good solution that proves the task is solvable and verifies grader configuration. A 0% pass rate across many trials is most often a broken task, not an incapable agent.
### 3. Grade Outcomes, Not Paths
Don't check exact tool call sequences. Agents can find creative but valid solutions that rigid path-checking would fail unfairly.

### 4. Build Balanced Problem Sets
Test both where a behavior SHOULD and SHOULD NOT occur. One-sided evals create one-sided optimization. For example, a search agent needs evals for queries requiring search AND queries it should answer from existing knowledge.
### 5. Build In Partial Credit
An agent that identifies the problem but fails to execute the fix is meaningfully better than one that fails immediately. Design graders to distinguish degrees of success.
### 6. Start Small
20-50 tasks from real or anticipated use cases is a strong start. Early agents show large effect sizes, so small sample sizes suffice. Evals get harder to build the longer you wait.
### 7. Isolate Trials
Each trial must start from a clean environment. Shared state causes correlated failures. Agents have been observed gaining unfair advantages by examining artifacts (git history, logs, temp files) from previous trials.

### 8. Run Multiple Trials
LLM outputs vary between runs. A single trial is statistically meaningless. Run 3-10 trials per task depending on the stakes.

### 9. Read the Transcripts
You won't know if graders work well unless you read transcripts and grades from many trials. Failures should seem fair. Reading transcripts is how you verify evals measure what matters.
### 10. Watch for Saturation
An eval at 100% provides no improvement signal. When evals saturate, replace them with harder tasks that test the next level of capability.
## Common Eval Dimensions by System Type

### Coding Agents
- Tool selection correctness
- Code compilation and test passage
- Auth gating and permission enforcement
- Prompt injection resistance
- Multi-step task completion
- Graceful termination and error recovery

### Conversational / Customer Support Agents
- Task resolution rate (did the agent solve the user's problem?)
- Turn efficiency (resolved in under N turns?)
- Tone and empathy appropriateness
- Escalation accuracy (knows when to hand off to human)
- Policy compliance (follows business rules)
- Refusal handling (declines out-of-scope requests gracefully)
- Multi-turn consistency (maintains consistent behavior, knowledge, and personality across the conversation — distinct from state management which is about data)

### Research / Analysis Agents
- Factual accuracy and source grounding
- Completeness of analysis (covers all relevant dimensions)
- Citation quality (authoritative, relevant sources)
- Reasoning quality (logical, non-contradictory)
- Appropriate hedging (distinguishes certainty levels)

### Content Generation
- Adherence to style guide / brand voice
- Factual accuracy
- Audience appropriateness
- Format compliance (length, structure, tone)
- Originality and relevance

### RAG Systems
- Retrieval groundedness (claims supported by sources)
- Source attribution accuracy
- Hallucination boundary (declines when retrieval is empty)
- Query understanding across phrasings
- Chunk relevance ranking

### Data / Workflow Automation
- Output correctness (matches expected data transformations)
- Error handling (graceful failure on malformed input)
- Idempotency (running twice produces same result)
- Edge case handling (nulls, empty sets, large inputs)

### Agent Harness / Scaffold
- Routing correctness (right agent/tool for the task — accuracy degrades past 15-20 tools)
- Context assembly quality (are the right facts in the window when the model reasons?)
- Context compaction fidelity (do critical instructions survive window compression?)
- Hot/cold memory separation (always-loaded conventions vs. on-demand retrieval)
- Error handling and recovery (inject faults: 500s, malformed JSON, timeouts — does the agent recover or loop?)
- Cost and latency governance (do orchestration loops drain tokens? are execution steps bounded?)
- Multi-agent coordination and handoff (correct specialist selected, clean state transfer)
- State management across turns and sessions
- Sensor coverage (do your checks — linters, type checkers, validators — actually fire? silence may mean blind spots, not quality)
- Harness transferability (does your scaffold work across different models? infrastructure improvements often generalize)
- Premature completion detection (does the agent declare done before finishing?)
- Over-ambition detection (does the agent exhaust its context window mid-task?)

### Cross-Cutting Dimension: Harness Quality
Applies to any system where infrastructure (routing, context assembly, tool dispatch, error recovery) mediates between the model and the task. Research shows scaffold differences can dominate outcomes even under fixed base models — the harness is often the bottleneck, not the LLM. Evaluate the harness independently: does routing select the right specialist? Does context compaction preserve critical instructions? Do sensors (linters, validators, checks) actually fire, or are they silent blind spots? Does the system recover from injected faults or loop indefinitely? Harness improvements tend to transfer across models, making them high-leverage eval targets.

### Cross-Cutting Dimension: Multi-Turn Consistency
Applies to any agent that operates over multiple turns within a single session. The agent should maintain consistent behavior, knowledge claims, and personality throughout a conversation. Research consistently highlights this as a key failure mode — agents that contradict themselves, forget earlier context, or shift tone unpredictably across turns. Test by designing multi-turn scenarios where later turns reference or depend on earlier ones.

### Cross-Cutting Dimension: Cross-Session Coherence
Distinct from multi-turn consistency. Applies to any agent that persists state across separate sessions (hours, days apart). Agents that score perfectly on single-session benchmarks can plummet to 40-60% on multi-session tasks where later sessions depend on information from earlier ones. Test by designing eval scenarios that span multiple sessions with dependencies between them.

### Cross-Cutting Dimension: Memory Quality (for agents with persistent memory)
Applies to any agent that writes, stores, retrieves, or forgets information across interactions. Evaluate across four layers:

1. **Task effectiveness** — Does memory improve task outcomes compared to a memoryless baseline?
2. **Memory quality** — Retrieval precision/recall, contradiction rate, staleness of recalled facts, coverage of task-relevant information.
3. **Efficiency** — Latency per memory operation, tokens consumed by memory content, storage growth over time.
4. **Governance** — Privacy leakage rate, deletion compliance, access-scope violations.

Key eval patterns for memory:
- **Selective forgetting**: Can the agent discard outdated information without losing critical facts? Inability to forget gradually poisons retrieval precision over long-running deployments.
- **Consolidation quality**: When the agent compresses or summarizes memories, does safety-critical information survive?
- **Contradiction handling**: When new information conflicts with stored information, does the agent resolve it correctly?
- **Memory attribution**: Can the agent distinguish what it knows from memory vs. parametric knowledge? Agents sometimes ignore retrieved records in favor of stale parametric knowledge.
- **Memory diff debugging**: Track what changed in memory between turns — more diagnostic than traditional logs for identifying whether failures stem from retrieval (wrong records), write path (never stored), compression (detail lost), or reasoning (correct records, wrong conclusion).

## Anti-Patterns to Avoid

1. **Vibe-based development**: No evals at all, shipping on intuition.
2. **Generic metrics**: Off-the-shelf hallucination scores that don't correlate with user satisfaction.
3. **Over-specifying paths**: Checking exact tool call sequences instead of outcomes.
4. **Single trials**: One run per task treated as definitive.
5. **Shared state**: Leftover data between trials causing correlated failures. Agents have been observed gaining unfair advantages by examining artifacts from previous trials.
6. **Pre-deployment only**: 93% of evaluations happen before deployment, only 2% post.
7. **Stale evals**: Writing evals once and never updating them.
8. **Tool fixation**: Believing another eval tool will fix process problems.
9. **No human oversight**: Over-relying on automated judges without calibration.
