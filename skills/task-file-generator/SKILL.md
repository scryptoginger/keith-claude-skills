---
name: task-file-generator
description: 'Use this skill whenever someone has a short idea — a sentence or two describing something they want built, scaffolded, configured, or automated — and needs it turned into a structured, multi-phase task file that an AI coding agent (Claude Code, Cursor, or similar) can execute autonomously. Trigger on phrases like "write a task file for," "scaffold this project," "break this into phases," "plan this build," "create an autonomous task," "overnight session plan," or any time someone describes a project idea and wants it decomposed into an executable plan with phases, deliverables, and validation steps. Not for writing code directly, executing the task, managing an existing project, or generating single-file scripts — this Skill produces the plan, not the implementation.'
---

# Task File Generator

This skill turns a 1–3 sentence project idea into a structured, multi-phase task file designed to be handed to an AI coding agent for autonomous execution. The output is a complete session plan — phased, scoped, with commit messages, validation checks, and clear boundaries — so the operator can start an overnight or unattended Claude Code session with confidence that the agent knows what to build, in what order, and how to verify its own work.

This is a meta-primitive: a Skill whose output is a directive that drives further AI work. The task file doesn't build anything — it tells another AI session exactly how to build it.

## Inputs Contract

The required input is a project description of 1–3 sentences. The description should convey:

* **What** to build (the artifact — an app, a bot, a system, a tool)
* **Why** it exists (the purpose — even one clause is enough)
* **Any hard constraints** (language, framework, deployment target, existing repos to reference)

If the description is too vague to decompose into phases (e.g., "build something cool"), ask one clarifying question — no more. If the user provides only a "what" with no "why" or constraints, proceed with reasonable defaults and note assumptions in the Context section of the output.

Optional inputs that improve output quality:
* A reference repo or existing codebase to model after
* Deployment target (local server, cloud, container, etc.)
* Session type (overnight autonomous, paired interactive, time-boxed sprint)
* Technology preferences or restrictions

## What you produce

A single Markdown document with these sections, in this exact order. The document should be copy-pasteable into a Claude Code session as the complete set of instructions for an autonomous build.

### 1. Header

```markdown
# [Project Name] — Autonomous Task File [NN]: [Short Title]
**Session Type:** [Overnight autonomous / Paired interactive / Time-boxed sprint]
**Reference Repo:** [URL or "none"]
**Target Repo:** [URL or "create if not exists"]
**Commit frequently:** After every major milestone. Push to GitHub after every commit.
```

### 2. Context

A short block (5–10 lines) explaining:
* What this project is
* What reference material exists (repos, docs, prior art)
* The guiding philosophy or design principles (if any)
* What this project is NOT — explicit scope exclusions

This section is written *for the AI agent*, not for a human manager. It should give the agent enough understanding to make reasonable judgment calls during implementation without asking the operator.

### 3. Phases

The core of the task file. Each phase is a numbered block following this structure:

```markdown
## PHASE N — [Title]

### N.1 [Sub-step title]
[Detailed instructions — what to implement, what schema to use, what conventions to follow]

### N.2 [Sub-step title]
[...]

**Commit message:** `[type]: [description]`
```

**Phase design rules:**

* **Phase 0 is always environment setup.** Dependencies, git identity, repo initialization, virtual environment. Never skip this.
* **Each phase produces a testable artifact.** If you can't describe what to test after the phase, the phase is scoped wrong — split it or merge it.
* **Each phase ends with a commit message.** Use conventional commit format: `feat:`, `chore:`, `docs:`, `fix:`, `test:`.
* **Mark implementation depth explicitly.** For each component, state whether it should be implemented fully, stubbed (structure + interface, returns placeholder data), or skipped (noted for a future session). Never leave this ambiguous.
* **The most critical component gets its own phase and is implemented early.** In a trading system, that's the risk engine. In a web app, that's auth. In a data pipeline, that's the schema. Identify it and prioritize it.
* **Order phases by dependency.** Infrastructure → core abstractions → critical path → supporting features → integration → validation. An agent executing top-to-bottom should never hit a forward dependency.
* **Include code scaffolds where precision matters.** Class signatures, schema definitions, API endpoint signatures, and config structures should be specified — not as full implementations, but as contracts the agent must honor. Prose descriptions of "build a thing that does X" without structural guidance produce inconsistent results.

