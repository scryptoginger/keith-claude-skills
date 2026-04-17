# merlin-log-triage: Skill Context Artifact

Everything needed to write the SKILL.md. Based on real MERLIN execution logs.

---

## 1. Sample MERLIN Log Excerpt

This is a redacted, condensed excerpt from a real MERLIN execution log. It contains one full Master Cycle plus the start of an Hourly Tech Cycle, with 5 distinct anomalies seeded across it.

```
[2026-04-01 12:17:25 UTC] [RECONCILER] Starting transaction reconciliation sweep...
[2026-04-01 12:17:25 UTC] [RECONCILER] Fetching transactions since ID 4 (history=0 closes, open=0 positions)
[2026-04-01 12:17:25 UTC] [RECONCILER] No new transactions since ID 4. Books are clean.
[MERLIN] Initiating Master Live Cycle for His Majesty...

==================================================
EXECUTING: /home/user/merlin/sync_broker.py
==================================================
[SYNC] Synchronizing MERLIN Blackboard with OANDA Live Environment...
[SYNC] No open positions. Sync complete.
[MERLIN] Cleared pending signals from Blackboard for fresh run.

==================================================
EXECUTING: /home/user/merlin/agents/oracle/run_oracle.py
==================================================
[ORACLE] Polling Alpha Vantage for macro sentiment...
  Alpha Vantage: 50 articles
[ORACLE] Fetching RSS intelligence from multiple sources...
[RSS] Failed to fetch Reuters Business: <urlopen error [Errno -2] Name or service not known>
  Reuters Business: 0 articles (offline or filtered)
  FXStreet: 8 relevant articles
  Yahoo Finance: 4 relevant articles
  Investing.com Forex: 8 relevant articles
  MarketWatch: 2 relevant articles
[ORACLE] Total RSS articles aggregated: 22
[ORACLE/X] Missing XAI_API_KEY.
[ORACLE/X] Missing XAI_API_KEY.
[ORACLE/X] Missing XAI_API_KEY.
[ORACLE] Total intelligence: 50 AV + 25 RSS = 75 articles
[JOURNAL] Cycle f9497554 started
[CALENDAR] Failed to fetch https://nfs.faireconomy.media/ff_calendar_nextweek.json: HTTP Error 404: Not Found
[CALENDAR] Fetched 0 relevant events from API.
[ORACLE] Calendar context injected.
  EUR/USD: 62.5% retail LONG -- mild bearish lean
  GBP/USD: Balanced (59.1% long / 40.9% short)
  USD/JPY: 64.4% retail LONG -- mild bearish lean
  EUR/GBP: 61.6% retail LONG -- mild bearish lean
  AUD/USD: 62.6% retail LONG -- mild bearish lean
  USD/CHF: 62.0% retail LONG -- mild bearish lean
  USD/CAD: 63.2% retail LONG -- mild bearish lean
[ORACLE] OANDA retail sentiment injected.
[COT] Fetching CFTC data...
[COT] No data parsed -- CFTC format may have changed
[ORACLE] COT data injected.
[MACRO] Fetching macro indicators...
[MACRO] Fear&Greed:8 (Extreme Fear) | VIX:24.18 | 10Y:4.311% | DXY:99.41
[ORACLE] Macro context injected (F&G:8 VIX:24.18 DXY:99.41).
[ORACLE] Transmitting aggregated intelligence to Gemini Flash for synthesis...
[ORACLE] Gemini error: HTTP Error 400: Bad Request
[MERLIN] The Oracle has synthesized multi-source intelligence to the blackboard.
[JOURNAL] Oracle deliberation recorded (75 articles, 0 pair verdicts).

==================================================
EXECUTING: /home/user/merlin/agents/augur/augur_engine.py
==================================================
[CORRELATOR] Starting macro correlation analysis...
  Fetching DX-Y.NYB (dxy)...
     dxy: 99.416 (-0.544% DOWN)
  Fetching GC=F (gold)...
     gold: 4775.70 (+2.756% UP)
  Fetching CL=F (oil)...
     oil: 98.34 (-2.999% DOWN)
  Fetching ^TNX (yield_10y)...
     yield_10y: 4.311 (-0.714% DOWN)

[CORRELATOR] Computing per-pair biases...
  EUR/USD: BULLISH (strength: 78)
  GBP/USD: BULLISH (strength: 73)
  USD/JPY: BEARISH (strength: 17)
  AUD/USD: BULLISH (strength: 63)
  USD/CHF: BEARISH (strength: 17)
  USD/CAD: BEARISH (strength: 39)

[CORRELATOR] Macro Regime: NEUTRAL
[CORRELATOR] Blackboard updated.
[CORRELATOR] Done.

==================================================
EXECUTING: /home/user/merlin/agents/technician/technician_engine.py
==================================================
[TECHNICIAN] Loading theory nodes...
[TECHNICIAN] No Theory class found in donchian_breakout/theory.py
[TECHNICIAN] No Theory class found in event_horizon/theory.py
[TECHNICIAN] 9 theory nodes loaded: ['rsi_reversal', 'macd_crossover', 'bollinger_bounce', 'ema_trend', 'support_resistance', 'mean_reversion', 'session_momentum', 'news_fade', 'seer_ml']
[TECHNICIAN] Fetching H1 market data for EUR/USD...
  support_resistance: EUR/USD -> SHORT (73%)
  seer_ml: EUR/USD -> SHORT (54%)
[MTF] Running multi-timeframe analysis for EUR/USD...
[MTF] EUR/USD: FLAT @ 0% | MTF BLOCKED -- H4+D1 both oppose SHORT (H4:BULLISH D1:BULLISH ADX:50.2)
  [AUGUR] EUR/USD: AUGUR:off
  EUR/USD consensus: FLAT | Conf: 0% | Agreed: ['support_resistance', 'seer_ml']
[TECHNICIAN] Fetching H1 market data for EUR/GBP...
  bollinger_bounce: EUR/GBP -> LONG (79%)
  ema_trend: EUR/GBP -> LONG (74%)
  support_resistance: EUR/GBP -> LONG (90%)
  mean_reversion: EUR/GBP -> LONG (73%)
  session_momentum: EUR/GBP -> SHORT (85%)
[MTF] Running multi-timeframe analysis for EUR/GBP...
[MTF] EUR/GBP: FLAT @ 0% | FLAT
  [AUGUR] EUR/GBP: AUGUR:off
  EUR/GBP consensus: FLAT | Conf: 0% | Agreed: []
[TECHNICIAN] Fetching H1 market data for USD/CAD...
  ema_trend: USD/CAD -> SHORT (62%)
  support_resistance: USD/CAD -> LONG (81%)
  seer_ml: USD/CAD -> LONG (52%)
[MTF] Running multi-timeframe analysis for USD/CAD...
[MTF] USD/CAD: LONG @ 64.2% | H4:NEUTRAL D1:BULLISH ADX:35.2
  [AUGUR] USD/CAD: AUGUR:neutral(39)
  USD/CAD consensus: LONG | Conf: 64.2% | Agreed: ['support_resistance', 'seer_ml']
[MERLIN] The Technician has successfully written theory-aggregated signals to the blackboard.

==================================================
EXECUTING: /home/user/merlin/signal_logger.py
==================================================
[SIGNAL LOGGER] Scan #1 logged to 2026-04-01.json (8 signals)

==================================================
EXECUTING: /home/user/merlin/agents/shield/shield_engine.py
==================================================
[2026-04-01 16:00:40 UTC] [RECONCILER] Starting transaction reconciliation sweep...
[2026-04-01 16:00:40 UTC] [RECONCILER] Fetching transactions since ID 4 (history=0 closes, open=0 positions)
[2026-04-01 16:00:40 UTC] [RECONCILER] No new transactions since ID 4. Books are clean.
[SHIELD] Initiating Risk Assessment & Confluence Checks...
[SHIELD] Daily drawdown limit intact (Current: 0.0%).
[SHIELD] Active trades count is safe (0/5).
[SHIELD] Calendar Guard loaded.
[SIGNAL INTEL] USD/CAD LONG streak: 9 scans | Confidence boosted 64.2 -> 79.2 (+15)
[SHIELD] Analyzing USD/CAD LONG | Entry: 1.38922 | SL: 1.38763 | Tech Conf: 79.2
[SHIELD] Dynamic rules -> Min Conf: 70 | Size: 1.0x | TP: 1.0x
[SHIELD] Oracle Data -> Sentiment: 0 | Confidence: 0 | Volatility: Low
[SHIELD/KELLY] Failed to compute Kelly: [Errno 2] No such file or directory: '/home/user/merlin/quant/quant_report.json'
VETO: LONG signal lacks fundamental confluence. Oracle sentiment (0) < +35.
[SIGNAL INTEL] Veto logged for USD/CAD LONG @ 1.38922
[MERLIN] Shield execution complete. Blackboard updated.

==================================================
EXECUTING: /home/user/merlin/broker_execution.py
==================================================
[BROKER] Testing connection to OANDA fxTrade Practice API...
[BROKER] Connection successful! Current Account Balance: $10,000.00
[BROKER] Running execution scan (DRY RUN - Live call commented out)...

==================================================
[MERLIN] Running Transaction Reconciler (catch-all sync)...
[2026-04-01 16:00:41 UTC] [RECONCILER] Starting transaction reconciliation sweep...
[2026-04-01 16:00:41 UTC] [RECONCILER] Fetching transactions since ID 4 (history=0 closes, open=0 positions)
[2026-04-01 16:00:41 UTC] [RECONCILER] No new transactions since ID 4. Books are clean.

==================================================
[MERLIN] Master Cycle Complete. Reviewing final Blackboard state...

Pair: EUR/USD | Direction: FLAT
Confidence: 0%
Status: PENDING_SHIELD_REVIEW
Reasoning: [support_resistance, seer_ml] consensus SHORT. Price 1.16036 at swing resistance 1.16144 (tested 1x).

Pair: USD/CAD | Direction: LONG
Confidence: 64.2%
Status: VETO
Reasoning: USD/CAD is testing a strong support level at 1.39010.
Shield Notes: VETO: LONG signal lacks fundamental confluence. Oracle sentiment (0) < +35.

[MERLIN] Initiating Hourly Tech Cycle...

==================================================
EXECUTING: sync_broker.py
==================================================
[ARCHIVIST] Processing CLOSED_LOSS trade: EUR/GBP LONG
[ARCHIVIST] Transmitting trade history to Claude Haiku for post-mortem analysis...
[ARCHIVIST] Theory attribution: ema_trend -> LOSS $-149.97
[ARCHIVIST] Theory attribution: support_resistance -> LOSS $-149.97
[ARCHIVIST] Theory attribution: mean_reversion -> LOSS $-149.97
[ARCHIVIST] Rules updated for EUR/GBP: conf_min=67 | size_mod=0.9x | tp_mod=0.95x
[ARCHIVIST] Trade archived. Feedback loop updated.
[TELEGRAM] Missing token or chat_id.
[SYNC] Synchronizing MERLIN Blackboard with OANDA Live Environment...
[SYNC] Position EUR/GBP is NO LONGER OPEN on OANDA. Fetching closure details...
[JOURNAL] Orphan outcome saved for EUR/GBP (no matching cycle found).
[SYNC] Telegram close notify failed: DiscordNotifier.send_trade_alert() got an unexpected keyword argument 'entry'
[SYNC] EUR/GBP Closed. PnL: $-149.97 | Status: CLOSED_LOSS
STDERR:
Traceback (most recent call last):
  File "/home/user/merlin/agents/archivist/archivist_engine.py", line 304, in <module>
    main()
  File "/home/user/merlin/agents/archivist/archivist_engine.py", line 283, in main
    notify_trade_close(
  File "/home/user/merlin/agents/archivist/../../telegram_notifier.py", line 140, in notify_trade_close
    discord.send_trade_alert(pair, direction, entry=None, sl=None, tp=None, pnl=pnl, action="CLOSE")
TypeError: DiscordNotifier.send_trade_alert() got an unexpected keyword argument 'entry'
```

