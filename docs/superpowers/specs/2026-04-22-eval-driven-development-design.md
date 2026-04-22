# Eval-Driven Development: A Comprehensive Guide

A practitioner's reference for building reliable AI agents, RAG systems, and agent harnesses through systematic evaluation. Grounded in verified research and industry practice.

---

## Table of Contents

1. [What Is Eval-Driven Development](#1-what-is-eval-driven-development)
2. [Approach A: Error-Analysis-First EDD](#2-approach-a-error-analysis-first-edd)
3. [Approach B: Spec-First EDD](#3-approach-b-spec-first-edd)
4. [Approach C: Hybrid Lifecycle EDD (Recommended)](#4-approach-c-hybrid-lifecycle-edd-recommended)
5. [Eval Taxonomy](#5-eval-taxonomy)
6. [Harness Engineering & EDD](#6-harness-engineering--edd)
7. [Tooling Landscape](#7-tooling-landscape)
8. [Anti-Patterns & Pitfalls](#8-anti-patterns--pitfalls)
9. [Sources](#9-sources)

---

## 1. What Is Eval-Driven Development

### Definition

Eval-driven development (EDD) is a methodology for building AI systems where evaluations — programmatic assessments of model behavior — serve as the primary feedback mechanism driving development decisions. It adapts the core insight of test-driven development (TDD) to the non-deterministic world of LLMs: define what success looks like before (or immediately after) building, then iterate until the system meets those criteria.

An eval is a test for an AI system: give an AI an input, then apply grading logic to its output to measure success (Anthropic, "Demystifying Evals for AI Agents," Jan 2026).

### Why Traditional Testing Falls Short

TDD works because for a given input, there is a single, deterministic, knowable, correct output to assert against. LLMs break this assumption in three ways:

1. **Non-determinism**: The same prompt can produce different valid outputs across runs.
2. **Infinite output space**: Ask an LLM to draft an email, and there are thousands of valid responses.
3. **Emergent behavior**: Agents using tools across many turns can find creative solutions that exceed what static tests anticipate. Anthropic found that Opus 4.5 discovered a policy loophole on the tau2-bench benchmark that was technically a better solution but "failed" as written.

The Pragmatic Engineer newsletter identifies three fundamental gaps this creates:

- **Gulf of Comprehension** — You can't manually inspect every response at scale.
- **Gulf of Specification** — The gap between what you want and what your prompts actually instruct.
- **Gulf of Generalization** — Even with perfect instructions, models can fail on novel inputs.

### EDD vs. TDD

| Dimension | TDD | EDD |
|-----------|-----|-----|
| Output | Deterministic | Probabilistic |
| Assertion | Exact match | Scoring (0-1), rubrics, LLM-as-judge |
| Pass criteria | 100% or fail | Threshold-based (e.g., 80% pass rate) |
| Runs needed | 1 | Multiple trials per task (pass@k, pass^k) |
| Grader complexity | Simple assertions | Code-based + model-based + human |
| Failure modes | Predictable | Emergent, discovered through analysis |

### The Prediction That Defines Successful AI Products

Hamel Husain, who has consulted on dozens of LLM products, states: "Unsuccessful AI products almost always share a common root cause: a failure to create robust evaluation systems." In his experience, 60-80% of development time on successful AI products is spent on error analysis and evaluation.

### Core Terminology

These definitions come from Anthropic's evals guide and are used consistently throughout this document:

- **Task**: A single test with defined inputs and success criteria.
- **Trial**: Each attempt at a task. Multiple trials are run because model outputs vary between runs.
- **Grader**: Logic that scores some aspect of the agent's performance. A task can have multiple graders.
- **Transcript** (trace/trajectory): The complete record of a trial — outputs, tool calls, reasoning, intermediate results.
- **Outcome**: The final state in the environment after a trial (e.g., whether a database record actually exists, not just whether the agent said it was created).
- **Evaluation harness**: Infrastructure that runs evals end-to-end — provides instructions/tools, runs tasks concurrently, records steps, grades outputs, aggregates results.
- **Agent harness (scaffold)**: The system enabling a model to act as an agent — processes inputs, orchestrates tool calls, returns results.
- **Evaluation suite**: A collection of tasks measuring specific capabilities or behaviors.

---

## 2. Approach A: Error-Analysis-First EDD

### Origin

This approach is championed by Hamel Husain, former GitHub ML lead and creator of the most popular AI evals course (2,000+ engineers trained, including teams at OpenAI and Anthropic). His core argument: writing evaluators before implementing features "sounds appealing but creates more problems than it solves." LLMs have infinite surface area for potential failures — you cannot predict what will go wrong before you observe it.

### Philosophy

Write evaluators for errors you discover, not errors you imagine. Start with error analysis — a decades-old ML discipline adapted from qualitative research methods like grounded theory. Let failure modes emerge from real data, bottom-up.

The exception: EDD may work for specific constraints where you know exactly what success looks like, such as "never mention competitors" or "always include a disclaimer." Writing those evaluators early is acceptable because the failure mode is known in advance.

### The Workflow

#### Step 1: Build a Custom Data Viewer

Invest in a simple, domain-specific tool for reviewing traces. Hamel emphasizes: "You must remove all friction from the process of looking at data."

Off-the-shelf tools like LangSmith, Arize, or Braintrust are good starting points but are generic. Real estate startup NurtureBoss built a custom viewer using Gradio in a few hours. The viewer should consolidate all relevant information on one screen: user messages, AI responses, tool calls, retrieved documents, and final outcomes.

For a Go + Python stack, consider:
- Python: Streamlit or Gradio for rapid prototyping of trace viewers
- Go: Embed a simple web UI or export traces to a format your Python viewer can consume
- Both: Structured JSON logging of all agent interactions from day one

#### Step 2: Open Coding (Bottom-Up Annotation)

Review at least 100 diverse traces. Write open-ended notes about observed problems. The critical rule: **avoid predefined checklists** like "hallucination" or "toxicity." Let failures emerge organically.

Bad annotation: "Hallucination detected."
Good annotation: "Agent cited a document that doesn't exist in the retrieval set. The query was about Q3 revenue but the retrieved chunks were from Q2. The agent interpolated a number."

Focus on identifying only the first upstream failure in complex traces — early errors cascade downstream.

#### Step 3: Axial Coding (Create a Taxonomy)

Group open-ended notes into 5-10 themes. An LLM can suggest initial clusters, but humans must review and refine. NurtureBoss found that just three issues accounted for most of their problems.

Example taxonomy for an agent harness:
- Tool selection errors (agent picks wrong tool for the task)
- Context window overflow (relevant info pushed out by irrelevant retrieval)
- Premature termination (agent stops before completing all steps)
- Instruction drift (agent follows early instructions, ignores later constraints)
- State management failures (agent loses track of multi-turn state)

#### Step 4: Prioritize with Data

Use a pivot table to count failure frequency. This transforms qualitative insights into a quantitative roadmap. Fix the top 3 failure modes first — they likely account for the majority of user-visible problems.

#### Step 5: Write Evals for Discovered Failures

Now — and only now — write evals. Each eval targets a specific, observed failure mode. Build a golden dataset of test cases that exercise the failure pattern with diverse variations.

#### Step 6: Fix, Measure, Repeat

Fix the top failure mode. Run evals. Confirm the fix. Check for regressions. Move to the next failure mode. This is the flywheel: **Analyze -> Measure -> Improve -> Automate -> Repeat.**

### Three Levels of Evaluation (Husain's Framework)

**Level 1: Unit Tests** — Assertion-based tests that run fast and cheaply on every code change. Break the LLM's capabilities into features and scenarios. Use synthetic test cases generated by LLMs. Track results over time via CI (GitHub Actions) and dashboards. Unlike traditional unit tests, 100% pass rate isn't necessarily required — acceptable failure rates are a product decision.

**Level 2: Human & Model Eval** — For quality dimensions that assertions can't capture. Requires logged traces. Build an LLM-based evaluator by having a human grade 25-50 examples alongside the model evaluator, then iteratively refine until they agree. Key tip: use PASS/FAIL, not Likert scales. Binary decisions force clarity. A fail is actionable; a "3" is ambiguous.

**Level 3: A/B Testing** — Traditional A/B testing applied to AI products. Only appropriate for mature products with sufficient traffic. Cost and cadence scale across levels: Level 1 runs on every commit, Level 2 on a set cadence, Level 3 only after significant product changes.

### When to Use Approach A

- You have an existing system producing real outputs
- You don't yet know what your failure modes are
- You want to avoid wasting effort on evals that don't matter
- Your domain is novel enough that failure modes aren't predictable

### Limitations

- Requires running the system first — can't eval before you build
- Slower to establish a safety net against regressions
- Risk of shipping known-bad behavior while you're still in the analysis phase

---

## 3. Approach B: Spec-First EDD

### Origin

This approach is advocated by Anthropic's engineering team and aligns with how Claude Code was developed. The core thesis: "build evals to define planned capabilities before agents can fulfill them, then iterate until the agent performs well" (Anthropic, "Demystifying Evals for AI Agents").

Eugene Yan frames this as "simply the scientific method in disguise" — a cycle of observation, hypothesis, experimentation, and measurement. ML teams have practiced this concept for decades using validation and test sets.

### Philosophy

Evals are the spec. Before building a feature, define what success looks like programmatically. This forces product teams to specify what success means for the agent, resolving ambiguity between engineers. The eval exists before the implementation, just like a test in TDD.

### The Workflow

#### Step 0: Define Success Criteria

Before writing any eval, answer: "What does this agent/feature need to do, and how will we know it did it?" Write these down as concrete, unambiguous criteria. Two domain experts should independently reach the same pass/fail verdict on any given output.

Ambiguity in task specifications becomes noise in metrics (Anthropic).

#### Step 1: Write Eval Tasks

Each task specifies:
- **Input**: The prompt, context, or scenario the agent receives
- **Expected behavior**: What success looks like (not necessarily exact output)
- **Graders**: How to score the output (code-based, LLM-based, or both)
- **Reference solution**: A known-good solution that proves the task is solvable and verifies grader configuration

Start with 20-50 tasks drawn from anticipated use cases. A 0% pass rate across many trials is most often a signal of a broken task, not an incapable agent.

#### Step 2: Build Balanced Problem Sets

Test both where a behavior should and shouldn't occur. One-sided evals create one-sided optimization.

Example from Anthropic: The Claude.ai web search team built evals covering both directions — queries requiring search AND queries the model should answer from its own knowledge. Balancing undertriggering vs. overtriggering took many rounds of refinements.

For an agent harness, this means testing:
- Tasks the agent should handle autonomously
- Tasks the agent should escalate or refuse
- Inputs that should trigger tool use vs. direct response
- Edge cases at the boundaries of agent capabilities

#### Step 3: Set Up the Eval Harness

Each trial must start from a clean environment. Shared state (leftover files, cached data) causes correlated failures from infrastructure issues rather than agent performance. Anthropic found that in some internal evals, "Claude gained an unfair advantage on some tasks by examining the git history from previous trials."

For a Go + Python multi-provider stack:
- Containerize eval environments (Docker) for isolation
- Reset all state between trials
- Support multiple LLM providers in eval configuration
- Log full transcripts for every trial

#### Step 4: Run, Read, Refine

Run the eval suite. Read the transcripts — not just the scores. "You won't know if your graders are working well unless you read the transcripts and grades from many trials" (Anthropic).

Look for:
- Grader failures (passing bad outputs, failing good ones)
- Task ambiguity (valid solutions marked as failures)
- Systematic patterns in failures

#### Step 5: Iterate on Implementation

With evals as the spec, iterate on the agent/harness implementation until pass rates reach acceptable thresholds. Capability evals should start at a low pass rate, giving teams a hill to climb. As they approach saturation, they "graduate" to become regression suites.

#### Step 6: Integrate into CI/CD

Automated evals run on each agent change and model upgrade as the first line of defense against quality problems. Regression evals should have a nearly 100% pass rate — a decline signals something is broken.

### The Claude Code Example

Claude Code started with fast iteration based on feedback from Anthropic employees and external users. They added evals first for narrow areas like concision and file edits, then for more complex behaviors like over-engineering. These evals helped identify issues, guide improvements, and focus research-product collaborations. Internally, Anthropic builds features that work today but bet on future model capabilities. Capability evals starting at low pass rates make these bets visible — when a new model drops, running the suite reveals which bets paid off.

### The Fireworks AI Workflow (Claude Code + MCP)

Fireworks AI documented a practical EDD workflow using Claude Code:

1. **Ground the agent with MCP servers** — Connect to domain-specific documentation via Model Context Protocol before generating evals
2. **Meta-prompt with structured layers** — Persona assignment, task definition, tool awareness, project-specific instructions
3. **Generate initial eval tasks** — Start with 4 diverse test cases covering core scenarios (functional, auth gating, complex queries, security)
4. **Expand with AI assistance** — Use Claude Code to expand 4 tests into 32 variations, saving hours of manual work
5. **Run and iterate** — Use structured JSONL datasets with fields: id, prompt, expected_behaviors (as arrays), test_type

Key insight: "The developer's role shifts from writing every line of code to defining the high-level goals and then supervising the AI."

### When to Use Approach B

- Building new capabilities in well-understood domains
- Non-negotiable constraints are known upfront (security, compliance, format)
- You need regression protection from day one
- Multiple engineers need to agree on what "done" means

### Limitations

- Hard to write good evals for behaviors you haven't observed yet
- Risk of testing the wrong things — evals for imagined failures waste effort
- Can create false confidence if evals don't cover real-world failure modes
- Requires significant upfront investment before any implementation begins

---

## 4. Approach C: Hybrid Lifecycle EDD (Recommended)

### Origin

This approach synthesizes insights from all sources, grounded primarily in the EDDOps academic framework (Xia et al., arXiv 2411.13768) and Anthropic's practical guidance. The EDDOps paper found that 93.28% of evaluations in the literature occur before deployment, while post-deployment (2.24%) and continuous evaluation (4.48%) remain severely underexplored. The hybrid approach corrects this imbalance.

### Philosophy

Use spec-first evals for known constraints and error-analysis-driven evals for emergent behaviors. Unify offline (development-time) and online (runtime) evaluation in a closed feedback loop. Evaluation evidence drives both runtime adaptation and governed redevelopment activities.

No single evaluation layer catches every issue. Like the Swiss Cheese Model from safety engineering, multiple methods combined ensure that failures slipping through one layer are caught by another (Anthropic).

### The Architecture

```
+------------------------------------------------------------------+
|                    HYBRID LIFECYCLE EDD                            |
|                                                                    |
|  DEVELOPMENT TIME (Offline)          RUNTIME (Online)              |
|  +---------------------------+    +---------------------------+    |
|  | Spec-First Evals          |    | Production Monitoring     |    |
|  | - Known constraints       |    | - Distribution drift      |    |
|  | - Security/auth gating    |    | - Latency/cost tracking   |    |
|  | - Format compliance       |    | - Error rate alerting     |    |
|  | - Tool call correctness   |    | - User feedback capture   |    |
|  +---------------------------+    +---------------------------+    |
|             |                                |                     |
|             v                                v                     |
|  +---------------------------+    +---------------------------+    |
|  | Error-Analysis Evals      |    | Trace Analysis            |    |
|  | - Discovered failures     |    | - Transcript review       |    |
|  | - Quality regressions     |    | - Failure taxonomy        |    |
|  | - Edge case coverage      |    | - Pattern detection       |    |
|  | - LLM-as-judge (tuned)    |    | - A/B test results        |    |
|  +---------------------------+    +---------------------------+    |
|             |                                |                     |
|             +----------- FEEDBACK LOOP ------+                     |
|                           |                                        |
|                    Eval Suite Evolution                             |
|                    - New tasks from production failures             |
|                    - Graduated capability -> regression evals       |
|                    - Saturated evals replaced with harder ones      |
+------------------------------------------------------------------+
```

### The Five Phases

#### Phase 1: Instrument First

Before writing a single eval, instrument your systems with comprehensive tracing. Every agent interaction — user input, LLM calls, tool invocations, retrieved documents, final output, outcome state — must be logged in a structured, queryable format.

For your stack:
- **Go services**: Structured JSON logging with OpenTelemetry-compatible trace IDs
- **Python agents**: Trace decorators that capture input/output/tool calls at every step
- **Multi-provider**: Normalize trace format across Claude, OpenAI, and other providers
- **Agent harness**: Log routing decisions, context assembly, and orchestration metadata

This is non-negotiable. Without traces, neither spec-first nor error-analysis evals have data to work with.

#### Phase 2: Spec-First for Known Constraints

Write evals immediately for behaviors where success is unambiguous and non-negotiable:

**For agents:**
- Tool selection correctness (given a task, does the agent pick the right tool?)
- Auth gating (does the agent refuse unauthorized operations?)
- Security (does the agent resist prompt injection?)
- Termination conditions (does the agent stop when the task is complete?)

**For RAG systems:**
- Retrieval groundedness (are claims supported by retrieved documents?)
- Source attribution (does the system cite its sources?)
- Hallucination boundaries (does the system decline when retrieval returns nothing relevant?)

**For the agent harness:**
- Routing correctness (does the harness route to the right agent/model?)
- Context assembly (does the harness provide the right context to the model?)
- Error handling (does the harness recover gracefully from model failures?)
- Cost/latency constraints (does the harness stay within budget?)

Use code-based graders wherever possible for these — they're fast, cheap, deterministic.

#### Phase 3: Ship, Observe, Analyze

Deploy with your spec-first evals as the regression floor. Then begin error analysis on real production traces:

1. **Sample 100+ traces** across diverse usage patterns
2. **Open coding** — annotate failures without predefined categories
3. **Axial coding** — group into 5-10 failure themes
4. **Prioritize** — count frequency, rank by user impact
5. **Write evals** for the top 3 failure modes

This is where you discover what you didn't know to test for. Common discoveries in agentic systems:
- The agent works on simple tasks but fails on multi-step ones requiring state tracking
- RAG retrieval is good but the agent ignores retrieved context in favor of parametric knowledge
- The harness routes correctly most of the time but fails on ambiguous queries at the boundary between two agents
- Tool calls succeed individually but the agent sequences them incorrectly

#### Phase 4: LLM-as-Judge for Subjective Quality

Some quality dimensions can't be graded with code: response helpfulness, tone appropriateness, explanation clarity, reasoning quality. For these, build LLM-as-judge evaluators:

1. **Collect human judgments** — Have domain experts grade 25-50 examples with PASS/FAIL and a written critique explaining the reasoning
2. **Build the judge** — Prompt a powerful model with a rubric derived from human judgments
3. **Calibrate** — Run the judge on the same examples humans graded. Measure agreement.
4. **Partition data** — Avoid the judge memorizing answers by measuring how well it generalizes to unfamiliar data
5. **Monitor drift** — Periodically re-calibrate against fresh human judgments

Key practices from the research:
- Use PASS/FAIL, not Likert scales (Hamel Husain). Binary decisions force clarity.
- Grade each dimension with an isolated judge rather than one judge for all dimensions (Anthropic).
- Give the judge a way out — include an instruction to return "Unknown" when uncertain (Anthropic).
- Track precision and recall separately rather than raw agreement when classes are imbalanced (Husain).

#### Phase 5: Close the Loop

This is what distinguishes the hybrid approach from the other two. The feedback loop has three mechanisms:

**Graduation**: Capability evals with consistently high pass rates graduate to become regression suites. What measured "can we do this at all?" now measures "can we still do this reliably?"

**Saturation detection**: An eval at 100% provides no improvement signal. As evals approach saturation, large capability improvements appear as small score increases. When this happens, replace saturated evals with harder ones. Qodo initially dismissed Opus 4.5 because their one-shot coding evals didn't capture gains on longer, more complex tasks — they had to build new agentic evals to see the improvement.

**Production-to-dev pipeline**: Every production failure that makes it past the eval suite becomes a new eval task. This ensures the suite evolves with real-world usage, not just developer imagination.

### Non-Determinism: pass@k vs. pass^k

Two metrics matter for agentic systems, and they tell opposite stories:

- **pass@k**: Likelihood of at least one correct solution in k attempts. Rises as k increases. Important for coding agents where finding the solution matters more than finding it on the first try.
- **pass^k**: Probability that all k trials succeed. Falls as k increases. Matters for customer-facing agents where users expect reliable behavior every time.

At k=1, both metrics are identical. By k=10, pass@k approaches 100% while pass^k approaches 0%. Choose the right metric for your use case.

### Eval Ownership

Anthropic found the most effective model was dedicated evals teams owning core infrastructure, while domain experts and product teams contribute most eval tasks. Non-engineers should contribute: product managers, customer success managers, and salespeople can all contribute eval tasks representing real user scenarios.

### When to Use Approach C

- Building multi-system architectures (agents + RAG + harness)
- Going from ad-hoc testing to systematic evaluation
- Need both regression protection and quality improvement
- Operating across multiple LLM providers where behavior varies
- Want to compound eval value over time

---

## 5. Eval Taxonomy

### Grader Types

#### Code-Based Graders

| Method | Description | Best For |
|--------|-------------|----------|
| String matching | Exact or fuzzy match on output | Classification, entity extraction |
| Binary tests | Does the code run? Do tests pass? | Coding agents |
| Static analysis | Linters, type checkers, security scanners | Code quality |
| Outcome verification | Check the environment state after execution | Tool-using agents |
| Tool call verification | Did the agent call the right tools with right params? | Agent harness routing |
| Transcript analysis | Pattern matching on the interaction log | Multi-turn behavior |

**Strengths**: Fast, cheap, objective, reproducible, easy to debug.
**Weaknesses**: Brittle to valid variations, lack nuance, limited for subjective tasks.

**Rule of thumb**: If a failure can be verified with code, always use a code-based eval (Pragmatic Engineer).

#### Model-Based Graders (LLM-as-Judge)

| Method | Description | Best For |
|--------|-------------|----------|
| Rubric-based scoring | Judge scores against explicit criteria | Quality assessment |
| Natural language assertions | Judge evaluates free-form claims about output | Flexible quality checks |
| Pairwise comparison | Judge picks the better of two outputs | A/B testing, model comparison |
| Reference-based evaluation | Judge compares output to a reference answer | RAG groundedness |
| Multi-judge consensus | Multiple judges vote, majority wins | High-stakes decisions |

**Strengths**: Flexible, scalable, captures nuance, handles open-ended output.
**Weaknesses**: Non-deterministic, more expensive, requires calibration with human graders.

#### Human Graders

| Method | Description | Best For |
|--------|-------------|----------|
| SME review | Domain expert assessment | Gold standard quality |
| Spot-check sampling | Random sample review at cadence | Ongoing quality monitoring |
| Inter-annotator agreement | Multiple humans grade same output | Calibrating LLM judges |
| A/B testing with users | Real users choose preferred outputs | Production validation |

**Strengths**: Gold standard quality, matches expert judgment, used to calibrate model-based graders.
**Weaknesses**: Expensive, slow, doesn't scale.

### Capability vs. Regression Evals

**Capability evals** target tasks the agent struggles with. They should start at a low pass rate, giving teams a hill to climb. They measure: "Can we do this at all?"

**Regression evals** protect against backsliding. They should have a nearly 100% pass rate. A decline signals something is broken. They measure: "Can we still do this reliably?"

As agents mature, capability evals with high pass rates graduate to become regression suites.

### Scoring Strategies

- **Weighted**: Different graders contribute proportionally to the final score
- **Binary (all must pass)**: Every grader must pass for the task to pass — appropriate for non-negotiable constraints
- **Hybrid**: Critical graders are binary gates; quality graders are weighted
- **Partial credit**: An agent that identifies the problem but fails to execute the fix is meaningfully better than one that fails immediately (Anthropic)

---

## 6. Harness Engineering & EDD

### What Is Harness Engineering

In the context of AI agents, harness engineering is the practice of building the orchestration layer — the scaffold that enables a model to act as an agent. The harness processes inputs, manages context, orchestrates tool calls, handles errors, routes between agents, and returns results.

The distinction matters: the **agent harness** is what you engineer. The **evaluation harness** is what tests it. EDD connects them through a feedback loop where the evaluation harness drives the evolution of the agent harness.

### How EDD Specifically Boosts Harness Reliability

#### 1. Evals as the Harness Specification

Without evals, harness requirements live in people's heads or vague documents. With EDD, evals are executable specifications. "Route financial questions to the finance agent" isn't a Jira ticket — it's a set of eval tasks with diverse financial queries, edge cases at category boundaries, and graders that verify routing correctness.

This is especially powerful for multi-agent architectures where the harness must route between specialized agents (deep agents, Claude Code, custom tools). Each routing decision can be eval'd independently.

#### 2. Regression Protection Across Model Upgrades

The harness often works with multiple LLM providers. When any provider releases a new model version, the harness behavior can change unpredictably. An eval suite run against each provider gives immediate signal: did the model upgrade break anything?

Anthropic reports that teams with evals can upgrade to new models in days rather than weeks. Without evals, model upgrades require extensive manual testing.

#### 3. Context Engineering Validation

The harness assembles context for the model — system prompts, retrieved documents, conversation history, tool descriptions. Context engineering is one of the highest-leverage activities in agent development, and evals can directly measure whether the assembled context leads to correct behavior.

Eval patterns for context engineering:
- Does the harness include relevant retrieved documents? (RAG groundedness)
- Does the harness truncate context appropriately when the window fills? (Context overflow handling)
- Does the harness provide the right tool descriptions for the task? (Tool availability)
- Does the harness maintain state correctly across turns? (Multi-turn coherence)

#### 4. Tool Orchestration Testing

Agent harnesses orchestrate tool calls — and the sequence matters. An agent might need to search, then read, then write, in the correct order. Evals can verify:

- Tool selection: Did the agent pick the right tool?
- Tool parameterization: Were the arguments correct?
- Tool sequencing: Were tools called in a valid order?
- Tool error handling: Did the harness recover when a tool failed?
- Outcome verification: Did the tool calls achieve the intended environmental change?

The Anthropic guide recommends against over-specifying exact tool call sequences ("don't grade the path, grade the outcome"), but verifying that the harness supports correct tool orchestration is distinct from verifying the agent chose the optimal path.

#### 5. Multi-Agent Coordination

For architectures using deep agents or multiple specialized agents, the harness coordinates handoffs, context sharing, and result aggregation. Evals at this layer test:

- Delegation correctness: Does the orchestrator delegate to the right sub-agent?
- Context propagation: Does relevant context travel with the delegation?
- Result synthesis: Does the orchestrator correctly combine results from multiple agents?
- Failure isolation: Does one sub-agent's failure cascade to others?

#### 6. Cost and Latency Governance

The harness controls cost (which model to call, how many tokens) and latency (timeouts, retries, caching). Evals can track these as metrics alongside correctness:

- Token usage per task
- Latency per task
- Cost per task
- Error rates and retry counts

Anthropic's eval framework tracks these metrics alongside correctness scores, enabling teams to detect when a change improves quality but degrades cost or latency.

### Harness Eval Architecture for Go + Python

```
+------------------------------------------------------+
|  EVAL RUNNER (Python)                                 |
|  - Orchestrates eval execution                        |
|  - Manages trial isolation (Docker containers)        |
|  - Collects transcripts and outcomes                  |
|  - Runs graders (code-based + LLM-as-judge)           |
|  - Aggregates results, generates reports              |
+------------------------------------------------------+
        |                           |
        v                           v
+-------------------+    +-------------------+
| GO AGENT HARNESS  |    | PYTHON AGENTS     |
| - Routing logic   |    | - Claude Code     |
| - Context assembly|    | - Deep agents     |
| - Tool orchestr.  |    | - RAG pipelines   |
| - Multi-provider  |    | - Custom agents   |
| - State mgmt      |    | - Agent skills    |
+-------------------+    +-------------------+
        |                           |
        v                           v
+------------------------------------------------------+
|  TRACE STORE                                          |
|  - Structured JSON logs                               |
|  - Queryable by task, trial, agent, provider          |
|  - Feeds error analysis and LLM-judge calibration     |
+------------------------------------------------------+
```

The eval runner lives in Python (richer eval tooling ecosystem), while the agent harness can be Go, Python, or both. The trace store is the shared substrate enabling both spec-first and error-analysis workflows.

---

## 7. Tooling Landscape

### Eval Frameworks

Anthropic's recommendation: "quickly pick a framework that fits your workflow, then invest your energy in the evals themselves by iterating on high-quality test cases and graders."

| Framework | Type | Language | Strengths | Best For |
|-----------|------|----------|-----------|----------|
| **Braintrust** | SaaS | Python, TS | Combines offline eval with production observability, experiment tracking, pre-built scorers | Teams wanting eval + monitoring in one platform |
| **Promptfoo** | Open source | CLI, multi-lang | Provider-agnostic, CI/CD friendly, red-teaming support | Multi-provider stacks, security-focused teams |
| **LangSmith** | SaaS | Python, TS | Tracing + evals + dataset management, LangChain integration | LangChain-based stacks |
| **Langfuse** | Open source | Multi-lang | Self-hosted, data residency, tracing + evals | Teams with data sovereignty requirements |
| **Arize Phoenix** | Open source | Python | Tracing, debugging, evals | Teams wanting open-source observability |
| **Harbor** | Open source | Python | Containerized agent environments, standardized task/grader format, benchmark registry | Agent eval isolation |
| **Eval Protocol** | Open source | Python | Decorator-based eval tests, JSONL datasets, MCP integration | Claude Code / MCP workflows |
| **OpenAI Evals** | Open source | Python | Community benchmarks, standardized eval format | OpenAI-centric stacks |

### For Your Stack Specifically

**Go + Python + Multi-Provider + Claude Code + Deep Agents:**

- **Promptfoo** is a strong fit for multi-provider testing — it supports Claude, OpenAI, and custom providers out of the box, with CLI-first workflow that integrates with Go build pipelines.
- **Braintrust** if you want SaaS with production monitoring baked in.
- **Custom eval runner** in Python wrapping your Go harness via HTTP or gRPC — this gives maximum control over how the Go harness is exercised.
- **Eval Protocol** for Claude Code-specific eval workflows with MCP integration.

### Trace and Observability

| Tool | Focus |
|------|-------|
| OpenTelemetry | Distributed tracing standard — works across Go and Python |
| Langfuse | LLM-specific tracing with eval integration |
| Arize Phoenix | Open-source LLM observability |
| Custom JSON logging | Maximum control, lowest overhead |

### LLM-as-Judge Models

For building LLM judges, use the most powerful model you can afford (Husain). The judge model should be at least as capable as the model being evaluated. Claude Opus or GPT-4-class models are standard choices for judging.

---

## 8. Anti-Patterns & Pitfalls

### Pitfall 1: "Vibe-Based Development"

The most common anti-pattern. Changing a prompt, testing a few inputs, shipping if it "looked good to me." The Pragmatic Engineer calls this out directly. It works for demos but not products.

**Fix**: Any of the three approaches. The bar is low — even 20 eval tasks is better than vibes.

### Pitfall 2: Generic Off-the-Shelf Metrics

Using generic "hallucination scores" or "helpfulness" metrics without customization. Hamel Husain warns these "create a false sense of security, leading teams to optimize for scores that don't actually correlate with user satisfaction."

**Fix**: Build domain-specific evals grounded in your actual failure modes. A mental health startup had impressive generic metrics that were completely unactionable.

### Pitfall 3: Over-Specifying Eval Paths

Checking exact tool call sequences instead of outcomes. If the agent finds a creative but valid path to the solution, rigid path-checking fails it unfairly. Anthropic found that Opus 4.5 discovered better solutions that "failed" because graders checked the path, not the outcome.

**Fix**: Grade what the agent produced, not the path it took. Check outcomes (database state, file contents, environment changes), not process.

### Pitfall 4: Ignoring Non-Determinism

Running one trial per task and treating the result as definitive. LLM outputs vary between runs — a single trial is statistically meaningless.

**Fix**: Run multiple trials per task. Use pass@k (at least one success in k tries) or pass^k (all k succeed) depending on your use case.

### Pitfall 5: Shared State Between Trials

Failing to isolate trials from each other. Leftover files, cached data, or git history from previous trials can cause correlated failures or unfair advantages.

**Fix**: Each trial starts from a clean environment. Use containers, temporary directories, or database snapshots.

### Pitfall 6: Eval-Only Pre-Deployment

93% of evaluations in the literature happen before deployment. Only 2% happen post-deployment (EDDOps paper). This means most teams have no idea how their agents behave in the real world.

**Fix**: Close the loop. Production monitoring, trace analysis, user feedback, and A/B testing are not optional extras — they're part of the eval lifecycle.

### Pitfall 7: Saturated Evals

Keeping evals at 100% pass rate indefinitely. They provide no improvement signal and mask real capability differences between model versions.

**Fix**: Monitor for saturation. Replace saturated evals with harder ones. Ensure your eval suite grows in difficulty as your agents improve.

### Pitfall 8: Treating Evals as a One-Time Investment

Writing evals once and never updating them. As the product evolves, evals that were relevant become stale, and new failure modes emerge.

**Fix**: Eval suites need ongoing attention and clear ownership. Production failures should automatically feed back into the eval suite.

### Pitfall 9: Tool Fixation

Believing another evaluation tool or metric will solve fundamental process problems. Eugene Yan: "An LLM-as-Judge Won't Save The Product — Fixing Your Process Will."

**Fix**: Evals are practices rooted in the scientific method, not tools or artifacts. No tool compensates for not looking at the data.

### Pitfall 10: Removing Human Oversight

Over-relying on automated evaluators without continued human annotation and review. Automated evals can't compensate for neglect.

**Fix**: Periodically sample and annotate data. Re-calibrate LLM judges against human judgment. Never stop looking at traces.

---

## 9. Sources

All sources were fetched and verified on April 22, 2026.

### Primary Sources (Full Content Verified)

1. **[Demystifying Evals for AI Agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)** — Anthropic Engineering Blog, January 2026. Comprehensive guide to building evals for AI agents. Covers eval lifecycle, grader types, non-determinism handling, Claude Code's internal eval practices, and the Swiss Cheese Model for multi-layered evaluation.

2. **[Your AI Product Needs Evals](https://hamel.dev/blog/posts/evals/)** — Hamel Husain, March 2024. Argues that eval systems are the #1 predictor of AI product success. Introduces the three-level evaluation framework (unit tests, human & model eval, A/B testing) with the Rechat case study.

3. **[LLM Evals: Everything You Need to Know](https://hamel.dev/blog/posts/evals-faq/)** — Hamel Husain & Shreya Shankar, January 2026. Comprehensive FAQ on LLM evals. Contains the contrarian argument against eval-driven development and the case for error-analysis-first approaches.

4. **[EDDOps: Evaluation-Driven Development and Operations of LLM Agents](https://arxiv.org/abs/2411.13768)** — Xia, Lu, Zhu, Xing, Zhao, Zhang. arXiv, November 2024 (v3 November 2025). Academic paper presenting a process model and reference architecture for evaluation-driven development and operations. Key finding: 93% of evaluations are pre-deployment, only 2% post-deployment.

5. **[An LLM-as-Judge Won't Save The Product — Fixing Your Process Will](https://eugeneyan.com/writing/eval-process/)** — Eugene Yan, April 2025. Frames EDD as the scientific method applied to AI products: observe, annotate, hypothesize, experiment, measure, iterate.

6. **[Eval-Driven Development with Claude Code](https://fireworks.ai/blog/eval-driven-development-with-claude-code)** — Fireworks AI, 2025. Practical TDD-to-EDD workflow using Claude Code with MCP servers. Shows how to generate and expand eval tasks using AI assistance.

7. **[A Pragmatic Guide to LLM Evals for Devs](https://newsletter.pragmaticengineer.com/p/evals)** — Pragmatic Engineer Newsletter (with Hamel Husain), 2025. Developer-focused guide to moving past vibe checks. Introduces the Three Gulfs, open/axial coding for error analysis, and the PASS/FAIL recommendation for LLM judges.

### Secondary Sources (Search-Verified)

8. **[Eval-Driven System Design: From Prototype to Production](https://developers.openai.com/cookbook/examples/partners/eval_driven_system_design/receipt_inspection)** — OpenAI Cookbook. Practical cookbook example of eval-driven system design.

9. **[Evaluation-Driven Development Workflows: Best Practices and Real-World Scenarios](https://www.databricks.com/dataaisummit/session/evaluation-driven-development-workflows-best-practices-and-real-world)** — Data + AI Summit 2025, Databricks.

10. **[Survey on Evaluation of LLM-based Agents](https://arxiv.org/abs/2503.16416)** — arXiv, March 2025. First comprehensive survey of evaluation methodologies for LLM-based agents across four dimensions: fundamental capabilities, application-specific benchmarks, generalist agent benchmarks, and evaluation frameworks.

11. **[Selecting The Right AI Evals Tool](https://hamel.dev/blog/posts/eval-tools/)** — Hamel Husain. Guide to choosing eval tooling.

12. **[AI Evals for Engineers & PMs](https://maven.com/parlance-labs/evals)** — Hamel Husain & Shreya Shankar, Maven course. The most popular course on AI evals, trained 2,000+ engineers including teams at OpenAI and Anthropic.

13. **[Why AI Evals Are the Hottest New Skill for Product Builders](https://www.lennysnewsletter.com/p/why-ai-evals-are-the-hottest-new-skill)** — Lenny's Newsletter, featuring Hamel Husain & Shreya Shankar.

14. **[Awesome LLM Evaluation](https://alopatenko.github.io/LLMEvaluation/)** — Curated directory of LLM evaluation methods, benchmarks, and tools.

15. **[Claude Code Best Practices](https://code.claude.com/docs/en/best-practices)** — Anthropic official documentation on Claude Code usage patterns.
