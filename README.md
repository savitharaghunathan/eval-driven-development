# Eval-Driven Development Plugin

A Claude Code plugin that defines evals from an approved design spec before writing implementation plans. Works for any AI system — coding agents, customer support agents, research agents, RAG systems, content generation, data pipelines, or any LLM-powered application.

## Where It Fits

```
Brainstorming (approved spec)
  → /eval-driven-development:define-evals   ← THIS PLUGIN
    → Writing Plans (implementation plan against evals)
      → Implementation (build to pass evals)
```

The plugin bridges the gap between "what are we building" and "how do we know it works." Evals become the definition of done.

## What It Does

Takes an approved design spec and produces:

1. Eval tasks derived from every requirement in the spec
2. Grader selection for each task (code-based, LLM-as-judge, or outcome verification)
3. Balanced problem sets (positive, negative, and boundary cases)
4. Pass criteria (thresholds, pass@k vs pass^k, regression vs capability classification)
5. Non-functional metrics (cost, safety, robustness, governance)
6. Memory-specific metrics for agents with persistent state (retrieval quality, forgetting, cross-session coherence)
7. Harness-specific metrics for orchestration infrastructure (routing accuracy, compaction fidelity, fault recovery)
8. An eval plan file that the writing-plans skill consumes as acceptance criteria

## Installation

### Local Testing

```bash
claude --plugin-dir /path/to/eval-driven-development
```

### From Git

Add to your project's `.claude/settings.json` or install via marketplace once published.

## Usage

```
/eval-driven-development:define-evals
```

Or let Claude invoke it automatically when transitioning from an approved spec to implementation planning.

## Plugin Structure

```
eval-driven-development/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   └── define-evals/
│       ├── SKILL.md                    # Core skill instructions
│       └── references/
│           ├── eval-taxonomy.md        # Grader types and tradeoffs
│           ├── eval-guide.md           # Task writing rules and dimensions
│           ├── eval-plan-template.md   # Output format for eval plans
│           └── sources.md              # All verified citations
├── docs/
│   └── superpowers/specs/
│       └── 2026-04-22-eval-driven-development-design.md  # Full research synthesis
├── LICENSE
└── README.md
```

## Research Basis

Grounded in 25 verified sources including:

- [Demystifying Evals for AI Agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents) — Anthropic, Jan 2026
- [Your AI Product Needs Evals](https://hamel.dev/blog/posts/evals/) — Hamel Husain, Mar 2024
- [EDDOps: Evaluation-Driven Development and Operations](https://arxiv.org/abs/2411.13768) — Xia et al., arXiv
- [tau-bench](https://arxiv.org/abs/2406.12045) — Yao, Shinn et al. Agent reliability benchmark
- [An LLM-as-Judge Won't Save The Product](https://eugeneyan.com/writing/eval-process/) — Eugene Yan, Apr 2025
- [Survey on Evaluation of LLM-based Agents](https://arxiv.org/abs/2503.16416) — arXiv, Mar 2025
- [Harness Engineering](https://martinfowler.com/articles/harness-engineering.html) — Boeckeler, Apr 2026
- [Memory for Autonomous LLM Agents](https://arxiv.org/abs/2603.07670) — Du, arXiv, Mar 2026

Full source list with descriptions: `skills/define-evals/references/sources.md`

## License

Apache-2.0
