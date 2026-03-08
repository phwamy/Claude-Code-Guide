# Claude Code: Multi-Module Application Development Guide

> A practical reference for building complex, multi-module applications using Claude.ai + Claude Code with multi-session and multi-agent workflows.

---

## Table of Contents

1. [General Prompting Tips](#general-prompting-tips)
2. [Document System](#document-system)
3. [Claude Code Built-In Capabilities](#claude-code-built-in-capabilities)
4. [Recommended Build Order](#recommended-build-order)
5. [Off-the-Shelf Plugins & Community Resources](#off-the-shelf-plugins--community-resources)

---

## General Prompting Tips

Before touching Claude Code, refine your project with Claude.ai using these techniques:

- **Be specific**: State the task, context, number of modules, tech stack, and constraints upfront
- **Use examples**: Show Claude the output format you want
- **Encourage thinking**: Ask Claude to "think step-by-step" or "explain your reasoning" for complex architecture decisions
- **Iterate**: Use follow-up prompts like "That's close, but adjust X to be more like Y"
- **Role-play**: Ask Claude to act as a senior architect, CFO, or product manager to surface perspectives you'd miss
- **Allow uncertainty**: Tell Claude "If you're unsure, say so — don't guess"
- **Break down complexity**: If Claude misses steps, work through them one message at a time

---

## Document System

The following table covers every document needed to build a multi-module application — ordered by when to create them in the project lifecycle.

| Document | Purpose | When & Who Creates | Reader | When & How to Update/Maintain | What's Ready |
|---|---|---|---|---|---|
| **`PRD.md`** | Product vision, use cases, scope, success criteria, out-of-scope | **Pre-build · You + Claude.ai** via interview-style conversation | Product owner, Orchestrator agent | When scope, use cases, or success criteria change. You update with Claude.ai, then re-share with Orchestrator. Version it (v1.1, v1.2). | ✍️ No built-in. Build a custom `/commands/prd-interview.md` that interviews you then generates the doc. |
| **`ASSUMPTIONS.md`** | All assumptions made, risk level, status (holding / under review / invalidated) | **Pre-build · You + Claude.ai** during PRD phase — surface what you're taking for granted | Product owner, Orchestrator | When any assumption is challenged or invalidated during build. Agents append new assumptions; you review and resolve. | ✍️ No built-in. Instruct agents in CLAUDE.md: *"Append to ASSUMPTIONS.md when you make a judgment call."* |
| **`ARCHITECTURE.md`** | System design, module map, data flow, tech stack decisions, integration points | **Pre-build · You + Claude.ai**, then validated in Claude Code Phase 0 | Orchestrator agent, all builder agents | When a module boundary, integration pattern, or stack decision changes. Orchestrator proposes; you approve. | ✍️ No built-in. Custom skill `architecture.md` can generate from PRD + stack constraints. |
| **`SCENARIOS.md`** | End-to-end user journeys and business scenarios that cut across multiple modules. Defines "happy path" and key edge cases as narrative flows, not test code. Used to validate that module boundaries and interfaces are correct before build. | **Pre-build · You + Claude.ai** after ARCHITECTURE.md is stable — derived from PRD use cases | Product owner, Orchestrator agent, all builder agents | When PRD use cases change or new cross-module flows are discovered during build. Orchestrator flags if a scenario can no longer be satisfied. | ✍️ No built-in. Author in Claude.ai. Prompt: *"Given this PRD and architecture, write the 10 most important cross-module scenarios as narrative flows with actor, steps, and expected outcome."* |
| **`CLAUDE.md`** | Always-loaded project brain: stack, conventions, rules, boundaries, what NOT to do | **Phase 0 · Claude Code** scans repo and generates starter. You refine immediately after. | Every agent, every session, automatically | When conventions, stack, or ground rules change. Edit manually or re-run `/init`. Keep under ~200 lines or context cost grows. | ⚙️ Built-in `/init` command generates a starter by scanning your repo. You refine. |
| **`SCHEMA.md`** | Canonical data model, entities, relationships, shared TypeScript types/interfaces | **Phase 0 · Orchestrator agent** drafts from ARCHITECTURE.md; you approve before any parallel work | All builder agents (read-only), Orchestrator (write) | Only via explicit approval from you. Agents flag schema gaps in OPEN_QUESTIONS.md — never edit SCHEMA.md unilaterally. | 🔧 No built-in. Create a custom skill `schema-design.md` triggered during Phase 0. Lock it before parallel streams begin. |
| **`MODULES.md`** | Module registry: name, owner agent, status, dependencies, exposed API/component interfaces | **Phase 0 · Orchestrator agent** generates from PRD + ARCHITECTURE.md | Orchestrator agent, You | When a module is created, completed, or its interface changes. Agents update their own module row on completion. | 🔧 No built-in. Build `/commands/update-modules.md` that agents call when completing or modifying a module. |
| **`OPEN_QUESTIONS.md`** | Unresolved decisions — agents park questions here instead of guessing autonomously | **Phase 0 · Orchestrator agent** seeds with known unknowns from PRD; agents append during build | You (resolve), Orchestrator (routes resolutions back to agents) | Continuously during build. Agents append; you answer; Orchestrator closes items and informs affected agents. Never let it go stale. | 🔧 No built-in. One line in CLAUDE.md enforces it: *"When uncertain, append to OPEN_QUESTIONS.md and stop. Do not guess."* |
| **`AGENT_BRIEFING_[module].md`** | Per-module scoped context: boundaries, owned files, inputs, outputs, edge cases, done criteria | **Before each workstream · Orchestrator agent** slices PRD + SCHEMA + MODULES into a focused brief | That module's builder agent only | Regenerate if SCHEMA.md or module interface changes before the workstream completes. Do not patch — regenerate cleanly. | 🔧 No built-in. Build `/commands/gen-briefing.md` that takes a module name and outputs its briefing from current PRD + SCHEMA. |
| **`/agents/[module].md`** | Subagent definition: tools allowed, model tier, memory scope, system prompt, file boundaries | **Before each workstream · You** define once; reuse across sessions for the same module | Spawned automatically by Orchestrator for parallel workstreams | When a module's tool permissions or model tier needs adjusting. Stable once defined per module. | ⚙️ Framework built-in. Use `/agents` command to scaffold interactively. You write the definition files. |
| **`/skills/[domain].md`** | Reusable domain expertise loaded on-demand (e.g. `api-design`, `db-migrations`, `testing`) | **Before or during build · You** author once; available to all agents that need it | Any agent — auto-invoked when task context matches skill description | When the domain pattern evolves (e.g. new testing framework adopted). Update the skill file; all agents pick it up immediately. | ⚙️ Framework built-in. Auto-invocation works once files exist. Content is yours to write or install from community GitHub repos. |
| **`TESTS.md`** | Test strategy, coverage requirements, test types per module (unit/integration/e2e), shared test data setup, and which SCENARIOS.md flows must have e2e coverage. Links scenarios to specific test suites. | **Before parallel build · You + Orchestrator** — after SCENARIOS.md and SCHEMA.md are locked | All builder agents, QA agent | When a new scenario is added or a module's done-criteria changes. Builder agents must check TESTS.md before marking a module complete. | 🔧 No built-in. Prompt: *"Given SCENARIOS.md and SCHEMA.md, generate a TESTS.md that maps each scenario to test type, required coverage, and shared fixtures."* Consider a custom skill `test-strategy.md`. |
| **`MEMORY.md`** | Rolling cross-session log: decisions made, patterns found, gotchas discovered, key file locations | **During build · Agents** write automatically at session end; Orchestrator reads at session start | Orchestrator agent (reads at start, appends at end) | Every session boundary. Agents curate it — auto-truncated to 200 lines. You review periodically to promote stable entries into CLAUDE.md. | ⚙️ Partially built-in. `memory: project` frontmatter in agent definition auto-manages read/write. You design the format. |
| **`CHANGELOG.md`** | What changed, when, which modules and documents were impacted, who decided | **During build · Agents + hooks** append automatically on significant writes | Product owner, Orchestrator | On every meaningful code or doc change. Hook on `PostToolUse` Write events auto-appends entries, or agents call `/commands/log-change.md`. | 🔧 No built-in. Create a `PostToolUse` hook to auto-append, or a slash command agents invoke manually. |
| **`/commands/handoff.md`** | Slash command: writes a structured session summary before `/clear` so next session starts informed | **During build · You** create once, invoke at every session boundary | You + next Claude Code session (loaded via MEMORY.md) | Rarely needs updating once written. Extend it if your summary format evolves. | 🔧 No built-in but trivial to create. High-value, low-effort. Build it on Day 1 of Claude Code work. |
| **`Plugins`** | Bundle of skills + commands + agents + hooks into one distributable, namespaced unit | **Post-build · You** once your stack is stable and worth sharing across projects or teams | Team members, other projects | When the bundled components are updated. Republish the plugin; consumers reinstall. | ⚙️ Built-in via `/plugins`. Build last — only after your individual components are proven and stable. |

### Legend

| Icon | Meaning |
|---|---|
| ✍️ | Start in **Claude.ai** before Claude Code. Human + AI co-authoring. No automation ready yet. |
| ⚙️ | **Framework built-in** to Claude Code. You configure or refine — the mechanism already exists. |
| 🔧 | **Not built-in.** You need to build a custom skill, command, hook, or agent to automate this. |

---

## Claude Code Built-In Capabilities

Claude Code has **7 native layers** that stack together for multi-session, multi-agent workflows.

### Layer 1 — `CLAUDE.md` (Persistent Project Memory)
Your always-loaded session header. Every agent, every session reads it automatically. Use it to define stack, conventions, module ownership rules, and what agents must never do. Keep it under ~200 lines to avoid bloating every agent's context.

### Layer 2 — Skills (Auto-Invoked Domain Expertise)
Markdown files in `.claude/skills/` (project) or `~/.claude/skills/` (global). Unlike slash commands, skills **activate automatically** when their description matches the task context — the agent reads the skill file on demand, like loading domain expertise on the fly. Author your own or install community skills from GitHub.

### Layer 3 — Slash Commands (Explicit Repeatable Workflows)
Stored in `.claude/commands/`. Triggered manually by you or explicitly by agents. Use for structured workflows you want deterministic control over: `/handoff`, `/gen-briefing`, `/update-modules`, `/log-change`.

### Layer 4 — Subagents (Parallel Isolated Workstreams)
Defined as markdown files with YAML frontmatter in `.claude/agents/` (project) or `~/.claude/agents/` (global). Frontmatter controls: tools allowed, model tier, memory scope, MCP servers available.

Each subagent gets its own isolated context window — one agent's deep read of auth code doesn't pollute another's analysis. Claude Code ships with 3 hidden built-in agents:
- **Explore** — Haiku, read-only, fires on "where is X?" queries
- **Plan** — Read-only, used before implementation in plan mode
- **General-purpose** — Full tool access, used for complex delegated tasks

### Layer 5 — Hooks (Automated Lifecycle Rules)
Configured in `.claude/settings.json`. Cover **17 lifecycle events** across session, tool, agent, and notification phases. Exit code `2` gives a hard block. Common uses: block writes outside module boundary, auto-run tests after edits, enforce lint before commit, auto-append CHANGELOG entries.

### Layer 6 — `MEMORY.md` (Dynamic Cross-Session Memory)
Set `memory: project` in agent frontmatter. The agent's system prompt auto-includes the first 200 lines of MEMORY.md and instructs the agent to curate it when it exceeds that length. Periodically promote stable entries from MEMORY.md into CLAUDE.md.

### Layer 7 — Plugins (Bundled Distribution)
Bundle skills + commands + agents + hooks into a single installable unit via `/plugins`. Namespaced to avoid conflicts. Build last — only once your individual components are stable and worth sharing across projects or teams.

---

## Recommended Build Order

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STAGE 0 — Discovery
Tool: Claude.ai  |  Author: You  |  Claude.ai is thinking partner
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  You + Claude.ai  →  PRD.md            (interview-style, versioned)
  You + Claude.ai  →  ASSUMPTIONS.md    (surface what you're taking for granted)
  You + Claude.ai  →  ARCHITECTURE.md   (module map, data flow, stack decisions)
  You + Claude.ai  →  SCENARIOS.md      (cross-module user journeys from PRD use cases)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STAGE 1 — Foundation
Tool: Claude Code  |  Orchestrator executes  |  You approve all outputs
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  /init                →  CLAUDE.md            (auto-generated, you refine)
  Orchestrator         →  SCHEMA.md            (drafted from ARCHITECTURE; you lock it)
  Orchestrator         →  MODULES.md           (module registry from PRD + ARCHITECTURE)
  Orchestrator         →  OPEN_QUESTIONS.md    (seed with known unknowns)
  You + Orchestrator   →  TESTS.md             (test strategy from SCENARIOS + SCHEMA)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STAGE 2 — Workstream Setup
Tool: Claude Code  |  You configure  |  Orchestrator dispatches
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  You                  →  /agents/[module].md      (one definition per module)
  You                  →  /skills/[domain].md      (reusable expertise files)
  You                  →  /commands/handoff.md     (build once, use every session)
  Orchestrator         →  AGENT_BRIEFING_[x].md    (generated per module before spawn)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STAGE 3 — Parallel Build
Tool: Claude Code  |  Builder agents execute  |  You resolve blockers daily
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Builder agents       →  Code + tests             (confined to module directory)
  Agents (auto)        →  MEMORY.md                (appended each session end)
  Agents (auto)        →  CHANGELOG.md             (appended via hook on writes)
  Agents (flag)        →  OPEN_QUESTIONS.md        (never guess — park and stop)
  You                  →  OPEN_QUESTIONS.md        (resolve daily)
  Orchestrator         →  Routes resolutions back to affected agents

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STAGE 4 — Integration
Tool: Claude Code  |  Orchestrator leads  |  You review
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Orchestrator         →  Validates SCENARIOS.md flows end-to-end
  Orchestrator         →  Validates TESTS.md coverage requirements met
  Orchestrator         →  MODULES.md (mark all modules complete)
  Orchestrator         →  End-to-end wiring (reads SCHEMA + all briefings)
  You + Orchestrator   →  CHANGELOG.md (tag integration milestone)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
STAGE 5 — Stabilize & Share
Tool: Claude.ai + Claude Code  |  You decide what to keep and share
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  You                  →  Promote stable MEMORY.md entries into CLAUDE.md
  You                  →  Archive AGENT_BRIEFING files (no longer needed)
  You                  →  Plugins (bundle if reusing across projects/teams)
```

### Session Boundary Pattern (repeat every session)

```
Session start:   CLAUDE.md (auto) + MEMORY.md (auto) → agent is context-loaded
During session:  OPEN_QUESTIONS.md (park blockers) + CHANGELOG.md (auto-log)
Session end:     /handoff → write summary → /clear → next session starts clean
```

---

## Off-the-Shelf Plugins & Community Resources

These are real, installable resources — not things you need to build from scratch.

### High-Value Custom Commands to Build First

These don't exist as built-ins but are trivial to create and immediately valuable:

| Command | What It Does | Build Effort |
|---|---|---|
| `/handoff` | Writes structured session summary before `/clear` | Low — 1 markdown file |
| `/prd-interview` | Claude interviews you and generates PRD.md | Medium — interview prompt + output format |
| `/gen-briefing` | Generates AGENT_BRIEFING for a named module from current PRD + SCHEMA | Medium — needs file-reading logic |
| `/update-modules` | Updates MODULES.md registry row for a completed module | Low — structured append command |
| `/log-change` | Appends a structured entry to CHANGELOG.md | Low — template fill |

### Community Skill & Agent Libraries

- **awesome-claude-code** (GitHub) — curated list of community skills, commands, hooks, and agents
- **github.com/wshobson/agents** — 72 plugins covering Python, Kubernetes, LLM apps, blockchain, and more. Includes a full orchestration pipeline: `backend-architect → database-architect → frontend-developer → test-automator → security-auditor → deployment-engineer`

### Memory Plugins

- **claude-mem** (npm: `claude-mem`) — automatically captures session activity, compresses it with AI, and injects relevant context back into future sessions. Adds vector search on top of plain-text MEMORY.md notes
- **mem0 MCP server** — adds semantic vector search over longer memory histories when MEMORY.md grows too large for linear reading

### Orchestration Platforms

- **ruflo** (`npx ruflo@latest`) — full multi-agent orchestration platform built on Claude Code. Handles context compaction lifecycle (blocks auto-compaction, archives turns to SQLite, restores importance-ranked context on session start). Adds human-agent task claiming, semantic embeddings, and a knowledge graph over memory entries

### Installing Community Plugins

```bash
# Inside a Claude Code session
/plugins install <plugin-name>

# Or add an MCP server directly
claude mcp add ruflo -- npx -y ruflo@latest mcp start
```

### What to Build vs. What to Install

| Need | Recommendation |
|---|---|
| Session memory across projects | Install `claude-mem` plugin |
| Multi-agent orchestration at scale | Evaluate `ruflo` |
| Domain-specific skills (testing, API design, DB) | Browse `awesome-claude-code`, then customize |
| PRD → structured docs pipeline | Build custom — this is your highest-value investment |
| AGENT_BRIEFING generation | Build custom — tightly coupled to your PRD format |
| OPEN_QUESTIONS enforcement | Just a line in CLAUDE.md — no plugin needed |

---

## Key Principles

1. **CLAUDE.md is what every agent always knows. Agent Briefings are what only that agent needs to know.** Keep them separate — context costs drop dramatically.

2. **PRD is for humans and planning. Agent Briefings are PRD slices compiled for machines to execute.** They are different documents serving different purposes.

3. **Lock SCHEMA.md before any parallel work begins.** It is the signed contract between all agents. Agents flag schema gaps in OPEN_QUESTIONS.md — never edit it unilaterally.

4. **SCENARIOS.md validates your architecture before you build it.** If a scenario can't be satisfied by your module map and interfaces, your architecture is wrong. Fix it before writing code.

5. **TESTS.md bridges SCENARIOS.md and code.** Every scenario should trace to at least one test. Agents check TESTS.md before marking a module done.

6. **Never let OPEN_QUESTIONS.md go stale.** Resolve daily. An unanswered question is a silent assumption waiting to break something.

7. **Promote MEMORY.md → CLAUDE.md regularly.** Stable discoveries belong in the always-loaded context, not buried in a rolling log.
