# Claude Code Project Setup Guide
_Run this once per new project, before your first Claude Code session._

---

## What You're Setting Up

| Asset | Location | How |
|-------|----------|-----|
| CLAUDE.md | project root | Copy template, fill 4 fields |
| 7 commands | `.claude/commands/` | Copy from `../commands/` |
| Hook + config | `.claude/hooks/` + `.claude/settings.json` | Ask Claude Code to configure |
| 2 starter templates | `docs/` | Copy from `../templates/` |
| Pre-built docs | `docs/` | Copy from your Claude.ai session |

---

## Prerequisites

Before starting, confirm you have:

- [ ] Claude Code installed (`claude --version` works in terminal)
- [ ] Node.js installed (`node --version` works in terminal)
- [ ] A project folder created and `git init` run
- [ ] These files ready one level up (`../`) from your project folder:
  - `commands/` folder — contains 7 .md command files
  - `templates/` folder — contains OPEN_QUESTIONS.md and MEMORY.md
  - `CLAUDE.md` — project brain template
- [ ] These docs generated in Claude.ai (via `project-scoping` + `solution-design` skills):
  - `PRD.md`, `ROADMAP.md`, `ASSUMPTIONS.md`, `ARCHITECTURE.md`, `SCENARIOS.md`

---

## Step 1 — Create the Folder Structure

```bash
mkdir -p docs/briefings
mkdir -p docs/archive
mkdir -p .claude/commands
mkdir -p .claude/hooks/scripts
mkdir -p .claude/agents
mkdir -p src/shared
mkdir -p tests/unit tests/integration tests/e2e
```

This matches the canonical structure defined in CLAUDE.md. Do not rename
or reorganize — agents navigate by exact path.

---

## Step 2 — Install the CLAUDE.md Template

Copy `CLAUDE.md` from one level up into your project root:

```bash
cp ../templates/CLAUDE.md ./CLAUDE.md
```

Then fill in the four fields at the top:

```markdown
- **Name**: [Your project name]
- **Phase**: 0 — Setup
- **Stack**: [Your stack — e.g. Next.js · Node.js · PostgreSQL]
- **Repo root**: [Absolute path — e.g. /Users/you/projects/myapp]
```

Leave everything else untouched. Stack conventions get filled in after Phase 0
once the Orchestrator confirms the stack from your ARCHITECTURE.md.

---

## Step 3 — Install the Commands

Copy all seven command files from one level up:

```bash
cp ../commands/*.md .claude/commands/
```

Verify all seven are present:
```bash
ls .claude/commands/
# gen-briefing.md   handoff.md      log-change.md    phase-complete.md
# sync-docs.md      update-claude.md  update-modules.md
```

These are now available as slash commands inside any Claude Code session:
`/gen-briefing`, `/handoff`, `/log-change`, `/phase-complete`,
`/sync-docs`, `/update-claude`, `/update-modules`

---

## Step 4 — Configure the Hook (delegate to Claude Code)

Open Claude Code in your project directory and paste this prompt:

```
I need you to set up a PostToolUse hook that automatically logs code file
changes to docs/CHANGELOG.md. Please:

1. Create the hook script at:
   .claude/hooks/scripts/changelog-append.js

   The script should:
   - Read the written file path from stdin (JSON input with field "path"
     or "file_path")
   - Only process code files matching: src/**/*.ts, src/**/*.tsx,
     src/**/*.js, src/**/*.jsx, tests/**/*.ts, tests/**/*.test.ts
   - Skip *.md, *.json, *.env, and anything inside .claude/
   - Append a row to docs/CHANGELOG.md in this format:
     | [date] | [filepath] | agent | [created/modified] [filename] | auto-logged |
   - If docs/CHANGELOG.md doesn't exist, create it with a standard
     header row first
   - Infer change type as "created" if the file didn't exist before the
     write, "modified" if it did

2. Register the hook in .claude/settings.json:
   - Type: PostToolUse
   - Matcher: Write tool
   - Command: node .claude/hooks/scripts/changelog-append.js

3. Make the script executable with chmod +x

4. Test it: write a dummy file to src/test-hook.ts, confirm a row appears
   in docs/CHANGELOG.md, then delete src/test-hook.ts.

Confirm when complete with: "Hook installed and verified."
```

Wait for "Hook installed and verified." before proceeding to Step 5.

---

## Step 5 — Add the Starter Templates

Copy the starter templates from one level up into `docs/`:

