# Eval Grader Taxonomy

Quick reference for choosing grader types. See `sources.md` in this directory for the full reference list.

## Grader Selection Guide

```text
Is there a single correct answer or finite set of valid answers?
  → Yes → CODE-BASED grader (string match, binary test, state diff)
  → No  → Does the agent modify external state (files, DB, APIs)?
            → Yes → OUTCOME VERIFICATION grader (state diff, API check, idempotency)
            → No  → LLM-AS-JUDGE grader (rubric scoring, pairwise comparison)

Second pass — does this task ALSO have structural constraints (format, schema, tool calls)?
  → Yes → Add a CODE-BASED grader alongside the primary grader
```

Most real-world tasks benefit from multiple grader types — code-based for structure, LLM-as-judge for quality, outcome verification for environmental state. Default to combining graders rather than picking exactly one.

## Code-Based Graders

| Method | Description | Best For |
| -------- | ------------- | ---------- |
| String matching | Exact or fuzzy match on output | Classification, entity extraction |
| Binary tests | Does the code run? Do tests pass? | Coding agents |
| Static analysis | Linters, type checkers, security scanners | Code quality |
| Outcome verification | Check environment state after execution | Tool-using agents |
| Tool call verification | Right tools called with right params? | Agent harness routing |
| Transcript analysis | Pattern matching on interaction log | Multi-turn behavior |

**Use when**: There is a single correct answer, success is binary, or the check is deterministic.
**Strengths**: Fast, cheap, objective, reproducible.
**Weaknesses**: Brittle to valid variations, lacks nuance.

## LLM-as-Judge Graders

| Method | Description | Best For |
| -------- | ------------- | ---------- |
| Rubric-based scoring | Judge scores against explicit criteria | Quality assessment |
| Natural language assertions | Judge evaluates free-form claims | Flexible quality checks |
| Pairwise comparison | Judge picks the better of two outputs | A/B testing, model comparison |
| Reference-based evaluation | Judge compares to reference answer | RAG groundedness |
| Multi-judge consensus | Multiple judges vote, majority wins | High-stakes decisions |

**Use when**: Multiple valid outputs exist, quality is subjective, or grading needs nuance.
**Strengths**: Flexible, scalable, captures nuance.
**Weaknesses**: Non-deterministic, expensive, requires calibration.

### LLM-as-Judge Best Practices

- Use PASS/FAIL, not Likert scales. Binary forces clarity.
- Grade each dimension with an isolated judge, not one judge for all dimensions.
- Give the judge a way out — instruct it to return "Unknown" when uncertain.
- Track precision and recall separately, not raw agreement.
- Calibrate with 25-50 human-graded examples before trusting the judge.
- Use the most powerful model affordable for judging.
- Default quality dimensions when building rubrics: **coherence, accuracy, clarity, relevance, efficiency**. These five cover the most common real-world LLM usage patterns, derived from large-scale usage analysis. Adapt to your domain — e.g., "efficiency" might mean turn count for a support agent or token usage for a generation task.

## Outcome Verification Graders

| Method | Description | Best For |
| -------- | ------------- | ---------- |
| State diff | Compare environment before/after | Database, file system changes |
| API call verification | Check external API was called correctly | Integration testing |
| Idempotency check | Running twice produces same outcome | Reliability testing |

**Use when**: The agent modifies external state and the outcome matters more than the process.

## Scoring Strategies

- **Weighted**: Graders contribute proportionally to final score.
- **Binary gate**: All graders must pass (for non-negotiable constraints).
- **Hybrid**: P0 graders are binary gates; P1/P2 graders are weighted.
- **Partial credit**: Identifying the problem without fixing it scores higher than total failure.

## Non-Determinism Metrics

- **pass@k**: At least one success in k trials. Use for capability exploration.
- **pass^k**: All k trials succeed. Use for user-facing reliability.
- At k=1, both metrics are identical. By k=10, they tell opposite stories.
