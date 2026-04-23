# Plugin Publishing and CI Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Restructure the repo so the installable plugin lives under `plugin/`, add CI for validation and release tagging, and update the README.

**Architecture:** Move `.claude-plugin/` and `skills/` into a `plugin/` subdirectory. Add two GitHub Actions workflows at `.github/workflows/`. The plugin itself is unchanged — only its location in the repo moves.

**Tech Stack:** GitHub Actions, markdownlint-cli2 (npm), lychee (Rust link checker)

---

## File Structure

| File | Action | Responsibility |
|------|--------|----------------|
| `plugin/.claude-plugin/plugin.json` | Move from `.claude-plugin/plugin.json` | Plugin manifest |
| `plugin/skills/define-evals/SKILL.md` | Move from `skills/define-evals/SKILL.md` | Core skill instructions |
| `plugin/skills/define-evals/references/*` | Move from `skills/define-evals/references/*` | Reference files (4 files) |
| `.gitignore` | Create | Exclude `.claude/`, `.DS_Store`, `*.swp` |
| `.github/workflows/validate.yml` | Create | PR/push validation pipeline |
| `.github/workflows/release.yml` | Create | Auto-tag releases on version bump |
| `README.md` | Modify | Update install path, structure diagram, add dev section |

---

### Task 1: Add .gitignore

**Files:**
- Create: `.gitignore`

- [ ] **Step 1: Create .gitignore**

```
.claude/
.DS_Store
*.swp
```

- [ ] **Step 2: Verify .claude/settings.local.json is now ignored**

Run: `git status`
Expected: `.claude/settings.local.json` no longer shows as untracked. The `.gitignore` file shows as a new untracked file.

- [ ] **Step 3: Remove .claude/settings.local.json from tracking if it was committed**

Run: `git ls-files .claude/settings.local.json`
If output is non-empty, run: `git rm --cached .claude/settings.local.json`
If output is empty, skip this step (file was never tracked).

- [ ] **Step 4: Commit**

```bash
git add .gitignore
git commit -s -m "Add .gitignore to exclude .claude/, .DS_Store, and swap files"
```

---

### Task 2: Move plugin files into plugin/ subdirectory

**Files:**
- Move: `.claude-plugin/plugin.json` → `plugin/.claude-plugin/plugin.json`
- Move: `skills/define-evals/SKILL.md` → `plugin/skills/define-evals/SKILL.md`
- Move: `skills/define-evals/references/eval-guide.md` → `plugin/skills/define-evals/references/eval-guide.md`
- Move: `skills/define-evals/references/eval-taxonomy.md` → `plugin/skills/define-evals/references/eval-taxonomy.md`
- Move: `skills/define-evals/references/eval-plan-template.md` → `plugin/skills/define-evals/references/eval-plan-template.md`
- Move: `skills/define-evals/references/sources.md` → `plugin/skills/define-evals/references/sources.md`

- [ ] **Step 1: Create the plugin directory structure**

```bash
mkdir -p plugin/.claude-plugin
mkdir -p plugin/skills/define-evals/references
```

- [ ] **Step 2: Move plugin manifest**

```bash
git mv .claude-plugin/plugin.json plugin/.claude-plugin/plugin.json
```

- [ ] **Step 3: Move skill and reference files**

```bash
git mv skills/define-evals/SKILL.md plugin/skills/define-evals/SKILL.md
git mv skills/define-evals/references/eval-guide.md plugin/skills/define-evals/references/eval-guide.md
git mv skills/define-evals/references/eval-taxonomy.md plugin/skills/define-evals/references/eval-taxonomy.md
git mv skills/define-evals/references/eval-plan-template.md plugin/skills/define-evals/references/eval-plan-template.md
git mv skills/define-evals/references/sources.md plugin/skills/define-evals/references/sources.md
```

- [ ] **Step 4: Remove empty original directories**

```bash
rmdir skills/define-evals/references skills/define-evals skills .claude-plugin
```

If `rmdir` fails because directories are already gone (git mv cleaned them up), that's fine — skip.

- [ ] **Step 5: Verify the new structure**

```bash
find plugin -type f | sort
```