```bash
cp ../templates/OPEN_QUESTIONS.md docs/OPEN_QUESTIONS.md
cp ../templates/MEMORY.md         docs/MEMORY.md
```

Replace `[Project Name]` in both files:

```bash
# macOS
sed -i '' 's/\[Project Name\]/YourProjectName/g' docs/OPEN_QUESTIONS.md docs/MEMORY.md

# Linux
sed -i 's/\[Project Name\]/YourProjectName/g' docs/OPEN_QUESTIONS.md docs/MEMORY.md
```

---

## Step 6 — Copy Your Pre-Built Docs

```bash
cp ../PRD.md           docs/PRD.md
cp ../ASSUMPTIONS.md   docs/ASSUMPTIONS.md
cp ../ARCHITECTURE.md  docs/ARCHITECTURE.md
cp ../SCENARIOS.md     docs/SCENARIOS.md

# ROADMAP.md stays at project root — NOT inside docs/
# Agents read docs/ freely; root-only placement enforces the off-limits rule
cp ../ROADMAP.md       ./ROADMAP.md
```

---

## Step 7 — Verify Before Opening Phase 0

Run each check and confirm the expected output before proceeding:

```bash
# CLAUDE.md at root — confirm your project name appears
head -5 CLAUDE.md

# All 7 commands present
ls .claude/commands/ | wc -l
# → 7

ls .claude/commands/
# → gen-briefing.md  handoff.md  log-change.md  phase-complete.md
# → sync-docs.md  update-claude.md  update-modules.md

# Hook script installed
ls .claude/hooks/scripts/changelog-append.js
# → file exists

# Hook registered
grep -q "PostToolUse" .claude/settings.json && echo "✅ Hook registered"

# Starter templates in docs/
ls docs/OPEN_QUESTIONS.md docs/MEMORY.md
# → both present

# Pre-built scoping docs in docs/
ls docs/PRD.md docs/ASSUMPTIONS.md docs/ARCHITECTURE.md docs/SCENARIOS.md
# → all four present

# ROADMAP.md at root only — must NOT be inside docs/
ls ROADMAP.md && echo "✅ root" && ! ls docs/ROADMAP.md 2>/dev/null && echo "✅ not in docs/"

# Archive folder ready
ls -d docs/archive/
# → exists (empty is fine — /phase-complete populates it)
```

---

## Step 8 — Run Phase 0

Open Claude Code in your project root and paste the **Phase 0 Starter Prompt**
(see `phase-0-starter-prompt.md`).

Phase 0 is fully autonomous except for one gate: reviewing and locking SCHEMA.md.
Everything else — MODULES.md, TESTS.md, briefings, open questions seeding — runs
to completion without your input.

---

## Reference — All Commands

| Command | Who calls | When |
|---------|----------|------|
| `/gen-briefing [module]` | Orchestrator | Before launching each builder agent |
| `/update-modules` | Builder agent | When module status or interface changes |
| `/sync-docs` | Auto (via `/update-modules`) + `/phase-complete` + you manually | When all modules complete; before phase archive; after [SCHEMA CHANGE] resolved |
| `/log-change` | Any agent | After any meaningful doc file change |
| `/update-claude` | You (owner only) | When a new project-wide rule is needed |
| `/phase-complete [N]` | You (owner only) | After all modules complete, `/sync-docs` clean, P0 tests passing |
| `/handoff` | You | Before every `/clear` — no exceptions |

---

## Reference — File Ownership

| File | Who writes | Who reads |
|------|-----------|-----------|
| `CLAUDE.md` | You via `/update-claude` | All agents (auto-loaded every session) |
| `docs/SCHEMA.md` | Orchestrator (Phase 0) → locked by you | All agents (read-only after lock) |
| `docs/MODULES.md` | Agents via `/update-modules` | All agents |
| `docs/OPEN_QUESTIONS.md` | Any agent (append) → you (resolve) | Orchestrator, you |
| `docs/MEMORY.md` | `/handoff` only | Orchestrator (session start) |
| `docs/CHANGELOG.md` | Hook (code files) + `/log-change` (doc files) | You, Orchestrator |
| `docs/PHASE_TRANSITION.md` | `/phase-complete` | You (take to Claude.ai for next scoping) |
| `docs/briefings/*.md` | Orchestrator via `/gen-briefing` | Assigned builder agent only |
| `docs/archive/phase-[N]/` | `/phase-complete` | Humans auditing past decisions — never agents |
| `ROADMAP.md` | You only | You only — never agents |