### 4. Validation Phase (always the final numbered phase)

A concrete checklist the agent runs before ending the session:

```markdown
## PHASE [LAST] — Final Validation

Before ending the session, run this checklist and fix anything that fails:

1. Import check — all modules must import without error
2. Dry run — the main entry point runs without crashes
3. Data integrity — all generated JSON/config files are valid
4. Git status — nothing uncommitted
5. Git log — verify commit history matches expected phases
```

Include actual bash commands or Python one-liners the agent can execute, not just descriptions of what to check.

### 5. Push Reminders

```markdown
## PUSH REMINDERS

Push to GitHub after EVERY commit. Do not batch pushes.
```

Short, emphatic, non-negotiable. Agents that batch pushes lose work when sessions crash.

### 6. What Success Looks Like

A numbered list of concrete things the operator should be able to do when the session ends:

```markdown
## WHAT SUCCESS LOOKS LIKE

When this session ends, [operator] should be able to:
1. [concrete action — e.g., "run a dry-run cycle with no crashes"]
2. [concrete action — e.g., "open the dashboard in a browser"]
3. [concrete action — e.g., "read the README and understand setup"]
```

This is the acceptance criteria. Write it from the operator's perspective, not the agent's.

### 7. Next Session Preview

```markdown
## NEXT SESSION PREVIEW (do not implement now, just note)

- [Feature or improvement deferred to next session]
- [...]
```

Explicit scope boundary. Tells the agent what's coming next so it can design for extensibility without actually building it. Also prevents scope creep — if the agent starts implementing something on this list, it's gone off-plan.

## Output format

Output as a single Markdown document. Do not split into multiple files. The entire task file should be copy-pasteable into a Claude Code session as one artifact.

Use fenced code blocks for all bash commands, Python snippets, directory trees, and JSON schemas. The agent will execute these literally — ensure they're correct and runnable.

## Judgment guidance

**On phase count.** A typical task file has 8–15 phases. Fewer than 5 means phases are too coarse — the agent will make too many unsupervised decisions within a single phase. More than 20 means phases are too granular — the overhead of committing and context-switching outweighs the benefit. Aim for phases that take 5–20 minutes of agent work each.

**On stub vs. implement.** Default to "implement fully" for core-path components and "stub" for supporting features. The first session should produce a system that *runs end-to-end on the critical path*, even if secondary features return placeholder data. A working skeleton with stubs is more valuable than a half-implemented full system.

**On the Context section.** This is the most underrated section. A weak Context section produces an agent that makes wrong assumptions silently. A strong Context section produces an agent that makes reasonable judgment calls without asking. Include: what the project IS, what it ISN'T, what existing work to reference, and what principles to follow. Write it as if briefing a competent contractor who's never seen your codebase.

**On commit messages.** Specify them. Agents left to write their own commit messages produce noise like "update files" and "add stuff." Prescribed commit messages create a readable git log that the operator can scan in 10 seconds to verify progress.

**On overnight sessions.** If the session type is "overnight autonomous," the task file must be self-contained — no "ask the operator" steps, no decisions that require human judgment mid-session. Everything the agent needs to know must be in the document. If a decision genuinely requires human input, defer it to the next session and note it in the Next Session Preview.

**On scope creep.** The task file is a *contract*. If the agent encounters something interesting that isn't in the plan, it should note it in the Next Session Preview, not implement it. The strongest signal of a well-designed task file is an agent that finishes exactly what was specified and nothing more.

## Example output

This is a condensed excerpt from a real task file used to bootstrap VIGIL (a crypto trading system) from MERLIN (a forex trading system) in a single overnight Claude Code session. The full task file is 15 phases covering environment setup through final validation.