### Sample VIGIL Excerpt (Synthetic)

No VIGIL logs exist on disk. VIGIL is referenced in the MERLIN architecture as a monitoring/watchdog agent. This is a plausible synthetic excerpt based on the MERLIN conventions:

```
[2026-04-01 17:00:00 UTC] [VIGIL] Watchdog cycle starting...
[2026-04-01 17:00:01 UTC] [VIGIL] Checking agent health: ORACLE...
[2026-04-01 17:00:01 UTC] [VIGIL] ORACLE last heartbeat: 2026-04-01 16:00:35 UTC (59m26s ago) -- OK
[2026-04-01 17:00:01 UTC] [VIGIL] Checking agent health: TECHNICIAN...
[2026-04-01 17:00:01 UTC] [VIGIL] TECHNICIAN last heartbeat: 2026-04-01 16:00:38 UTC (59m23s ago) -- OK
[2026-04-01 17:00:02 UTC] [VIGIL] Checking agent health: SHIELD...
[2026-04-01 17:00:02 UTC] [VIGIL] SHIELD last heartbeat: 2026-04-01 16:00:41 UTC (59m21s ago) -- OK
[2026-04-01 17:00:02 UTC] [VIGIL] Checking broker connectivity...
[2026-04-01 17:00:02 UTC] [VIGIL] OANDA API: 200 OK (latency: 142ms)
[2026-04-01 17:00:03 UTC] [VIGIL] Checking blackboard staleness...
[2026-04-01 17:00:03 UTC] [VIGIL] WARNING: Blackboard last_updated 2026-04-01 12:26:42 UTC -- stale (4h33m)
[2026-04-01 17:00:03 UTC] [VIGIL] Checking drawdown...
[2026-04-01 17:00:03 UTC] [VIGIL] Current drawdown: -3.09% (limit: -5.0%) -- OK
[2026-04-01 17:00:04 UTC] [VIGIL] Cycle complete. 1 warning(s), 0 critical(s).
```

