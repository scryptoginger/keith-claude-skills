---
name: merlin-log-triage
description: Use this skill whenever someone hands over a MERLIN execution log (or VIGIL watchdog log) and needs a fast read on what happened — for post-cycle review, debugging a failed run, investigating why no trades executed, or daily operator-style triage. Trigger on phrases like "triage this log," "what went wrong in this run," "analyze MERLIN output," "post-mortem on the cycle," or any time a `.log` / `.txt` file path is provided alongside MERLIN-domain context (Oracle, Shield, Technician, Augur, blackboard, cycle, VETO). Recency window: a single log file or a contiguous excerpt — not historical trend analysis across many runs. Not for live trading decisions, fixing the underlying issues, restarting agents, or general application logs unrelated to MERLIN/VIGIL infrastructure.
---

# MERLIN Log Triage

This skill turns a MERLIN execution log into a structured triage report in under a minute. The goal is operator-style situational awareness: *what happened in this cycle, what's broken, and what to do about it* — not exhaustive forensics. The audience is the operator (me) reviewing a cycle on a phone or in a terminal, deciding whether to intervene now, next session, or not at all.

## Inputs Contract

The required input is a path to a log file, OR a pasted log excerpt of at least ~20 lines. If neither is provided, request one — do not attempt triage on conversational descriptions of what happened.

If a file path is given, read the file directly. If the file is very large (>2000 lines), apply the head/tail/keyword-scan strategy described in **Judgment guidance** rather than refusing.

If the input is clearly not a MERLIN/VIGIL log (no `[MERLIN]`, `[ORACLE]`, `[TECHNICIAN]`, `[SHIELD]`, `[RECONCILER]`, or `Master Live Cycle` markers in the first 20 lines), drop into Generic Log mode — described in **Judgment guidance** — rather than forcing the MERLIN taxonomy onto it.

This skill reads logs only. It does not connect to OANDA, restart agents, modify configuration, or take any action on the trading system. Triage only.

## What you produce

A structured report with four sections, in this exact order. The report should be readable end-to-end in under 60 seconds.

### 1. Header

* **Log file** — filename or `(pasted excerpt)`
* **Lines analyzed** — total line count
* **Time range** — first to last timestamp found (UTC)
* **Cycles detected** — count of Master Cycles and Hourly Tech Cycles

### 2. Summary (2–4 sentences)

A narrative answer to "what happened in this log." Cover:

* Did the cycle(s) complete or stall?
* Were any trades opened, closed, or vetoed?
* Overall system state — healthy, degraded, or broken?
* Single most important thing the operator should know

This is the section that gets read first and sometimes only. Make it count.

### 3. Anomalies

Each anomaly is a structured entry. Order anomalies by severity (CRITICAL → WARNING → INFO), then by line number within each severity tier.

```markdown
### [SEVERITY] Short title
- **Category:** <taxonomy name>
- **Occurrences:** <N>  *(only include this line if N > 1)*
- **Lines:** <line numbers or timestamp range>
- **Evidence:**
  ```
  <quoted log line(s), max 3 lines>
  ```
- **Explanation:** <1–2 sentences: what this means and why it matters, including any cascading effects>
```

If no anomalies are detected, output: *"None detected. Clean run."* Do not manufacture problems.

**Anomaly taxonomy** (use these exact category names):

* **API Failure / Rate Limit** — external API errors, timeouts, unparseable responses
* **Silent Failure** — component reports success but produced no useful output (e.g., Oracle synthesizes after Gemini errored, "0 pair verdicts")
* **Missing Configuration** — env vars, API keys, prerequisite files missing
* **Stalled Loop / Saturation** — system running but structurally cannot act (max trades reached, systematic VETO across cycles)
* **Decision Logic Divergence** — MTF blocks, correlation guard vetoes, contradictory layer outputs
* **Crash / Unhandled Exception** — Python tracebacks, unhandled errors
* **Connection Drop** — broker connection failure, DNS failures, connection refused
* **Unexpected Position Change** — position appears or disappears without expected cycle, orphan outcomes

Group repeated identical events into a single entry with `Occurrences: <N>`. Do not emit three separate entries for `Missing XAI_API_KEY` appearing three times in one cycle.

### 4. Recommended Actions

A table, ordered by urgency:

```markdown
| # | Action | Rationale | Urgency |
|---|--------|-----------|---------|
| 1 | <specific, actionable step> | <why> | Immediate |
| 2 | ... | ... | Next session |
| 3 | ... | ... | Monitor |
```

Urgency tiers:

* **Immediate** — blocks trading or indicates the system's view of reality has diverged from broker reality
* **Next session** — should be fixed before the next Master Cycle, but not actively dangerous right now
* **Monitor** — no action required yet, but watch for recurrence

If no actions are needed: *"No action required."*

## Output format

Markdown only. Render the full report inline in the chat — do not write a file unless explicitly asked. The report should render cleanly in both a terminal (via `glow`, `bat`, etc.) and in a chat client. Keep code fences small. Do not pad the report with horizontal rules between every section.

## Judgment guidance

**On severity calibration.** The single most important judgment call is distinguishing *cosmetic noise* from *cascading systemic failure*. A missed RSS feed is INFO. A failed Gemini call is CRITICAL because it zeros out Oracle sentiment, which causes Shield to VETO every signal in the cycle, which means the system is "live" but effectively offline. When you see a low-level error, ask: *what does this break downstream?* If it cascades, escalate the severity.

