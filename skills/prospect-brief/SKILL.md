---
name: prospect-brief
description: Use this skill whenever someone needs to research a company before reaching out to them — for sales prospecting, business development, partnership outreach, cold email prep, or any "I need to know enough about this company in 90 seconds to write something that doesn't sound templated" situation. Trigger on phrases like "research this company," "prospect brief," "background on," "what should I know about," or any time a company name or website is provided alongside an outreach goal. Recency window for signals is the last 12 months relative to today's date. Not for product questions, financial analysis, one-line factual lookups, technical "background on [topic]" queries, or general curiosity about a company outside an outreach context.
---

# Prospect Brief

This skill helps a salesperson, BD person, or founder turn a company name or URL into a short, useful prospect brief in under two minutes. The goal is not exhaustive research — it's *enough* research to write a personalized outreach message that lands.

## Inputs Contract

At minimum, the input required is a company name OR URL, AND an outreach goal. If either is missing, request it from the user once. If the user proceeds without supplying it, work with what's available and note in the Confidence section that output quality is degraded due to missing input.

If the company name is ambiguous (e.g. "Acme" — could be many companies), ask one clarifying question to disambiguate (industry, location, or URL). Do not guess.

This skill uses public web sources via web search and web fetch. It does not pull from paid databases.

## What you produce

A structured brief with five sections, in this exact order. Brevity is the point — a salesperson should be able to read the whole thing in 60 seconds.

### 1. Company snapshot (3–5 lines max)
- **Name and URL**
- **What they do** — one sentence in plain language, no marketing jargon
- **Size signal** — employee count range, revenue range, or stage if known
- **HQ location** — city/region

### 2. Recent signals (3–6 bullets)
Things that have happened in the last ~12 months (relative to today's date) that an outreach message could legitimately reference. Prioritize:
- Funding rounds, acquisitions, or major deals
- Leadership changes (new CEO, new VP of [relevant function])
- Product launches or major releases
- Public hiring signals (job postings in relevant areas)
- Press, podcasts, blog posts by leadership
- Industry awards or recognition

If you cannot find any signals, say so explicitly: *"No notable public signals in the last 12 months — recommend escalating to a human researcher or proceeding with a generic intro."*

### 3. Conversation hooks (2–4 bullets)
Things you'd actually mention in a first message. These are the bridges between "what I learned" and "what I'd say." Each hook should be:
- Specific (not "they care about growth")
- Tied to a recent signal where possible
- Phrased as something you could naturally drop into a sentence

### 4. Draft intro line (1–2 sentences)
A single, personalized opening sentence the user can drop into a cold email or LinkedIn message. It needs to sound human — as though the sender spent meaningful time thinking about this company before writing — but stay short and to the point. Reference at least one specific signal or hook by name. Stay under 40 words.

### 5. Confidence and gaps
- **Confidence level**: high / medium / low
- **What's missing**: list anything you couldn't find that a salesperson would normally want (e.g., "no recent funding info," "couldn't identify decision-maker for [function]")
- **Recommended next step**: usually one of "send the draft as-is," "verify [specific thing] before sending," or "escalate to human research"

## Output format

Output the brief as Markdown with the section headers above. Return Markdown, not JSON, unless the user explicitly asks for JSON. The audience is a human salesperson reading on their phone or in a CRM — Markdown renders cleanly in both.

## Judgment guidance

**On signal quality**: Not every news mention is a signal. A funding round is a signal. A press release about attending a conference is not. When in doubt, ask: "Could a salesperson reference this in a way that sounds like they actually pay attention to this company?" If yes, include it. If no, drop it.

**On the draft intro**: This is the hardest part and the most valuable. The intro should feel like it was written by someone who spent 20 minutes thinking about the prospect, even though the Skill spent 90 seconds. The point of this line is to prove the sender actually researched the prospect — generic openers like "I hope this finds you well" or "I came across your company" do the opposite, signaling a template and destroying the entire reason for doing the research. The same logic applies to vague praise ("big fan of what you're building") or vague observations ("saw you've been growing"): if the line could be sent unchanged to any company in their industry, it has failed its job. Aim for a sentence that would be embarrassing to send to the wrong prospect.

**On confidence**: Be honest. If the prospect is a small private company with little public footprint, the confidence is low and you say so. A salesperson would rather know the limits of the brief than discover them after sending a bad intro.

**On escalation**: If you cannot produce at least one specific signal AND one specific hook, recommend escalation. A bad personalized intro is worse than no intro at all.

## Example output

This is what a real run of this skill produced when researching TLDR (tldr.tech) for a candidate applying to their Senior SWE, Applied AI role.

---

# Prospect Brief: TLDR (tldr.tech)

**Outreach goal:** Cover letter for Senior SWE, Applied AI role — hook tied to the "build Claude Skills for non-technical teammates" framing.

## 1. Company snapshot

- **Name / URL:** TLDR — https://tldr.tech
- **What they do:** Publishes a network of daily tech newsletters (AI, Dev, Product, Infosec, Crypto, Marketing, Design, Fintech, ~14 verticals) summarizing industry news in a byte-sized format.
- **Size signal:** ~22 full-time employees, remote; ~30 part-time newsletter authors. 8-figure profitable revenue.
- **HQ:** Fully remote; founder Dan Ni based in the Bay Area.

## 2. Recent signals (last ~12 months)

- **7M+ subscribers** reported as of October 2025 — up from 5M in mid-2024.
- **Applied AI team now hiring** with explicit framing around modular Claude Skills that connect to systems of record, designed for non-technical teammates to invoke and chain together.
- **Heavy Anthropic coverage cadence in TLDR AI** — Claude Cowork, Claude Code features, Sonnet model news. They're not just users of Claude; they're a primary distribution channel for news *about* it.
- **Lean-team story is central to their brand** — publicly profiled as growing 8x with minimal headcount.

## 3. Conversation hooks

- **Recursive meta:** the candidate built a Claude Skill that matches the exact pattern in the JD and used it to produce this cover letter. The artifact is the answer.
- **Lean-team fit:** TLDR's operating ethos is small team + heavy AI leverage. A candidate who ships Skills letting non-engineers run production workflows maps directly to how they already operate.
- **Editorial-stack alignment:** TLDR covers Claude / Claude Code / Cowork weekly. Building with that stack is building toward what they write about.

---

(The full brief continues with sections 4 and 5, plus sources. See `examples/tldr-prospect-brief.md` for the complete output.)

## What this skill does NOT do

- Pull from paid databases (Crunchbase, ZoomInfo, Apollo) — public sources only
- Generate full email drafts — only the opening line; the rest is the user's job
- Fabricate information — if a signal isn't verifiable, do not include it
- Replace human research for high-stakes outreach — this is for volume, not for your top 5 accounts
