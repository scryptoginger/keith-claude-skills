# Test Queries: merlin-log-triage

A set of queries used to validate the Skill's behavior across expected and edge-case inputs. Run these against the Skill in Claude Code or Claude.ai with Skills enabled. The Skill should invoke on all MERLIN-flavored queries, gracefully handle the edge cases, and NOT invoke on the anti-triggers.

## Expected triggers — MERLIN logs

1. `Triage this MERLIN log: /path/to/merlin_execution.log`
2. `What went wrong in this cycle?` (with a log file attached or referenced)
3. `Run merlin-log-triage on the attached log excerpt.`
4. `Post-mortem on last night's MERLIN run — here's the log.`
5. `I got kicked out of my SSH session and MERLIN has been running without me. Can you check the log and tell me what I missed?`
6. `Why did Shield VETO everything in this cycle?` (with log context)
7. `Is the Oracle working? Here's the log from the last 24 hours.`

## Expected triggers — VIGIL logs

8. `Triage this VIGIL watchdog output.`
9. `Health check on VIGIL — anything flagged?`

## Edge cases — should handle gracefully

10. **Empty file** — `Triage this log: empty.log` → Summary notes empty file, no anomalies, recommends verifying process is writing output.
11. **Log with no anomalies** — clean run, Skill should report "None detected. Clean run." and not manufacture issues.
12. **Very large log (>2000 lines)** — Skill should apply head/tail/keyword-scan strategy and note the partial-analysis disclaimer in the Summary.
13. **Truncated log** — last line incomplete or mid-traceback. Skill should note truncation in Summary and add a WARNING anomaly for data integrity.
14. **Generic application log (no MERLIN markers)** — e.g., a Python app log or syslog excerpt. Skill should drop into Generic mode, note it in the header, and use the reduced taxonomy.
15. **Jenkins console output** — should trigger Generic mode and use build-failure / stack-trace / test-failure detectors.
16. **Binary or unrecognizable file** — Skill should decline structured triage and recommend providing a text log.

## Anti-triggers — should NOT invoke

17. `How do I restart the Oracle agent?` — operational question, not a triage request. Skill should not run; the operator wants instructions, not analysis.
18. `Explain how MERLIN's Shield module works.` — architectural question, no log provided.
19. `What's a good strategy for forex trading?` — domain question unrelated to log triage.
20. `Can you fix this bug in my Python code?` — code review, not log triage, even if the code is from MERLIN.

## Known limitations to verify

21. **Cross-cycle pattern detection** — if the same anomaly appears across multiple cycles in the log, the Skill should group and note the pattern in the explanation (often promoting severity). Verify this works on a log containing 3+ cycles with recurring Gemini failures.
22. **Root-cause chaining** — if one anomaly is downstream of another (e.g., Kelly failure caused by Quant crash), the Skill should note the relationship in the Recommended Actions. Currently this is partial — a future iteration could make it explicit.

## Regression tests

Whenever SKILL.md is edited, re-run at least:
- Query #1 against the sample log in `examples/sample-merlin-cycle-triage.md`'s source (the 15-day production log)
- Query #10 (empty file)
- Query #14 (generic application log — confirms Generic mode still works)

The example file's output should remain reproducible within minor wording variation.