---

```markdown
# VIGIL — Autonomous Task File 01: Bootstrap & Scaffold
**Session Type:** Overnight autonomous (Claude Code, danger mode)
**Reference Repo:** https://github.com/scryptoginger/merlin-trading.git
**Target Repo:** https://github.com/scryptoginger/vigil-trading.git (create if not exists)
**Commit frequently:** After every major milestone. Push to GitHub after every commit.

---

## CONTEXT

VIGIL is a crypto trading system modeled after MERLIN — a multi-agent AI forex trading bot
using a hub-and-spoke blackboard architecture. You have the MERLIN repo as your reference.
VIGIL is NOT a copy of MERLIN. It is a crypto-native rebuild using MERLIN's philosophy,
architecture patterns, and agent names where appropriate, adapted for 24/7 crypto markets.

MERLIN philosophy to carry forward:
- Remove human emotion entirely. Enforce discipline through code.
- The Shield has absolute veto. No overrides. Ever.
- Consensus over conviction. Min 2 theories must agree.
- 1% account risk per trade. Hard floor. Non-negotiable.
- The data tells you when the strategy needs to change.

---

## PHASE 0 — Environment Setup (do this first, do not skip)

### 0.1 System dependencies
sudo apt update && sudo apt upgrade -y
sudo apt install -y python3 python3-pip python3-venv git curl build-essential

### 0.2 Git identity
git config --global user.name "VIGIL Bot"
git config --global user.email "vigil@keithdev.local"

[...]

**Commit message:** `chore: initialize environment and clone repos`

---

## PHASE 4 — The Shield (implement fully — this is the most important agent)

agents/shield.py — Pure rules-based. No AI. Absolute veto authority.

The Shield must enforce ALL of the following before any trade is approved:

1. System halt check — if shield.daily_halt is True, veto everything
2. Daily drawdown check — if daily drawdown ≥ 3%, set halt, veto
3. Max positions check — if open positions ≥ 5, veto
4. Risk per trade — position size must be ≤ 1% of account equity
5. Confidence threshold — signal confidence must be ≥ 62%
[...]

**Commit message:** `feat: implement Shield with full veto rule set`

---

## PHASE 15 — Final Validation

Before ending the session, run this checklist and fix anything that fails:

python -c "from agents.shield import Shield; print('Shield OK')"
python -c "from core.blackboard import Blackboard; print('Blackboard OK')"
python vigil.py --dry-run

**Final commit message:** `chore: phase 15 validation pass — VIGIL session 01 complete`

---

## WHAT SUCCESS LOOKS LIKE

When this session ends, Keith should be able to:
1. ssh into the machine
2. cd ~/vigil-trading && git log --oneline — see 15+ commits
3. source venv/bin/activate && python vigil.py --dry-run — run a full cycle with no crashes
4. Open browser to localhost:5001 — see the live dashboard
5. Read README.md and understand how to configure .env and go live

---

## NEXT SESSION PREVIEW (do not implement now, just note)

- Wire Oracle to real Gemini 2.5 Flash API
- Implement Archivist post-trade analysis via Claude API
- Flesh out remaining theory stubs
- Binance testnet account setup and real paper trading loop
```

---

*(The full VIGIL bootstrap task file is 15 phases. See `examples/vigil-bootstrap-task-file.md` for the complete version.)*

## What this skill does NOT do

* **Execute the task file.** This Skill produces the plan. Running it is a separate step — hand it to Claude Code, Cursor, or whatever agent you use.
* **Write production code.** Code scaffolds in the task file are contracts and signatures, not implementations. The executing agent writes the actual code.
* **Manage existing projects.** This Skill generates greenfield task files. Refactoring, debugging, or extending an existing codebase is a different workflow.
* **Replace project management.** The task file is a single-session build plan, not a roadmap, sprint backlog, or requirements document.
* **Generate task files for non-coding work.** This is for software projects. Writing a blog post, planning a move, or organizing a conference is out of scope.
