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
claude --plugin-dir /path/to/eval-driven-development/plugin
```

### From Git

```bash
git clone https://github.com/savitharaghunathan/eval-driven-development.git
claude --plugin-dir /path/to/eval-driven-development/plugin
```

### From Release

Download the latest zip from [Releases](https://github.com/savitharaghunathan/eval-driven-development/releases), extract it, and point at the extracted directory:

```bash
unzip eval-driven-development-*.zip -d eval-driven-development-plugin
claude --plugin-dir /path/to/eval-driven-development-plugin
```

## Usage

```
/eval-driven-development:define-evals
```

Or let Claude invoke it automatically when transitioning from an approved spec to implementation planning.

## Development

The repo separates the installable plugin from development artifacts:

- `plugin/` — The installable plugin. This is what users point `--plugin-dir` at.
- `docs/` — Research synthesis and design specs. Reference material, not shipped with the plugin.
- `.github/workflows/` — CI for validation and release tagging.

## Repository Structure

```
eval-driven-development/
├── plugin/                            # Installable plugin (point --plugin-dir here)
│   ├── .claude-plugin/
│   │   └── plugin.json
│   └── skills/
│       └── define-evals/
│           ├── SKILL.md                    # Core skill instructions
│           └── references/
│               ├── eval-taxonomy.md        # Grader types and tradeoffs
│               ├── eval-guide.md           # Task writing rules and dimensions
│               ├── eval-plan-template.md   # Output format for eval plans
│               ├── example-eval-plan.md   # Filled-in example eval plan
│               └── sources.md              # All verified citations
├── docs/                              # Research and design specs (not shipped with plugin)
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

Full source list with descriptions: `plugin/skills/define-evals/references/sources.md`

## Contributing

### Lint locally

```bash
npx markdownlint-cli2 "plugin/skills/**/*.md" --config .github/markdownlint-config.yaml
```

The CI uses three disabled rules: MD013 (line length), MD033 (inline HTML, needed for `<HARD-GATE>` tags), MD041 (first-line heading, skill files use frontmatter).

### Test the plugin

```bash
claude --plugin-dir ./plugin
```

Then invoke the skill with `/eval-driven-development:define-evals` and verify it loads correctly.

## License

Apache-2.0