---

## 2. Log Format Spec

**Format:** Plain text, line-oriented. Not JSON lines. Not mixed. Pure plain text with structural conventions.

**Timestamp format:** Two patterns coexist:
- **Timestamped lines:** `[YYYY-MM-DD HH:MM:SS UTC]` -- used by RECONCILER, VIGIL, and some SHIELD entries. Always UTC.
- **Non-timestamped lines:** The majority of lines. No leading timestamp. Time can only be inferred from the surrounding timestamped lines or the cycle header context.

**Log level conventions:** There are no explicit log levels (no `INFO`, `WARN`, `ERROR` keywords). Instead, severity is conveyed by:
- Emoji prefix in the raw log (removed in the sample above for portability, but present in actual files)
- Keyword patterns: `Failed`, `error:`, `VETO:`, `STDERR:`, `Traceback`, `Missing`, `No data parsed`
- The Skill should infer severity from content, not from a level field.

**Component identifiers:** Always in square brackets: `[COMPONENT_NAME]`. Known components:

| Component | Role |
|-----------|------|
| `MERLIN` | Master orchestrator, cycle lifecycle |
| `RECONCILER` | Transaction reconciliation with broker |
| `SYNC` | Broker state synchronization |
| `ORACLE` | News/sentiment intelligence gathering |
| `ORACLE/X` | X/Twitter sentiment sub-module |
| `AUGUR` / `CORRELATOR` | Macro correlation engine |
| `TECHNICIAN` | Technical analysis, theory node aggregation |
| `MTF` | Multi-timeframe analysis filter |
| `SHIELD` | Risk management, confluence checks |
| `SHIELD/KELLY` | Kelly criterion position sizing |
| `SIGNAL INTEL` | Signal streak tracking |
| `SIGNAL LOGGER` | Signal persistence |
| `BROKER` | Order execution |
| `ARCHIVIST` | Post-mortem trade analysis, feedback loop |
| `CALENDAR` | Economic calendar events |
| `MACRO` | Macro indicators (F&G, VIX, DXY) |
| `COT` | CFTC Commitments of Traders data |
| `RSS` | RSS feed ingestion |
| `JOURNAL` | Decision journal logging |
| `TELEGRAM` | Notification delivery |
| `VIGIL` | Watchdog / health monitor |

