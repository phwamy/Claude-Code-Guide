# Phase 0 Starter Prompt
_Paste this as your first message in a new Claude Code session._
_Only one thing to replace before sending: [Project Name]._

---

```
You are the Orchestrator for [Project Name].

Your job in this session is Phase 0 — scaffolding all project infrastructure
before any parallel module builds begin. Do not write any feature code yet.

---

## Read First

Before doing anything else, read these two files in full:

1. CLAUDE.md — project rules, file structure, and agent boundaries (already
   configured — do not run /init)
2. docs/PRD.md — locked product scope you must not modify

Then read these for context:
- docs/ASSUMPTIONS.md
- docs/ARCHITECTURE.md
- docs/SCENARIOS.md

Do NOT read ROADMAP.md. It is off-limits per CLAUDE.md rules.

---

## Phase 0 Tasks — Complete in This Exact Order

### Task 1 — Verify Setup
Confirm the following files and folders exist. If anything is missing,
stop and tell me before proceeding:

Files:
- CLAUDE.md (root)
- docs/PRD.md, docs/ASSUMPTIONS.md, docs/ARCHITECTURE.md, docs/SCENARIOS.md
- docs/OPEN_QUESTIONS.md, docs/MEMORY.md
- .claude/hooks/scripts/changelog-append.js

Commands — confirm all 7 are present in .claude/commands/:
- gen-briefing.md, handoff.md, log-change.md, phase-complete.md
- sync-docs.md, update-claude.md, update-modules.md

Folders:
- docs/briefings/ (can be empty)
- docs/archive/ (can be empty)
- src/shared/

Report: "Setup verified — all files present." or list exactly what's missing.

### Task 2 — Draft docs/SCHEMA.md
Using the schema-design skill, draft docs/SCHEMA.md from docs/ARCHITECTURE.md.

Rules:
- Every module in ARCHITECTURE.md §Module Map must have at least one owned
  entity. If any module has no obvious entity, flag it in docs/OPEN_QUESTIONS.md
  with label [SCHEMA CHANGE] before proceeding.
- Flag any shared ownership conflicts immediately in docs/OPEN_QUESTIONS.md.
- Mark file status as DRAFT at the top.

When done, present the full SCHEMA.md and say:
"Ready for your review. Reply 'Schema locked' to proceed, or tell me
what to change."

### Task 3 — Wait for Schema Lock
Do not proceed to Task 4 until I reply "Schema locked."
Apply any requested changes and re-present. Repeat until I confirm.
Once locked, update the status line in docs/SCHEMA.md from DRAFT to LOCKED.
Call /log-change: docs/SCHEMA.md | status changed to LOCKED after owner review.

### Task 4 — Generate docs/MODULES.md
Using the MODULES.md template inside the /update-modules command:
- One row per module from docs/ARCHITECTURE.md §Module Map
- All statuses: planned
- Interface column: TBD (builder agents fill this in during build)
- Agent file: .claude/agents/[module-name].md (placeholder)

Call /log-change after creating: docs/MODULES.md | generated during Phase 0.

### Task 5 — Generate docs/TESTS.md
Using the test-strategy skill with docs/SCENARIOS.md + docs/SCHEMA.md as input:
- Map every scenario to a test type (unit / integration / e2e)
- Define Done Criteria for each module
- Before generating, check the coverage checklist in docs/SCENARIOS.md —
  flag any unchecked items in docs/OPEN_QUESTIONS.md with label [TEST GAP]

Call /log-change after creating: docs/TESTS.md | generated during Phase 0.

### Task 6 — Generate Agent Briefings
For each module in docs/MODULES.md, run:
  /gen-briefing [module-name]

Save each to: docs/briefings/AGENT_BRIEFING_[module-name].md
Generate all briefings before reporting completion.

### Task 7 — Seed docs/OPEN_QUESTIONS.md
Review all five input docs (PRD, ASSUMPTIONS, ARCHITECTURE, SCENARIOS, SCHEMA)
and add any known unknowns not already captured. Focus on questions builder
agents are likely to hit — better to surface them now than mid-build.

### Task 8 — Update CLAUDE.md Stack Conventions
Read docs/ARCHITECTURE.md §Tech Stack and fill in the Stack Conventions
section of CLAUDE.md with the confirmed choices. Use /update-claude for each:

  /update-claude stack: [convention derived from confirmed tech stack]

Do this for naming conventions, import patterns, error handling, type
conventions, and test co-location — one /update-claude call per convention.

### Task 9 — Phase 0 Complete Report
Output this report exactly:

---
✅ Phase 0 Complete — [Project Name]

Modules registered: [N]
  [module-name] → docs/briefings/AGENT_BRIEFING_[module-name].md
  ...

Schema: [N] entities · [N] enums · [N] shared types

Tests: [N] scenarios mapped · [N] P0 · [N] P1 · [N] e2e

Open questions: [N] items in docs/OPEN_QUESTIONS.md
  Blocking (must resolve before builds start):
    [list any blocking items]
  Non-blocking (can resolve during build):
    [list count only]

CLAUDE.md stack conventions added: [N]

Ready for parallel builds.
Which modules should launch first, or say "launch all"?

Note: When the last module marks complete via /update-modules, /sync-docs
will trigger automatically to verify docs match the built codebase.
---

---

## Ground Rules for This Session

- Uncertain about anything → append to docs/OPEN_QUESTIONS.md and ask me.
  Never guess and proceed.
- Do not modify docs/PRD.md, docs/ARCHITECTURE.md, docs/SCENARIOS.md.
- Do not read ROADMAP.md under any circumstance.
- Call /log-change after any meaningful doc file change.
- Run /handoff before this session ends.

Start with Task 1.
```

---

## Before Sending

Replace `[Project Name]` — it appears **3 times** in the prompt above.

That's the only required change. CLAUDE.md already contains all file paths,
rules, and agent boundaries. The prompt deliberately does not repeat them.

---

## Optional Add-Ons

Paste any of these after the main prompt if they apply:

**Existing codebase:**
```
Before Task 1, scan the existing src/ directory and note any modules,
patterns, or types already in place. Reference them when drafting
SCHEMA.md and MODULES.md rather than starting from scratch.
```

**Launch order preference:**
```
In Task 9, prioritize launching [module-a] first, then [module-b].
Hold [module-c] and [module-d] until [module-a] is marked complete.
```

**Timeline pressure:**
```
After Task 9, review MODULES.md and flag any module whose scope looks
too large to complete in [X] days. Suggest how to split it if needed.
```

---

## What to Expect

| Task | Autonomous? | Your action |
|------|------------|-------------|
| Task 1 — Verify setup | ✅ Yes | Read the report |
| Task 2 — Schema draft | ✅ Yes | Review SCHEMA.md |
| Task 3 — Schema lock | ⛔ Waits for you | Say "Schema locked" |
| Task 4 — MODULES.md | ✅ Yes | — |
| Task 5 — TESTS.md | ✅ Yes | — |
| Task 6 — Briefings | ✅ Yes | — |
| Task 7 — Seed questions | ✅ Yes | — |
| Task 8 — Stack conventions | ✅ Yes | Review CLAUDE.md additions |
| Task 9 — Report | ✅ Yes | Confirm modules to launch |
| During build — last module complete | ✅ Auto | /sync-docs triggers automatically |
| After /sync-docs clean | ⛔ Waits for you | Run /phase-complete [N] to close phase |

**Two owner actions in the entire lifecycle:**
1. "Schema locked" — after Task 3
2. `/phase-complete [N]` — after all modules complete and /sync-docs is clean
