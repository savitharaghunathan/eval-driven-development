# Plugin Publishing and CI Design

**Date**: 2026-04-23
**Status**: Draft

## Summary

Restructure the eval-driven-development repo so the installable plugin lives under `plugin/`, keeping docs and CI at the repo root. Add GitHub Actions for validation and release tagging.

## Repo Structure

```
eval-driven-development/
├── plugin/                              # Installable plugin root
│   ├── .claude-plugin/
│   │   └── plugin.json
│   └── skills/
│       └── define-evals/
│           ├── SKILL.md
│           └── references/
│               ├── eval-guide.md
│               ├── eval-taxonomy.md
│               ├── eval-plan-template.md
│               └── sources.md
├── docs/                                # Research and specs (not shipped with plugin)
│   └── superpowers/specs/
│       ├── 2026-04-22-eval-driven-development-design.md
│       └── 2026-04-23-plugin-publishing-design.md
├── .github/
│   └── workflows/
│       ├── validate.yml
│       └── release.yml
├── .gitignore
├── README.md
└── LICENSE
```

Users install with: `--plugin-dir /path/to/eval-driven-development/plugin`

## .gitignore

```
.claude/
.DS_Store
*.swp
```

## CI Pipeline: Validate (`validate.yml`)

Triggers on every PR and push to `main`.

### 1. Plugin Structure Check

Verify required files exist and are well-formed:

- `plugin/.claude-plugin/plugin.json` exists and is valid JSON with `name`, `description`, `version` fields
- `plugin/skills/define-evals/SKILL.md` exists and has YAML frontmatter with `description` field
- All four reference files exist:
  - `plugin/skills/define-evals/references/eval-guide.md`
  - `plugin/skills/define-evals/references/eval-taxonomy.md`
  - `plugin/skills/define-evals/references/eval-plan-template.md`
  - `plugin/skills/define-evals/references/sources.md`

### 2. Markdown Lint

Run `markdownlint-cli2` on all `.md` files under `plugin/skills/`.

Config: check for broken headings, trailing whitespace, inconsistent list markers. Allow long lines (skill files are prose-heavy).

### 3. Dead Link Check

Run `lychee` on `plugin/skills/define-evals/references/sources.md` to verify all URLs are reachable.

- Only runs on push to `main` (not on PRs, to avoid rate-limiting)
- Generous timeout for slow sources (arXiv)
- Retries on transient failures

## CI Pipeline: Release (`release.yml`)

Triggers on push to `main`.

### Steps

1. Read `version` from `plugin/.claude-plugin/plugin.json`
2. Check if git tag `v<version>` already exists — if yes, skip
3. Create GitHub release with tag `v<version>` and auto-generated release notes from commits since last tag

### Release Process

To cut a new release: bump `version` in `plugin.json`, merge to `main`. The pipeline creates the tag and release automatically.

No build artifacts needed — users clone the repo and point `--plugin-dir` at `plugin/`.

## README Updates

- Install instructions updated to point at `plugin/` subdirectory
- Plugin structure diagram reflects new layout
- Development section added explaining `docs/` vs `plugin/` separation
- `docs/` removed from plugin structure diagram