**Structural markers the Skill should look for:**
- **Cycle start:** Lines containing `Initiating Master Live Cycle` or `Initiating Hourly Tech Cycle`
- **Section separators:** `==================================================` followed by `EXECUTING: <script_path>`
- **Cycle end:** `Master Cycle Complete. Reviewing final Blackboard state...`
- **Blackboard dump:** Blocks of `Pair: <PAIR> | Direction: <DIR>` with `Confidence:`, `Status:`, `Reasoning:`, `Shield Notes:` on subsequent lines
- **Trade events:** `Order Filled. OANDA Transaction ID: <N>`, `Closed. PnL: $<amount> | Status: CLOSED_LOSS|CLOSED_WIN`
- **STDERR blocks:** `STDERR:` followed by Python tracebacks

---

## 3. Normal vs. Anomaly Baseline

### What a healthy MERLIN run looks like

A healthy Master Cycle proceeds through these stages in order, with each completing successfully:

1. **RECONCILER sweep** -- ends with `No new transactions` or `Sweep complete`
2. **SYNC** -- ends with `Sync complete` or `Synchronization complete`
3. **ORACLE** -- fetches articles from multiple sources, synthesizes via Gemini, ends with `The Oracle has synthesized`
4. **AUGUR/CORRELATOR** -- fetches macro instruments (DXY, gold, oil, 10Y yield), computes per-pair biases, reports `Macro Regime`
5. **TECHNICIAN** -- loads theory nodes (all load successfully), fetches H1 data per pair, runs MTF analysis, writes to blackboard
6. **SIGNAL LOGGER** -- logs scan with signal count
7. **SHIELD** -- reconciler sweep, drawdown check, active trades check, analyzes each non-FLAT signal, either APPROVED or VETO with reason
8. **BROKER** -- connection test succeeds, executes approved trades or reports no action
9. **Catch-all RECONCILER** -- final sweep
10. **Blackboard dump** -- all pairs listed, statuses are `OPEN`, `PENDING_SHIELD_REVIEW`, or `VETO`

