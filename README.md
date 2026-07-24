# Eval-Driven Development

An [Agent Skill](https://agentskills.io) that defines evals from an approved design spec before writing implementation plans. Works with any skills-compatible agent and for any AI system — coding agents, customer support agents, research agents, RAG systems, content generation, data pipelines, or any LLM-powered application.

## Where It Fits

```
Brainstorming (approved spec)
  → define-evals                              ← THIS SKILL
    → Writing Plans (implementation plan against evals)
      → Implementation (build to pass evals)
```

The skill bridges the gap between "what are we building" and "how do we know it works." Evals become the definition of done.

## What It Does

Takes an approved design spec and produces:

1. Eval tasks derived from every requirement in the spec
2. Grader selection for each task (code-based, LLM-as-judge, outcome verification, or statistical validation)
3. Balanced problem sets (positive, negative, and boundary cases)
4. Pass criteria (thresholds, pass@k vs pass^k, regression vs capability classification)
5. Per-archetype metrics (dimensions and graders tailored to the detected system type)
6. Universal metrics (cost, safety, robustness, governance, observability)
7. An eval plan file that serves as acceptance criteria for implementation

## Installation

Using [`skills`](https://skills.sh), the CLI for the Agent Skills spec:

```bash
npx skills add savitharaghunathan/eval-driven-development --skill define-evals
```

This detects your installed agent(s) automatically. To target a specific agent non-interactively, pass `--agent` (one of `claude-code`, `cursor`, `codex`, `gemini-cli`, `github-copilot`):

```bash
npx skills add savitharaghunathan/eval-driven-development --skill define-evals --agent claude-code --yes
```

For agents not covered by the `skills` CLI, see the [Agent Skills client showcase](https://agentskills.io/clients) for the correct skill directory, then copy `define-evals/` into it directly.

### From Release

Download the latest zip from [Releases](https://github.com/savitharaghunathan/eval-driven-development/releases), extract it, and copy the `define-evals/` directory into your agent's skill directory.

## Usage

Ask your agent to define evals for an approved spec, or use trigger phrases like:

- "define evals for this spec"
- "write evals for this spec"
- "create an eval plan"

The skill activates automatically when transitioning from an approved design spec to implementation planning.

## Repository Structure

```
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

## Research Basis

Grounded in 27 verified sources including:

- [Demystifying Evals for AI Agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) — Anthropic, Jan 2026
- [Your AI Product Needs Evals](https://hamel.dev/blog/posts/evals/) — Hamel Husain, Mar 2024
- [EDDOps: Evaluation-Driven Development and Operations](https://arxiv.org/abs/2411.13768) — Xia et al., arXiv
- [tau-bench](https://arxiv.org/abs/2406.12045) — Yao, Shinn et al. Agent reliability benchmark
- [An LLM-as-Judge Won't Save The Product](https://eugeneyan.com/writing/eval-process/) — Eugene Yan, Apr 2025
- [Survey on Evaluation of LLM-based Agents](https://arxiv.org/abs/2503.16416) — arXiv, Mar 2025
- [Harness Engineering](https://martinfowler.com/articles/harness-engineering.html) — Boeckeler, Apr 2026
- [Memory for Autonomous LLM Agents](https://arxiv.org/abs/2603.07670) — Du, arXiv, Mar 2026

Full source list with descriptions: `define-evals/references/sources.md`

## Contributing

### Lint locally

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

The CI uses three disabled rules: MD013 (line length), MD033 (inline HTML, needed for `<HARD-GATE>` tags), MD041 (first-line heading, skill files use frontmatter).

### Test the skill

Copy the `define-evals/` directory into your agent's skill directory and verify the skill activates when you mention eval planning.

## License

Apache-2.0
