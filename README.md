# keith-claude-skills

A small library of [Claude Skills](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview) built by [Keith Lutes](https://keithlutes.com) for applied AI work.

## What's in here

| Skill | Status | What it does |
|---|---|---|
| [`childrens-book-concept-developer`](skills/childrens-book-concept-developer) | ✅ Shipped | Aids an author in exploring an idea or concept for a children's book, socratically. |
| [`merlin-log-triage`](skills/merlin-log-triage) | ✅ Shipped | Takes a log excerpt from a multi-agent forex trading bot and produces a structured "what happened / what looks off" report and can offer suggestions for improvement. |
| [`prospect-brief`](skills/prospect-brief/) | ✅ Shipped | Turns a company name + outreach goal into a structured brief: snapshot, recent signals, conversation hooks, and a draft personalized intro line. |
| `task-file-generator`(skills/task-file-generator) | ✅ Shipped | Turns a 1–3 sentence idea into a multi-phase task file that Claude Code can execute autonomously. |


## Why composable Skills

Most agentic AI tutorials assume you'll wire everything together with bespoke code. Skills are different — they're small, named primitives with their own routing logic, designed so a non-technical teammate can invoke them by name and chain their outputs. The Skills in this repo are deliberately small and single-purpose. The composition story is in how they fit together, not in any one of them being clever.

Each Skill follows the same shape: a `SKILL.md` with YAML frontmatter (name + description) and a Markdown body that teaches Claude *what to produce* and *how to judge*, not *what tools to use*. Policy over mechanism.

## How to use a Skill from this repo

In Claude Code or Claude.ai with Skills enabled, point Claude at the SKILL.md and invoke it by name. For example:

> Use the prospect-brief skill at skills/prospect-brief/SKILL.md. Research [Company]. My outreach goal: [goal].

The Skill will read its own instructions and follow them. See [`skills/prospect-brief/examples/`](skills/prospect-brief/examples/) for a real run.

## Background

These Skills are part of a broader portfolio of applied AI work, including MERLIN (multi-agent forex trading bot (private repo)), VIGIL (its crypto counterpart, running in an isolated VM (also private)), and Pantry Forge (a household inventory app built by a Discord-driven Claude Code dispatcher (currently private and purpose-built for family use only, but I may provide a URL via DM by request, depends on who's asking ;) )).  
All run on a single Claude Max subscription, no per-token API billing.

## Contact

[keithlutes.com](https://keithlutes.com) · [GitHub](https://github.com/scryptoginger) · keithlutes@gmail.com