**Healthy markers:**
- Oracle produces >0 pair verdicts: `Oracle deliberation recorded (75 articles, 8 pair verdicts)`
- Oracle sentiment is non-zero on the Shield review: `Sentiment: 50 | Confidence: 65`
- Theory nodes all load (no `No Theory class found` warnings, or they're known/expected)
- No `STDERR:` or `Traceback` blocks
- Broker connection succeeds
- No `Failed to compute Kelly` errors

### What indicates something is off

| Pattern | What it means |
|---------|---------------|
| `Gemini error: HTTP Error 400/503` | Oracle LLM synthesis failed -- downstream, all sentiment will be 0, causing all signals to get VETO'd for "lacks fundamental confluence" |
| `0 pair verdicts` in Journal line | Confirms Oracle produced no actionable output |
| `Oracle sentiment (0) < +35` repeated across all pairs | Systemic -- Oracle is broken, not pair-specific |
| `STDERR:` + `Traceback` | Unhandled Python exception -- something crashed but the orchestrator continued |
| `Failed to compute Kelly: No such file or directory` | Missing prerequisite file, non-fatal but degrades position sizing |
| `Missing XAI_API_KEY` repeated 3x | Config gap -- X/Twitter sentiment module is completely disabled |
| `No Theory class found in <name>/theory.py` | Theory node failed to load -- reduces signal quality |
| `Local Qwen reasoning synthesis failed: Connection refused` | Local LLM server is down |
| `VETO: Maximum active trades limit reached (5/5)` | System is saturated -- no new trades possible regardless of signal quality |
| `Correlation Guard triggered. Net LONG exposure to EUR exceeds limit` | Over-concentration risk detected |

---

## 4. Anomaly Taxonomy

### 4.1 API Failure / Rate Limit

**Description:** An external API call returns an error, times out, or returns unparseable data.

**Log signatures:**
- `Gemini error: HTTP Error 400: Bad Request`
- `Gemini error: HTTP Error 503: Service Unavailable`
- `Failed to fetch Reuters Business: <urlopen error [Errno -2] Name or service not known>`
- `Failed to fetch <url>: HTTP Error 404: Not Found`
- `No data parsed -- CFTC format may have changed`
- `Local Qwen reasoning synthesis failed: <urlopen error [Errno 111] Connection refused>`

**Severity guidance:**
- **CRITICAL** when it's the Gemini/Oracle synthesis (cascading: zeroes out all sentiment, causes systematic VETOs)
- **WARNING** when it's a single data source (RSS feed, calendar, COT) -- degraded but not broken
- **INFO** when the source was already optional or has a cache fallback (e.g., `[MACRO] Loaded from cache`)

### 4.2 Silent Failure (Running But Not Acting)

**Description:** A component reports success but actually produced no useful output. The system continues operating on stale or empty data without raising an error.

**Log signatures:**
- `The Oracle has synthesized multi-source intelligence to the blackboard.` immediately after `Gemini error:` -- the success message is misleading
- `Oracle deliberation recorded (75 articles, 0 pair verdicts)` -- 75 articles in, 0 verdicts out
- `COT data injected.` immediately after `No data parsed` -- injected empty data
- `Oracle Data -> Sentiment: 0 | Confidence: 0` on every pair in the same cycle -- systemic zero, not genuine neutral

**Severity guidance:**
- **CRITICAL** when it causes cascading downstream effects (e.g., Oracle silent failure causes all trades to be VETO'd)
- **WARNING** when it affects one data stream but others compensate

### 4.3 Missing Configuration / Auth

**Description:** An environment variable, API key, or file is missing, disabling a subsystem.

**Log signatures:**
- `Missing XAI_API_KEY.` (repeated per pair or per call)
- `[TELEGRAM] Missing token or chat_id.`
- `Failed to compute Kelly: [Errno 2] No such file or directory: '<path>'`

**Severity guidance:**
- **WARNING** when it disables a non-critical subsystem (X sentiment, Telegram notifications, Kelly sizing)
- **CRITICAL** if it were to affect the broker connection or core trading path (not yet observed, but encode the rule)

### 4.4 Stalled Loop / System Saturation

**Description:** The system is running cycles but structurally cannot take action -- either because it's at max capacity or because a dependency is perpetually broken.

**Log signatures:**
- `VETO: Maximum active trades limit reached (5/5). System paused until a position closes.` -- Shield immediately vetoes everything without analysis
- Multiple consecutive cycles where every signal gets the same VETO reason (e.g., `Oracle sentiment (0) < +35` across 3+ Master Cycles in a row)
- Reconciler running repeatedly with `No new transactions` while open positions exist but no close events come through

**Severity guidance:**
- **WARNING** for max trades reached (working as designed, but operator should know)
- **CRITICAL** for systematic VETO due to broken Oracle persisting across 3+ cycles (the system is live but effectively offline)
- **INFO** for reconciler-only cycles with no new data (normal during quiet periods)

### 4.5 Decision Logic Divergence

**Description:** The system's multi-layer decision pipeline produces contradictory or counterintuitive results that warrant human review.

**Log signatures:**
- `MTF BLOCKED -- H4+D1 both oppose <direction>` -- lower timeframe says one thing, higher timeframes disagree
- `Correlation Guard triggered. Net LONG exposure to EUR exceeds limit of 2. Over-leveraged.` -- risk guard overrides a high-confidence signal
- A pair with 4 theories agreeing LONG (80%+) but consensus is FLAT because of MTF or session_momentum opposition
- `AUGUR:neutral` combined with `CORRELATOR` showing strong bias in the opposite direction of the technician signal

**Severity guidance:**
- **INFO** for MTF blocks (this is the filter working correctly -- report it but don't alarm)
- **WARNING** for correlation guard vetoes (position concentration issue)
- **WARNING** when high-confidence signals (>85%) are consistently vetoed across cycles (potential tuning issue)

### 4.6 Crash / Unhandled Exception

**Description:** A Python traceback or unhandled exception appears in the log, indicating a code defect.

**Log signatures:**
- `STDERR:` followed by `Traceback (most recent call last):`
- `TypeError:`, `KeyError:`, `AttributeError:`, `FileNotFoundError:` at the end of a traceback
- `DiscordNotifier.send_trade_alert() got an unexpected keyword argument` (specific known bug)

**Severity guidance:**
- **CRITICAL** if the traceback is in `broker_execution.py`, `shield_engine.py`, or `sync_broker.py` (core trading path)
- **WARNING** if the traceback is in notification/logging code (`telegram_notifier.py`, `archivist_engine.py`) -- trades still execute, but audit trail is broken
- Always CRITICAL if the traceback occurs and the cycle does not continue (check if subsequent section headers appear)

### 4.7 Connection Drop / Auth Failure

**Description:** Broker or critical service connection fails.

**Log signatures:**
- `[BROKER] Connection` not followed by `successful` on the next line
- `urlopen error [Errno 111] Connection refused` for local services (Qwen)
- `urlopen error [Errno -2] Name or service not known` for external services (DNS failure)
- Any HTTP 401/403 error

**Severity guidance:**
- **CRITICAL** for broker connection failure (cannot execute trades or sync state)
- **WARNING** for external data source connection failures (degraded intelligence)
- **WARNING** for local LLM connection refused (Qwen) -- falls back to other synthesis methods

### 4.8 Unexpected Position Change

**Description:** A position appears or disappears from the broker without the system having initiated or expected it.

**Log signatures:**
- `Position <PAIR> is NO LONGER OPEN on OANDA. Fetching closure details...` when no SL/TP was expected to trigger
- `Orphan outcome saved for <PAIR> (no matching cycle found)` -- a close event that doesn't match any recorded open cycle
- Reconciler `closes added: N` when the system had no record of pending closes

**Severity guidance:**
- **CRITICAL** always -- this means the system's internal state diverged from broker reality. Could indicate manual intervention, unexpected SL hits, or reconciliation bugs.

---

## 5. Output Schema Specifics

### Proposed Markdown output structure:

```markdown
# MERLIN Log Triage Report

**Log file:** `<filename>`
**Lines analyzed:** <N>
**Time range:** <first_timestamp> to <last_timestamp>
**Cycles detected:** <N> Master, <N> Hourly

## Summary

<2-4 sentence narrative: what happened in this log. Was it a healthy run? How many anomalies? What's the overall system state? Did trades execute?>

## Anomalies

### [CRITICAL] <short title>
- **Category:** <taxonomy category from section 4>
- **Lines:** <line numbers or timestamp range>
- **Evidence:**
  ```
  <quoted log line(s), max 3 lines>
  ```
- **Explanation:** <1-2 sentences: what this means, why it matters, any cascading effects>

### [WARNING] <short title>
...

### [INFO] <short title>
...

## Recommended Actions

| # | Action | Rationale | Urgency |
|---|--------|-----------|---------|
| 1 | <specific action> | <why> | Immediate / Next session / Monitor |
| 2 | ... | ... | ... |
```

### Schema sanity check against the sample log

The sample log in Section 1 contains these anomalies that the schema must capture:

1. **[CRITICAL] Oracle Gemini synthesis failed** -- Category: API Failure. Evidence: `Gemini error: HTTP Error 400`. Cascading effect: all sentiment=0, all signals VETO'd. This fits the schema.
2. **[CRITICAL] Silent failure: Oracle reported success despite failure** -- Category: Silent Failure. Evidence: `The Oracle has synthesized...` immediately after `Gemini error:` plus `0 pair verdicts`. Two lines of evidence needed. The schema's 3-line evidence cap handles this.
3. **[WARNING] Missing XAI_API_KEY (3 occurrences)** -- Category: Missing Config. Evidence: `Missing XAI_API_KEY.` The Skill should group repeated identical warnings into one anomaly entry with an occurrence count, not emit 3 separate entries. **Add field:** `Occurrences: <N>` for deduplicated warnings.
4. **[WARNING] Kelly criterion file missing** -- Category: Missing Config. Evidence: `Failed to compute Kelly: [Errno 2] No such file or directory`. Fits the schema.
5. **[WARNING] Archivist traceback (TypeError in notifier)** -- Category: Crash. Evidence: the traceback block. Needs multi-line evidence. The 3-line cap should quote the final 3 lines (the call site, the offending line, and the TypeError).
6. **[INFO] RSS source offline (Reuters)** -- Category: Connection Drop. Fits.
7. **[INFO] Calendar API 404** -- Category: API Failure. Fits.
8. **[INFO] COT data unparseable** -- Category: API Failure. Fits.

**Schema adjustment needed:** Add an `Occurrences` field (optional, only shown when >1). Otherwise the schema works as designed.

### Revised anomaly entry:

```markdown
### [SEVERITY] Short Title
- **Category:** <taxonomy name>
- **Occurrences:** <N> *(only if >1)*
- **Lines:** <line numbers or timestamp range>
- **Evidence:**
  ```
  <quoted log line(s), max 3 lines>
  ```
- **Explanation:** <1-2 sentences>
```

---

## 6. Generic-Log-Mode Notes

When the input is not a MERLIN log (e.g., Jenkins console output, syslog, application log), the Skill should:

### Detection

Determine the log type by checking the first 20 lines for MERLIN markers:
- If any of `[MERLIN]`, `[ORACLE]`, `[TECHNICIAN]`, `[SHIELD]`, `Master Live Cycle`, `[RECONCILER]` appear -> MERLIN mode
- Otherwise -> Generic mode

### Taxonomy applicability in Generic mode

| MERLIN Category | Generic Mode Equivalent |
|----------------|------------------------|
| API Failure | **Applies.** Look for HTTP error codes, connection refused, timeout, DNS failures |
| Silent Failure | **Skip.** Too domain-specific to detect generically |
| Missing Config | **Applies.** Look for "not found", "missing", "not set", "undefined" for env vars/files |
| Stalled Loop | **Skip.** Requires domain knowledge of expected cycle cadence |
| Decision Logic Divergence | **Skip.** MERLIN-specific |
| Crash / Unhandled Exception | **Applies.** Look for Python/Java/Node tracebacks, stack traces, `Exception`, `Error`, `FATAL` |
| Connection Drop | **Applies.** Same patterns as API Failure |
| Unexpected Position Change | **Skip.** Trading-specific |

### Generic fallback detectors

Add these for Generic mode only:
- **Build failure:** Lines containing `BUILD FAILED`, `FAILURE`, `build failed`, exit codes != 0, `ERROR:` at start of line
- **Test failure:** `FAILED`, `FAIL`, `failures=`, `errors=`, JUnit/pytest summary lines (`X passed, Y failed`)
- **Stack trace:** Contiguous blocks starting with `Traceback`, `Exception in thread`, `at com.`, `    at `, `Caused by:`
- **Timeout:** `timed out`, `TimeoutError`, `deadline exceeded`, `context deadline`
- **OOM / Resource exhaustion:** `OutOfMemoryError`, `Cannot allocate memory`, `Killed`, `OOMKilled`
- **Permission denied:** `Permission denied`, `Access denied`, `403 Forbidden`, `EACCES`

### Generic output differences

- The `Cycles detected` line in the Summary becomes `Sections detected` (count separator lines or timestamp gaps >5min)
- Anomaly categories use the generic names above instead of MERLIN taxonomy names
- Recommended Actions should be more cautious ("Investigate the stack trace" rather than "Restart the Oracle agent")

---

## 7. Edge Cases the Skill Should Handle

### Empty file
- Output a report with: Summary says "Log file is empty (0 lines). No analysis possible." Anomalies section says "None." Recommended Actions says "Verify the log path and ensure the process is writing output."

### Log truncated mid-line
- Detect: last line does not end with a newline, or ends mid-word/mid-bracket.
- Action: Note in Summary: "Log appears truncated at line <N>." Analyze everything up to the last complete line. Add a WARNING anomaly: "Log truncation detected" with category "Data Integrity" and urgency "Immediate" (the logging pipeline itself may be broken).

### Log with no anomalies
- Do not manufacture problems. Output: Summary says "Clean run. No anomalies detected across <N> lines / <N> cycles." Anomalies section says "None detected." Recommended Actions says "No action required."

### Log too large to fully process
- Strategy: **Head + Tail + Sampled middle.**
  - Read the first 500 lines (captures cycle start, initial state).
  - Read the last 500 lines (captures final state, most recent errors).
  - If the file is >2000 lines, scan the middle for lines matching anomaly keywords only (grep-style: `error`, `fail`, `traceback`, `STDERR`, `VETO`, `refused`, `timeout`, `Missing`).
  - Note in Summary: "Large log (<N> lines). Full analysis covers first/last 500 lines; middle was keyword-scanned."
  - Never claim comprehensive analysis of a partially-read log.

### Log in unexpected format
- If neither MERLIN markers nor recognizable log patterns (timestamps, log levels, stack traces) are found in the first 50 lines:
  - Output Summary: "Unrecognized log format. Unable to perform structured triage."
  - Attempt a best-effort keyword scan for universal error indicators: `error`, `fail`, `exception`, `critical`, `fatal`.
  - If any are found, list them as generic anomalies with line references.
  - Recommended Action: "Provide a MERLIN execution log or standard application log for full analysis."

### Binary / non-text file
- If the content appears to be binary (non-printable characters in the first 100 bytes): "This does not appear to be a text log file. No analysis performed."

### Extremely rapid repeated cycles
- If multiple Master Cycles appear within <2 minutes of each other (when timestamps are available), flag as WARNING: "Unusually rapid cycle frequency detected" -- may indicate a restart loop or misconfigured cron.