Expected output:
```
plugin/.claude-plugin/plugin.json
plugin/skills/define-evals/SKILL.md
plugin/skills/define-evals/references/eval-guide.md
plugin/skills/define-evals/references/eval-plan-template.md
plugin/skills/define-evals/references/eval-taxonomy.md
plugin/skills/define-evals/references/sources.md
```

- [ ] **Step 6: Verify old directories are gone**

```bash
ls .claude-plugin 2>&1 || echo "Gone"
ls skills 2>&1 || echo "Gone"
```

Expected: Both print "Gone" (or "No such file or directory").

- [ ] **Step 7: Commit**

```bash
git add plugin/
git commit -s -m "Move plugin files into plugin/ subdirectory

Separates installable plugin from docs and CI. Users install with:
  --plugin-dir /path/to/eval-driven-development/plugin"
```

---

### Task 3: Update README

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Update install instructions**

Change the Local Testing section from:

```bash
claude --plugin-dir /path/to/eval-driven-development
```

To:

```bash
claude --plugin-dir /path/to/eval-driven-development/plugin
```

- [ ] **Step 2: Update the Plugin Structure diagram**

Replace the entire Plugin Structure section with:

````markdown
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
│               └── sources.md              # All verified citations
├── docs/                              # Research and design specs (not shipped with plugin)
│   └── superpowers/specs/
├── .github/workflows/                 # CI: validation and release
├── LICENSE
└── README.md
```
````

- [ ] **Step 3: Update the From Git section**

Replace:

```markdown
### From Git

Add to your project's `.claude/settings.json` or install via marketplace once published.
```

With:

```markdown
### From Git

```bash
git clone https://github.com/savitharaghunathan/eval-driven-development.git
claude --plugin-dir /path/to/eval-driven-development/plugin
```

- [ ] **Step 4: Add Development section before Research Basis**

Add after the Usage section:

```markdown
## Development

The repo separates the installable plugin from development artifacts:

- `plugin/` — The installable plugin. This is what users point `--plugin-dir` at.
- `docs/` — Research synthesis and design specs. Reference material, not shipped with the plugin.
- `.github/workflows/` — CI for validation and release tagging.
```

- [ ] **Step 5: Update source file reference path**

Change:

```
Full source list with descriptions: `skills/define-evals/references/sources.md`
```

To:

```
Full source list with descriptions: `plugin/skills/define-evals/references/sources.md`
```

- [ ] **Step 6: Commit**

```bash
git add README.md
git commit -s -m "Update README for plugin/ subdirectory structure"
```

---

### Task 4: Create validate workflow

**Files:**
- Create: `.github/workflows/validate.yml`

- [ ] **Step 1: Create workflows directory**

```bash
mkdir -p .github/workflows
```

- [ ] **Step 2: Create validate.yml**

```yaml
name: Validate Plugin

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  structure:
    name: Plugin Structure
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Check plugin.json exists and is valid
        run: |
          test -f plugin/.claude-plugin/plugin.json
          python3 -c "
          import json, sys
          with open('plugin/.claude-plugin/plugin.json') as f:
              data = json.load(f)
          for field in ['name', 'description', 'version']:
              if field not in data:
                  print(f'Missing required field: {field}')
                  sys.exit(1)
          print('plugin.json is valid')
          "

      - name: Check SKILL.md exists and has frontmatter
        run: |
          test -f plugin/skills/define-evals/SKILL.md
          head -1 plugin/skills/define-evals/SKILL.md | grep -q '^---$'
          grep -q '^description:' plugin/skills/define-evals/SKILL.md
          echo "SKILL.md has valid frontmatter"

      - name: Check reference files exist
        run: |
          test -f plugin/skills/define-evals/references/eval-guide.md
          test -f plugin/skills/define-evals/references/eval-taxonomy.md
          test -f plugin/skills/define-evals/references/eval-plan-template.md
          test -f plugin/skills/define-evals/references/sources.md
          echo "All reference files present"

  lint:
    name: Markdown Lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install markdownlint-cli2
        run: npm install -g markdownlint-cli2

      - name: Create config
        run: |
          cat > .markdownlint-cli2.yaml << 'EOF'
          config:
            MD013: false        # Allow long lines (prose-heavy skill files)
            MD033: false        # Allow inline HTML
            MD041: false        # First line doesn't need to be a heading
          globs:
            - "plugin/skills/**/*.md"
          EOF

      - name: Run markdownlint
        run: markdownlint-cli2

  links:
    name: Check Links
    runs-on: ubuntu-latest
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4

      - name: Install lychee
        uses: lycheeverse/lychee-action@v2
        with:
          args: >-
            --timeout 30
            --retry-wait-time 5
            --max-retries 3
            --accept 200,403,429
            plugin/skills/define-evals/references/sources.md
          fail: true
```