**On silent failures.** These are the most dangerous and the easiest to miss. The pattern: a component logs an error, then logs success. *Both lines are true on their own and false in combination.* Always cross-reference success messages with the lines immediately preceding them. `The Oracle has synthesized` after `Gemini error: HTTP Error 400` is not a healthy line — it's a silent failure with two pieces of evidence.

**On grouping.** If the same anomaly appears 3+ times in one cycle (e.g., `Missing XAI_API_KEY` repeated per pair), emit one entry with `Occurrences: <N>`. If the same anomaly appears across multiple cycles in the log, group by cycle and note the cross-cycle pattern in the explanation — this often promotes the severity (one-time API blip vs. systemic outage).

**On clean runs.** A log with no anomalies is a real and valid output. Do not manufacture concerns to fill the section. *"None detected. Clean run."* is the correct output when warranted, and it's actually informative — it tells the operator the cycle is healthy.

**On large logs.** For files >2000 lines: read the first 500 lines (initial state + first cycle), the last 500 lines (final state + most recent errors), and keyword-scan the middle for: `error`, `fail`, `traceback`, `STDERR`, `VETO`, `refused`, `timeout`, `Missing`. Note in the Summary: *"Large log (N lines). Full analysis covers first/last 500 lines; middle was keyword-scanned."* Never claim comprehensive analysis of a partially-read log.

**On truncation.** If the last line is incomplete or doesn't end with a newline, note in the Summary: *"Log appears truncated at line N."* Add a WARNING anomaly with category "Crash / Unhandled Exception" — the logging pipeline itself may be broken, which is the kind of failure that hides every other failure.

**On Generic Log mode.** When the input has no MERLIN markers, fall back to a reduced taxonomy: API Failure, Missing Configuration, Crash / Unhandled Exception, Connection Drop. Add generic detectors for: build failures (`BUILD FAILED`, exit codes ≠ 0), test failures (`FAILED`, `failures=N`), stack traces (Python `Traceback`, Java `at com.`), timeouts, OOM, and permission errors. Use generic recommended actions (*"Investigate the stack trace at line N"*) rather than MERLIN-specific ones (*"Restart the Oracle agent"*). Note in the header: `Mode: Generic (no MERLIN markers detected)`.

**On binary or unrecognizable input.** If the file appears binary or has no recognizable log structure in the first 50 lines, do not force a triage. Output: *"Input does not appear to be a text log file"* or *"Unrecognized log format — unable to perform structured triage,"* with a best-effort keyword scan if any error indicators are found.

## Example output

This is a condensed version of a real run on a MERLIN execution log containing one Master Cycle and the start of an Hourly Tech Cycle. The full output is in `examples/sample-merlin-cycle-triage.md`.

---

# MERLIN Log Triage Report

**Log file:** `merlin-2026-04-01.log`
**Lines analyzed:** 198
**Time range:** 2026-04-01 12:17:25 UTC → 2026-04-01 16:00:41 UTC
**Cycles detected:** 1 Master, 1 Hourly Tech (partial)

## Summary

The Master Cycle completed structurally but produced no executable trades — Oracle's Gemini synthesis failed (HTTP 400) and the system continued as if it had succeeded, leaving Oracle sentiment at 0 across all pairs. Shield correctly VETO'd the one LONG signal (USD/CAD, 79.2% confidence) for "lacks fundamental confluence," which is the right behavior given the upstream failure but masks the real issue: Oracle is broken, not the market. One closed position (EUR/GBP CLOSED_LOSS, -$149.97) processed cleanly, but the Discord/Telegram notification crashed with a TypeError.

## Anomalies

### [CRITICAL] Oracle Gemini synthesis failed
- **Category:** API Failure / Rate Limit
- **Lines:** 60
- **Evidence:**
  ```
  [ORACLE] Gemini error: HTTP Error 400: Bad Request
  ```
- **Explanation:** Oracle's LLM synthesis call failed. Cascading: all pair sentiments default to 0, which trips Shield's "Oracle sentiment < +35" guard on every signal, vetoing the entire cycle.

### [CRITICAL] Silent failure — Oracle reported success after error
- **Category:** Silent Failure
- **Lines:** 60–62
- **Evidence:**
  ```
  [ORACLE] Gemini error: HTTP Error 400: Bad Request
  [MERLIN] The Oracle has synthesized multi-source intelligence to the blackboard.
  [JOURNAL] Oracle deliberation recorded (75 articles, 0 pair verdicts).
  ```
- **Explanation:** Oracle logged success immediately after the Gemini error. The journal line confirms: 75 articles in, 0 verdicts out. This is the failure pattern most likely to go unnoticed in routine review.

*(Full report continues with WARNING anomalies for the Archivist TypeError and missing Kelly file, INFO anomalies for the Reuters/Calendar/COT data source failures, and a Recommended Actions table. See `examples/sample-merlin-cycle-triage.md`.)*

## What this skill does NOT do

* **Fix the issues.** Triage is read-only. The Skill identifies what's broken; the operator (or another Skill) decides what to do.
* **Restart agents, modify configuration, or take action on the trading system.** Hard line — this Skill never touches the live system.
* **Replace human judgment on live trading.** Recommended Actions are advisory. Anything affecting live positions or capital requires operator review.
* **Fabricate anomalies to fill the report.** A clean run gets a clean report.
* **Cross-cycle trend analysis.** This Skill triages a single log or excerpt. Pattern detection across many runs over time is a different problem.
* **Replace structured monitoring.** This is operator-loop triage, not a substitute for VIGIL watchdog alerts, Prometheus metrics, or proper observability.
