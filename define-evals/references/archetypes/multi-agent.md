# Multi-Agent Archetype

## Signature Patterns

Multiple agents, delegation, coordination, shared state, supervisor/worker patterns.

**Examples:** Research swarm, pipeline of specialists, supervisor pattern, debate architecture.

## Extraction Checklist

When analyzing a spec for multi-agent systems, look for:

- Agent roster (who are all the agents, what are their roles?)
- Delegation patterns (who calls whom, under what conditions?)
- Shared state (what state is shared between agents, how is it synchronized?)
- Handoff protocols (how does work transfer between agents?)
- Failure cascades (if agent A fails, what happens to agents B and C?)
- Conflict resolution (what happens when two agents produce contradictory outputs?)
- Coordination overhead (how much extra cost/latency does coordination add?)
- Authority hierarchy (who has final say?)

## Default Eval Dimensions

- Delegation accuracy (correct specialist selected for sub-task)
- Handoff fidelity (information preserved across agent boundaries)
- Failure cascade containment (one agent's failure doesn't cascade)
- Coordination overhead (extra tokens/latency vs. single-agent baseline)
- Conflict resolution rate (contradictions resolved correctly)
- End-to-end task completion (the whole system achieves the goal)
- Deadlock/livelock detection (agents don't get stuck in loops)
- Routing accuracy (right specialist selected, degrades past 15-20 tools)
- Fault recovery (agent-level and system-level error handling)
- Sensor coverage (safety/quality checks actually fire when triggered)
- Model transferability (harness works across different LLMs)
- Memory quality and cross-session coherence — if shared/persistent state
- Consolidation fidelity (safety-critical info survives memory compression)

## Default Grader Patterns

| Requirement Type | Grader Type | Notes |
| ---------------- | ----------- | ----- |
| Delegation accuracy | code-based | Check which agent was selected |
| Handoff fidelity | LLM-as-judge | Compare information before/after handoff |
| Failure cascade containment | outcome verification | Inject failure, check other agents |
| Coordination overhead | code-based | Measure tokens/latency delta vs baseline |
| Conflict resolution rate | LLM-as-judge | Check contradictions resolved |
| End-to-end task completion | outcome verification | Check final state |
| Deadlock/livelock detection | code-based | Check for infinite loops or stuck states |
| Routing accuracy | code-based | Check specialist selection |
| Fault recovery | outcome verification | Inject fault, verify system recovery |
| Memory quality | code-based + LLM-as-judge | Retrieval metrics + contradiction detection |
| Consolidation fidelity | LLM-as-judge | Check safety-critical info survives compression |

## Common Failure Modes

- Wrong specialist selected for sub-task (delegation error)
- Information lost during agent handoff (context dropped at boundary)
- One agent's failure cascades to others (no isolation)
- Agents deadlock waiting for each other
- Agents produce contradictory outputs with no resolution
- Coordination overhead exceeds single-agent baseline (slower, not faster)
- Shared state becomes inconsistent under concurrent access

## Example Eval Tasks

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