- [ ] **Step 3: Verify YAML is valid**

```bash
python3 -c "import yaml; yaml.safe_load(open('.github/workflows/validate.yml'))"
```

Expected: No output (valid YAML).

- [ ] **Step 4: Commit**

```bash
git add .github/workflows/validate.yml
git commit -s -m "Add validate workflow for plugin structure, markdown lint, and link checks"
```

---

### Task 5: Create release workflow

**Files:**
- Create: `.github/workflows/release.yml`

- [ ] **Step 1: Create release.yml**

```yaml
name: Release

on:
  push:
    branches: [main]
    paths:
      - 'plugin/.claude-plugin/plugin.json'

jobs:
  release:
    name: Tag and Release
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Read version from plugin.json
        id: version
        run: |
          VERSION=$(python3 -c "import json; print(json.load(open('plugin/.claude-plugin/plugin.json'))['version'])")
          echo "version=$VERSION" >> "$GITHUB_OUTPUT"
          echo "tag=v$VERSION" >> "$GITHUB_OUTPUT"
          echo "Version: $VERSION"

      - name: Check if tag exists
        id: check
        run: |
          if git rev-parse "${{ steps.version.outputs.tag }}" >/dev/null 2>&1; then
            echo "exists=true" >> "$GITHUB_OUTPUT"
            echo "Tag ${{ steps.version.outputs.tag }} already exists, skipping release"
          else
            echo "exists=false" >> "$GITHUB_OUTPUT"
            echo "Tag ${{ steps.version.outputs.tag }} does not exist, creating release"
          fi

      - name: Create release
        if: steps.check.outputs.exists == 'false'
        uses: softprops/action-gh-release@v2
        with:
          tag_name: ${{ steps.version.outputs.tag }}
          name: ${{ steps.version.outputs.tag }}
          generate_release_notes: true
```

- [ ] **Step 2: Verify YAML is valid**

```bash
python3 -c "import yaml; yaml.safe_load(open('.github/workflows/release.yml'))"
```

Expected: No output (valid YAML).

- [ ] **Step 3: Commit**

```bash
git add .github/workflows/release.yml
git commit -s -m "Add release workflow for auto-tagging on version bump"
```

---

### Task 6: Final verification

- [ ] **Step 1: Verify full repo structure**

```bash
find . -type f -not -path '*/.git/*' -not -path './.claude/*' -not -name '.DS_Store' | sort
```

Expected output:
```
./.github/workflows/release.yml
./.github/workflows/validate.yml
./.gitignore
./docs/superpowers/plans/2026-04-23-plugin-publishing.md
./docs/superpowers/specs/2026-04-22-eval-driven-development-design.md
./docs/superpowers/specs/2026-04-23-plugin-publishing-design.md
./LICENSE
./plugin/.claude-plugin/plugin.json
./plugin/skills/define-evals/SKILL.md
./plugin/skills/define-evals/references/eval-guide.md
./plugin/skills/define-evals/references/eval-plan-template.md
./plugin/skills/define-evals/references/eval-taxonomy.md
./plugin/skills/define-evals/references/sources.md
./README.md
```

- [ ] **Step 2: Verify plugin works from new location**

```bash
claude --plugin-dir ./plugin --print-plugins 2>&1 || echo "Verify manually: claude --plugin-dir ./plugin"
```

If `--print-plugins` isn't a valid flag, test manually by starting a new Claude Code session with `claude --plugin-dir ./plugin` and checking that `/eval-driven-development:define-evals` appears as an available skill.

- [ ] **Step 3: Verify old plugin directories are gone**

```bash
test -d .claude-plugin && echo "FAIL: .claude-plugin still exists" || echo "OK: .claude-plugin removed"
test -d skills && echo "FAIL: skills still exists" || echo "OK: skills removed"
```

Expected: Both print "OK".
