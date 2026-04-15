---
name: prospect-brief
description: Use this skill whenever someone needs to research a company before reaching out to them — for sales prospecting, business development, partnership outreach, cold email prep, or any "I need to know enough about this company in 90 seconds to write something that doesn't sound templated" situation. Trigger on phrases like "research this company," "prospect brief," "background on," "what should I know about," or any time a company name or website is provided alongside an outreach goal. Not for product questions, financial analysis or one-line factual lookups. This skill produces a structured brief with company snapshot, recent signals, conversation hooks, and a draft personalized intro line.
---

# Prospect Brief

This skill helps a salesperson, BD person, or founder turn a company name or URL into a short, useful prospect brief in under two minutes. The goal is not exhaustive research — it's *enough* research to write a personalized outreach message that lands.

## Inputs Contract

At minimum, the input required is Company Name OR URL, AND an outreach goal. If this is missing, request from the User that they provide at least these pieces of information. If the information is not given, or unavailable, proceed with what _is_ available and provide notice to User of potential degraded output quality due to lack of sufficient input.

## What you produce

A structured brief with five sections, in this exact order. Brevity is the point — a salesperson should be able to read the whole thing in 60 seconds.

### 1. Company snapshot (3–5 lines max)
- **Name and URL**
- **What they do** — one sentence in plain language, no marketing jargon
- **Size signal** — employee count range, revenue range, or stage if known
- **HQ location** — city/region

### 2. Recent signals (3–6 bullets)
Things that have happened in the last ~12 months that an outreach message could legitimately reference. Prioritize:
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
A single, personalized opening sentence the user can drop into a cold email or LinkedIn message. It needs to sound human as though the Sender spent meaningful time and effort thinking about this company before crafting the response, but short and to the point. The more human, the better. It can't be canned or generic. 

### 5. Confidence and gaps
- **Confidence level**: high / medium / low
- **What's missing**: list anything you couldn't find that a salesperson would normally want (e.g., "no recent funding info," "couldn't identify decision-maker for [function]")
- **Recommended next step**: usually one of "send the draft as-is," "verify [specific thing] before sending," or "escalate to human research"

## Output format

Output the brief as Markdown with the section headers above. Do NOT wrap it in JSON unless explicitly asked. The audience is a human salesperson reading on their phone or in a CRM — Markdown renders cleanly in both.

## Judgment guidance

**On signal quality**: Not every news mention is a signal. A funding round is a signal. A press release about attending a conference is not. When in doubt, ask: "Could a salesperson reference this in a way that sounds like they actually pay attention to this company?" If yes, include it. If no, drop it.

**On the draft intro**: This is the hardest part and the most valuable. The intro should feel like it was written by someone who spent 20 minutes thinking about the prospect, even though the Skill spent 90 seconds. Avoid:
- Generic praise ("Big fan of what you're building")
- Vague references ("Saw you've been growing")
- Anything that could be sent to any company in their industry

**On confidence**: Be honest. If the prospect is a small private company with little public footprint, the confidence is low and you say so. A salesperson would rather know the limits of the brief than discover them after sending a bad intro.

**On escalation**: If you cannot produce at least one specific signal AND one specific hook, recommend escalation. A bad personalized intro is worse than no intro at all.

## Examples

### Example 1: Strong public signal
**Input**: "Research Linear (linear.app) — I want to pitch them on our developer onboarding tool."

**Expected output shape**: Snapshot mentions issue tracking for software teams; signals include any recent product launches, funding, or leadership posts; hooks tie those to developer onboarding; intro references a specific recent thing Linear has shipped or said publicly.

### Example 2: Weak public signal
**Input**: "Research Acme Plumbing of Sheboygan — I want to pitch them our scheduling software."

**Expected output shape**: Snapshot is sparse but honest; signals section says "no notable public signals"; recommend escalation or generic intro; confidence is low.

### Example 3: The meta case
**Input**: "Research TLDR (tldr.tech) — I'm applying for their Senior SWE Applied AI role and want to reference something specific in my cover letter."

**Expected output shape**: Snapshot covers TLDR's newsletter network; signals include recent hiring, recent newsletter launches, founder activity; hooks tie to the Applied AI hiring direction; intro line is suitable for a cover letter opening.

## What this skill does NOT do

- Pull from paid databases (Crunchbase, ZoomInfo, Apollo) — public sources only
- Generate full email drafts — only the opening line; the rest is the user's job
- Fabricate information — if a signal isn't verifiable, do not include it
- Replace human research for high-stakes outreach — this is for volume, not for your top 5 accounts
