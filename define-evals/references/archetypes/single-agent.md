# Single-Agent Archetype

## Signature Patterns

One LLM, tool use, single-turn or multi-turn interaction.

**Examples:** Coding assistant, search agent, CLI tool, file editor.

## Extraction Checklist

When analyzing a spec for single-agent systems, look for:

- Tool roster (what tools are available, what do they do?)
- Decision points (where does the agent choose between actions?)
- Context window management (how does it handle long contexts?)
- Termination conditions (how does it know it's done?)
- Error handling (what happens when a tool call fails?)
- Retry/recovery strategy

## Default Eval Dimensions

- Tool selection accuracy
- Turn efficiency (resolved in under N turns)
- Cost per task (tokens, API calls)
- Premature completion rate
- Graceful termination (stops when done, not mid-task)
- Prompt injection resistance
- Context compaction fidelity (critical instructions survive window compression)
- Fault recovery (recovers from tool errors, timeouts, malformed responses)
- Sensor coverage (safety/quality checks actually fire when triggered)
- Over-ambition rate (agent exhausts context window mid-task)
- Skill/capability invocation accuracy (uses intended skill, avoids irrelevant ones)
- Memory quality (retrieval precision/recall, contradiction rate) — if persistent state
- Cross-session coherence — if multi-session

## Default Grader Patterns

| Requirement Type | Grader Type | Notes |
| ---------------- | ----------- | ----- |
| Tool selection accuracy | code-based | Check tool name and params |
| Turn efficiency | code-based | Count turns |
| Cost per task | code-based | Sum tokens/API calls |
| Premature completion rate | code-based | Check termination state |
| Graceful termination | outcome verification | Check final state |
| Prompt injection resistance | code-based + LLM-as-judge | Verify agent resists manipulation |
| Context compaction fidelity | LLM-as-judge | Check instruction following post-compaction |
| Fault recovery | outcome verification | Inject fault, check recovery |
| Skill/capability invocation | code-based | Check invocation log |
| Memory quality | code-based + LLM-as-judge | Retrieval precision/recall + contradiction detection |
| Cross-session coherence | LLM-as-judge | Compare behavior across sessions |
| Sensor coverage | code-based | Verify checks fire when triggered |
| Over-ambition rate | code-based | Detect context window exhaustion mid-task |

## Common Failure Modes

- Agent selects the wrong tool when multiple similar tools are available
- Agent declares "done" before the task is actually complete (premature completion)
- Agent loops indefinitely on tool errors instead of recovering
- Agent loses critical instructions after context window compression
- Agent exhausts context window on long tasks (over-ambition)
- Agent uses brittle general knowledge instead of the intended skill

## Example Eval Tasks

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
