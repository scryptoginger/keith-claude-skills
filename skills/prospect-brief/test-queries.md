# prospect-brief test queries

Test cases for validating the `prospect-brief` skill's routing description. Generated during skill-creator review on April 14, 2026.

The "should trigger" set covers casual phrasings, lowercase, abbreviations, and personal context — the kinds of inputs a real salesperson would type. The "should NOT trigger" set is built from near-misses: queries that share keywords or concepts with prospect research but actually need something different.

## Should trigger

1. "i'm cold-emailing the VP Eng at linear.app tomorrow morning — give me like 90 seconds of context on them, something i can actually reference"
2. "can you pull a prospect brief on ramp.com? pitching them our AP automation tool, meeting is thursday"
3. "what should i know about figma before i reach out to their head of partnerships? we're an AI design plugin"
4. "working thru a list of 40 accounts my SDR built, first one is https://www.datadoghq.com — need a quick read on each"
5. "background on notion.so pls, writing an intro note to their COO"
6. "about to send a LinkedIn DM to the founder of replit.com, can you tell me what they've been up to lately"
7. "prep me for outreach to vercel — we sell observability and i want to not sound like every other vendor in their inbox"
8. "research tldr.tech, im applying for their senior swe applied ai role and need smth specific to drop in my cover letter"
9. "need a one-pager on stripe before my BD call monday, focus on anything they've shipped or said publicly in the last year"
10. "give me the rundown on mongodb.com, pitching an integration and want to not look like an idiot"

## Should NOT trigger (near-misses)

1. "what does stripe charge for their API? trying to price out a side project" — product/pricing, not outreach
2. "compare datadog and new relic for our 50-person eng team, we're picking one" — vendor evaluation for self
3. "who founded airbnb and when?" — one-line factual
4. "summarize this techcrunch article about vercel's latest round for me" — summarization of provided text
5. "explain how linear's cycles feature works, i'm trying to decide if our team should use it" — product feature explanation
6. "write me a follow-up email to my contact at figma, we last spoke 3 weeks ago" — email drafting, research not the ask. Near-miss because of "figma" + "email"
7. "what's mongodb trading at right now and what was their last earnings like" — financial/equity, out of scope
8. "i need background on the react 19 migration before i touch our codebase" — background on ≠ company research. Tricky keyword collision on "background on"
9. "research best practices for writing cold emails that don't get marked as spam" — meta research, no company
10. "can you dig into ramp's engineering blog and tell me how they structure their data pipeline? im studying for a system design interview" — touches "ramp" + "dig into" but intent is learning, not outreach. Hardest near-miss

## Notes on edge cases

The hardest cases are #6, #8, and #10 in the should-NOT-trigger set — they share strong surface features (company name, "background on", "research X") with the should-trigger set. The description's outreach-intent anchor is what separates them. If a routing failure happens, it'll likely be on one of these three.

## How to run these

Run the description optimizer against this test set:

```
scripts/run_loop.py --skill skills/prospect-brief --test-queries skills/prospect-brief/test-queries.md
```

(Requires skill-creator installed; see Anthropic's skills repo for setup.)
