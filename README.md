# FS Builds 1.3.0 GA + S4

## User manual, settings reference and release record

**Release:** 1.3.0 GA + S4 \| **Language:** Pine Script v6 \| **Platform:** TradingView\
**Reference instrument:** MNQ1! \| **Managed execution/research timeframes:** 5m and 15m\
**Source:** `FS_Builds_1.3.0_GA_S4.pine`\
**Source SHA-256:** `bc5dc1db49e19906fd81b11a3788bc6469db43daa9761b470a0f0411ee0ed718`

This manual describes the frozen GA source and the regression results recorded during development. It is an indicator, not an automated order-execution strategy. Historical modeled results are not brokerage fills or proof of future profitability.

## 1. Quick start

1.  Save a backup of the GA source and keep the original 1.2.0 GA indicator and existing alerts intact.
2.  In TradingView, open Pine Editor, create a separate indicator, paste the full GA source, save and add to chart.
3.  Select MNQ1! and a 5-minute or 15-minute timeframe.
4.  Open Settings \> Inputs. Select the desired Active session and AUTO --- Research Adaptive.
5.  Set Contracts to your intended quantity and Execution round-trip cost (points) to your estimated all-in cost per contract.
6.  Keep the execution planner enabled. Choose the dashboard visibility settings below.
7.  Verify the indicator title, session levels and expected profile. Do not assume existing alerts have migrated to the new script.

## 2. Recommended settings presets

These are recommended workflows, not three built-in presets. The source defaults are separately listed in the appendix.

  -----------------------------------------------------------------------------------------------------------------------------------------------
  Input                                Clean live chart                  Historical research            Forward observation
  ------------------------------------ --------------------------------- ------------------------------ -----------------------------------------
  Active session                       Market being analyzed             New York                       New York

  Trade risk profile                   AUTO --- Research Adaptive        AUTO --- Research Adaptive     AUTO --- Research Adaptive

  Managed Research Setup               Latest signal                     Failed Break or Late OR Fail   Failed Break or Late OR Fail

  Independent manual research          Off                               Off                            Off

  Enable research                      Off                               On                             On

  Show performance metrics             On (inactive when research off)   On                             On

  Track 3-bar and 6-bar SELL MFE/MAE   On                                On                             On

  Show unified SELL session summary    Off                               On                             Optional

  Show execution planner               On                                On                             On

  Show risk / reward zones             On                                On                             On

  Execution round-trip cost (points)   2.0 example                       2.0                            2.0

  Contracts                            Actual intended quantity          1 for reference                Actual intended quantity

  Projected P&L display                Points + net cash                 Points + net cash              Points + net cash

  Signal date from                     Irrelevant with research off      Aug 1, 2026 00:00 ET           Nov 13, 2026 01:30 ET

  Signal date until (exclusive)        Irrelevant with research off      Nov 13, 2026 00:00 ET          Dec 1, 2026 00:00 ET (extend as needed)
  -----------------------------------------------------------------------------------------------------------------------------------------------

The research dates filter signal-close timestamps, not live signal generation. The historical validation window is genuinely out-of-sample only if it was not used for tuning. For prospective observation, record the configuration before collecting new signals.

## 3. Hide dashboards (clean live trading)

### Top right: SELL AUDIT \| SESSION

Settings \> Inputs \> **S4 \| Failed-breakout outcome audit (DEV 0.2.38)**:

-   Uncheck **Show unified SELL session summary**.
-   Keep **Track 3-bar and 6-bar SELL MFE/MAE** on if internal tracking is desired.
-   **Summary table position** can alternatively move the table to any corner.
-   Keep compact audit labels and OR HIGH RECLAIMED diagnostics off for less clutter.

### Bottom right: research performance

Settings \> Inputs \> **S4 \| Advanced research**:

-   Uncheck **Enable research** to remove the research table and disable its simulation.
-   Unchecking **Show performance metrics** alone is not a complete hide control.
-   Re-enable research whenever historical analysis is needed; it recalculates on available chart history.

The planner is separately controlled by **Show execution planner** and **Show risk / reward zones** under **S4 \| On-chart execution planner**. Disabling the research table does not disable signal detection or the planner. Other optional diagnostic tables (ROOM crossing history, OR retest history, Developer Diagnostics) have their own controls.

## 4. Sessions and market structure

  Active session   Default local hours              Timezone
  ---------------- -------------------------------- ------------------
  Sydney           09:00--18:00                     Australia/Sydney
  Tokyo            09:00--18:00                     Asia/Tokyo
  London           08:00--17:00                     Europe/London
  New York         09:30--17:00                     America/New_York
  Custom           09:30--16:00 (fixed in source)   America/New_York

These are script-defined windows, not a claim about official exchange trading hours. IANA timezones handle daylight saving. The separate Reference session (High / Low / POC) defaults to London. Changing Active session changes the opening range and historical signal population; compare only equivalent sessions and settings.

Core chart features include previous session high/low and approximate POC, order blocks (OB), fair value gaps (FVG), 50% equilibrium, areas of interest (AoI), retained 2H support/resistance, selected opening range (OR), and automatic OR Fibonacci 0.500/0.618 levels. POC is an approximation, not exchange volume-at-price data.

**Opening Range Mode:** 30 Minutes by default; 15 Minutes is also available. Keep the tested OR mode and all signal parameters unchanged when reproducing benchmarks. Display controls and detection controls are distinct; turning off a plot is not necessarily disabling its calculation.

## 5. Signals and S4 interpretation

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  Marker / concept                                                  Meaning
  ----------------------------------------------------------------- -----------------------------------------------------------------------------------------------------------------
  WATCH                                                             Candidate interaction/retest; not a confirmed entry

  BUY / SELL                                                        Directional research trigger after applicable confirmation

  OR breakout/retest                                                Break, departure and subsequent qualifying retest

  Delayed wick recovery                                             Later directional close beyond qualifying retest candle within configured window

  SELL FAILED OR HIGH                                               Bearish failure after prior OR-high breakout and exhaustion, strong close below OR high and volume confirmation

  WATCH / SELL LATE OR FAIL                                         Established closes above OR high followed by approach and bearish breakdown confirmation

  ROOM                                                              Informational distance to eligible structural obstacle; does not gate WATCH

  2BAR, stars, engulfing, harami, tweezers, three-inside, H&S/M/W   Pattern context; not all informational markers are execution triggers
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Key defaults: failed-break monitoring 8 bars, upper wick 0.35, bearish body 0.50; late failure requires 6 consecutive closes above OR high, approach 25 points, confirmation window 6 bars, body 0.50, volume or upper-wick confirmation enabled (wick 0.35). OR retest departure 40 ticks; delayed confirmation window 2 bars. These are code defaults, not universally optimal settings.

## 6. Frozen AUTO profiles

  Setup          TF    Profile   Entry                          Preferred target   Evaluation
  -------------- ----- --------- ------------------------------ ------------------ ------------
  Failed Break   5m    A+        50% original-risk pullback     2R                 90m
  Failed Break   15m   B+        37.5% original-risk pullback   1R                 90m
  Late OR Fail   5m    C         Signal close                   2R                 30m
  Late OR Fail   15m   C         Signal close                   1R                 90m

A+, B+, C are execution styles, not signal-quality grades. AUTO is a fixed lookup, not live optimization. Manual profile selection locks the entry style while target/window remain preset-managed. New York was the research reference; Tokyo, London, Sydney and Custom have not received independent profile validation. Other timeframes may display core features, but managed research and planner are limited to 5m/15m.

**Managed Research Setup:** Latest signal (default) follows the latest qualifying setup; Failed Break and Late OR Fail pin the research dashboard. This selector does not filter planner signals or alerts.

**Independent manual research:** Off keeps research aligned with managed presets and greys out advanced manual inputs. On unlocks Display setup, Evaluation minutes, Target model, Entry model, Maximum original risk, Match cohorts, Stop wick buffer, Round-trip cost and Show selected profile only. It changes research, not the planner's profile-managed pricing.

## 7. Execution planner and modeled P&L

The planner models the two S4 bearish setups, not every BUY/SELL marker. For a short setup:

-   Original stop = signal high + 2 points (managed buffer).
-   Original risk = stop − signal close.
-   Entry = signal close + selected fraction of original risk, rounded to tick for the offset (minimum one tick for pullbacks).
-   Actual modeled risk = stop − entry.
-   TP1/TP2/TP3 = entry minus 1R/1.5R/2R respectively, tick-rounded.
-   Net cash = (signed outcome points − round-trip cost) × symbol point value × contracts.

For MNQ, the point value is \$2 per contract; a 0.25-point tick is \$0.50. The code uses the chart symbol's point value. A 2-point cost is \$4 per MNQ contract, not a universal broker commission estimate.

Example: Nov 10 5m Failed Break, entry 27,994, stop 28,023.25, risk 29.25 points, one MNQ, cost 2 points. Preferred 2R target 27,935.50 yields modeled +\$113 if reached. Stop at planned price yields −\$62.50; gaps can worsen loss.

Pending pullback orders can fill, expire, become invalid or encounter a data gap. The planner tracks the latest plan and its independent target alternatives. Do not add the three target P&Ls together. Unfilled entries are not realized losses. The planner does not submit orders, size positions from account equity or guarantee fills. Real fills, slippage, fees and risk may differ.

## 8. Research engine and dashboards

The session audit tracks 3- and 6-bar MFE (maximum favorable excursion), MAE (maximum adverse excursion), favorable threshold (20 points default), OR-high reclaim, queue/accounting and ledger status. On 5m, 3/6 bars mean 15/30 minutes; on 15m, 45/90 minutes. Excursions are not realized profit.

Advanced research evaluates independent hypothetical entry/target combinations. Managed mode uses adaptive entries (close, 25%, 37.5%, 50%) and risk multiples (0.5R, 1R, 1.5R, 2R), displaying the managed selection. Manual fixed-point mode uses entry offsets 0/5/10/15 points and targets 10/15/20/30 points. The 30m and 90m accumulators are separate.

Simulation assumptions verified in source:

-   Confirmed-bar processing; signal close is within the inclusive start/exclusive end filter.
-   Managed matched cohorts require a complete contiguous 90-minute observation even for the 30-minute model.
-   Gaps in the observation window are skipped. A pending pullback opening at or beyond the stop is invalid.
-   Pullback fill candle cannot earn a target; a stop on that candle can count.
-   Stop-first when both stop and target are reached on an already-filled candle; adverse stop gaps use the worse open.
-   Unresolved filled positions exit at evaluation-horizon close; unfilled entries expire.
-   Round-trip cost deducted once per modeled fill.
-   Outcomes may mature after the research end date. The date filter is not a strict outcome cutoff; use a boundary separation and allow full observation before comparing periods.

  -----------------------------------------------------------------------------------------------------------
  Metric                              Interpretation
  ----------------------------------- -----------------------------------------------------------------------
  Seen / gap skips / risk skips       Observed matured signals and exclusions

  Eligible / evaluated                Signals admitted to model evaluation

  Closed fills / fill rate            Filled modeled outcomes and proportion of evaluated signals

  TP hit rate                         Target hits among fills

  Profit factor                       Positive cost-adjusted outcomes divided by absolute negative outcomes

  Net pt/signal                       Total net points divided by evaluated signals, including misses

  Drawdown                            Peak-to-trough modeled cumulative net points in signal order

  Ledger balanced                     Internal accounting reconciliation, not proof of accuracy

  INSUFFICIENT                        Fewer than 10 fills

  WEAK SAMPLE                         10--29 fills

  30+ FILLS                           Initial sample threshold, not statistical proof
  -----------------------------------------------------------------------------------------------------------

### Historical coverage

START TRUNCATED means the first loaded candle is later than the requested start. START COVERED\* means history reaches the start; it does not prove adequate warm-up or continuous coverage. Loaded chart history varies by timeframe, plan and Replay position. Research is recalculated from loaded history, not permanently stored.

## 9. Recorded regression benchmarks

These are historical observations from the development conversation, not a new independent backtest. New York, development date filter Aug 1--Nov 13 exclusive, managed AUTO, 2-point cost.

  -----------------------------------------------------------------------------------------
  Metric            5m Failed Break   15m Failed Break   5m Late OR Fail   15m Late OR Fail
  --------------- ----------------- ------------------ ----------------- ------------------
  Signals seen                   33                 18                12                  5

  Gap skips                       4                  3                 2                  0

  Eligible                       29                 15                10                  5

  Closed fills                   25                 14                10                  5

  Fill rate                  86.21%             93.33%              100%               100%

  Profit factor                1.58               1.75              2.02               3.64

  Net pt/signal               +5.07              +7.45            +19.18             +23.20

  Drawdown (pt)                  73                 68              57.5                 44
  -----------------------------------------------------------------------------------------

Replay checkpoints: Nov 10 Failed Break, Nov 12 Late OR Fail. Advance Replay sufficiently to complete evaluation. Live 5m Late OR Fail had 9 fills, PF 2.86 and +27.39 pt/signal because earlier history was truncated (loaded Aug 9 versus requested Aug 1). The missing outcome is consistent with a 54.75-point loss. This is a sample difference, not improved trading performance. Live 15m Late OR Fail and both Failed Break aggregates reproduced their references.

## 10. Alerts and deployment safety

The source declares the following alert conditions (names exactly as coded):

-   S4 DEV OR Break Retest BUY
-   S4 DEV OR Break Retest SELL
-   S4 2BAR Bull
-   S4 2BAR Bear
-   S4 STRONG Bull
-   S4 STRONG Bear
-   S4 Evening Star
-   S4 Morning Star
-   Bullish OB
-   Bearish OB
-   Bullish FVG
-   Bearish FVG

These are the declared selectable conditions, not a claim that every S4 marker has a dedicated alert. Failed Break and Late OR Fail do not have separate named alertcondition declarations in this release. Keep original 1.2.0 GA alerts intact. TradingView alerts use a saved snapshot of script/settings; changing chart inputs does not automatically migrate an existing alert. For a new alert, select the desired condition and use Once Per Bar Close where appropriate; test delivery separately before retiring any existing alert.

## 11. Troubleshooting

  ---------------------------------------------------------------------------------------------------------------------
  Symptom                              Action
  ------------------------------------ --------------------------------------------------------------------------------
  Display setup greyed out             Use Managed Research Setup; leave Independent manual research off

  Dashboard follows unexpected setup   Pin Failed Break or Late OR Fail rather than Latest signal

  No research fills                    Check date filter, session, setup, loaded history and full observation horizon

  Live and Replay results differ       Match session, OR mode, dates, timeframe, cost, profile and loaded history

  START TRUNCATED                      Load more history if available or compare a common covered date range

  Planner absent                       Check 5m/15m, Show execution planner and qualifying S4 signal

  CE10117 compiled token limit         Revert to frozen GA; avoid duplicating engines

  CW10003 consistency warning          Review stateful function scope; do not assume a warning is harmless

  Chart clutter                        Hide session summary and research as described in section 3
  ---------------------------------------------------------------------------------------------------------------------

## 12. Release record and limitations

-   DEV 0.2.60: stable managed research baseline.
-   DEV 0.2.61 / 0.2.61a: abandoned parallel validation experiments; token ceiling and consistency warning.
-   DEV 0.2.62: historical coverage diagnostics and hardening.
-   RC1: minimal version promotion and Replay regression.
-   RC1a: managed research selector, compiled and tested on both timeframes.
-   1.3.0 GA + S4: frozen RC1a promotion; observed live chart installation.

GA supports live chart analysis. It is not a certification of profitability, complete non-repainting behavior, alert delivery or exhaustive resource safety. Exact compiled-token headroom is unknown. The source declares 500 labels, 500 lines and 400 boxes. Independent forward validation can be performed with the existing date filter in separate passes; no simultaneous forward-validation dashboard is included. Preserve GA and use a separate branch for future development.

## Appendix A. Complete exposed input reference

The following declarations are extracted verbatim (apart from leading indentation) from the frozen source, including defaults, options, limits and tooltips. Group identifiers refer to the original Pine source. Fixed internal settings are not necessarily exposed in the TradingView menu.

### 01 \| Active session

``` pine
preset = input.string("New York", "Active session", options=["Sydney","Tokyo","London","New York","Custom"], group=g1)
sydney = input.session("0900-1800", "Sydney hours (Sydney local)", group=g1)
tokyo = input.session("0900-1800", "Tokyo hours (Tokyo local)", group=g1)
london = input.session("0800-1700", "London hours (London local)", group=g1)
newYork = input.session("0930-1700", "New York hours (NY local)", group=g1)
```

### 02 \| Previous session levels

``` pine
referencePreset = input.string("London", "Reference session (High / Low / POC)", options=["Sydney","Tokyo","London","New York","Custom"], group=g2)
showPreviousSessionLevels = input.bool(true, "Show previous session levels (High / Low / POC)", group=g2)
```

### 03 \| Order blocks

``` pine
showBullOB = input.bool(true, "Bullish OB", group=g3)
showBearOB = input.bool(true, "Bearish OB", group=g3)
obMode = input.string("Fade", "Mitigated OB", options=["Hide","Fade","Keep"], group=g3)
obSearch = input.int(8, "Opposite candle lookback", minval=1, maxval=30, group=g3)
obATR = input.float(1.2, "Displacement / ATR", minval=0.1, step=0.1, group=g3)
obExtension = input.int(100, "Extension (bars)", minval=5, maxval=500, group=g3)
bullOBColor = input.color(color.teal, "Bull OB color", group=g3)
bearOBColor = input.color(color.maroon, "Bear OB color", group=g3)
```

### 04 \| Fair value gaps

``` pine
showBullFVG = input.bool(true, "Bullish FVG", group=g4)
showBearFVG = input.bool(true, "Bearish FVG", group=g4)
fvgMode = input.string("Fade", "Filled FVG", options=["Hide","Fade","Keep"], group=g4)
fvgMinATR = input.float(0.10, "Minimum gap / ATR", minval=0, step=0.05, group=g4)
fvgExtension = input.int(100, "Extension (bars)", minval=5, maxval=500, group=g4)
bullFVGColor = input.color(color.aqua, "Bull FVG color", group=g4)
bearFVGColor = input.color(color.fuchsia, "Bear FVG color", group=g4)
```

### 06 \| Zone styling

``` pine
activeTransparency = input.int(83, "Active fill transparency", minval=0, maxval=100, group=g6)
fadeTransparency = input.int(94, "Mitigated transparency", minval=70, maxval=100, group=g6)
```

### 08 \| Opening range (S4 engine)

``` pine
showORDashboard = input.bool(true, "Show session / grading dashboard (4H)", group=g9)
```

### 09 \| Label management

``` pine
prevLabelsOn = input.bool(true, "Previous session labels", group=g10)
prevLabelSide = input.string("Right", "Previous session label side", options=["Right","Left"], group=g10)
orHLLabelsOn = input.bool(true, "Opening Range H/L labels", group=g10)
orHLLabelSide = input.string("Left", "Opening Range H/L label side", options=["Right","Left"], group=g10)
fibLabelsOn = input.bool(true, "OR Fibonacci labels", group=g10)
fibLabelSide = input.string("Left", "OR Fibonacci label side", options=["Right","Left"], group=g10)
structureLabelsOn = input.bool(true, "MTF Structure labels", group=g10)
structureLabelSide = input.string("Right", "MTF Structure label side", options=["Right","Left"], group=g10)
labelFontInput = input.string("Small", "Global label font size", options=["Tiny","Small","Normal"], group=g10)
rightLabelExtraMinutes = input.int(60, "Extra right-side label offset (minutes)", minval=0, maxval=1440, step=15, group=g10, tooltip="Moves TF, 2H, previous-session and premarket labels farther right. Uses time coordinates rather than future bar indices.")
```

### 10 \| Area of Interest (AoI)

``` pine
showAOI = input.bool(true, "Show AoI", group=g11)
showBearAOI = input.bool(true, "Bearish resistance AoI", group=g11)
showBullAOI = input.bool(true, "Bullish support AoI", group=g11)
aoiMaxATR = input.float(1.5, "Maximum zone width / ATR", minval=0.05, maxval=20, step=0.05, group=g11)
aoiATRLength = input.int(14, "ATR length (4H reference)", minval=1, maxval=200, group=g11)
aoiDisplayMode = input.string("Full width", "AoI horizontal display", options=["Full width","From formation","Compact"], group=g11)
aoiHalfWindowMinutes = input.int(120, "AoI half-width (minutes, ±)", minval=15, maxval=1440, step=15, group=g11)
aoiShowLabels = input.bool(true, "AoI labels", group=g11)
aoiBullColor = input.color(color.teal, "Bullish AoI color", group=g11)
aoiBearColor = input.color(color.red, "Bearish AoI color", group=g11)
aoiOpacity = input.int(85, "AoI transparency", minval=0, maxval=100, group=g11)
aoiConfluenceOn = input.bool(true, "Highlight OB / FVG confluence", group=g11)
aoiConfluenceOB = input.bool(true, "Include active order blocks", group=g11)
aoiConfluenceFVG = input.bool(true, "Include active fair value gaps", group=g11)
aoiConfluenceColor = input.color(color.yellow, "Confluence border color", group=g11)
aoiConfluenceWidth = input.int(3, "Confluence border width", minval=1, maxval=4, group=g11)
```

### 11 \| 2H Support & Resistance

``` pine
show2HSR = input.bool(true, "Show 2H support / resistance", group=g13)
sr2LevelsPerSide = input.int(3, "Levels per side (above / below price)", minval=0, maxval=3, group=g13)
sr2MinDistance = input.float(50.0, "Minimum separation (price points)", minval=0.0, step=5.0, group=g13)
sr2Labels = input.bool(true, "Show labels", group=g13)
```

### 13 \| Alerts

``` pine
enableZoneAlerts = input.bool(true, "Enable OB / FVG alerts", group=gAlerts)
```

### Developer Diagnostics

``` pine
showStructureOrigins = input.bool(false, "Mark structure origin candles", group="Developer Diagnostics")
developerDiagnostics = input.bool(false, "Show developer diagnostic tables", group="Developer Diagnostics")
showTFBreakdown = input.bool(false, "Show TF Rating breakdown (diagnostic)", group="Developer Diagnostics")
```

### S4 \| Opening Range

``` pine
string s4_orMode = input.string("30 Minutes", "Opening Range Mode", options=["15 Minutes", "30 Minutes"], group=s4_gORMode, tooltip="Selects one Opening Range for S4 calculations, breakout/retest, rejection signals, plots, and box.")
bool s4_fibOn = input.bool(true, "Auto OR Fib 0.500 / 0.618", group=s4_gORMode)
bool s4_fibDirectionLabel = input.bool(true, "Show OR Fib direction audit label (DEV)", group=s4_gORMode)
```

### S4 \| Key levels

``` pine
bool s4_usePrevRTH = input.bool(true, "Previous ACTIVE-session high / low", group=s4_gLevels)
bool s4_usePrevFull = input.bool(false, "Previous REFERENCE-session high / low", group=s4_gLevels)
bool s4_usePOC = input.bool(false, "Use GA previous REFERENCE-session approximate POC", group=s4_gLevels)
```

### S4 \| OR breakout / retest (WATCH only)

``` pine
bool s4_enableORBreakRetest = input.bool(true, "Detect OR breakout and retest WATCH", group=s4_gORRetest)
bool s4_showORBreakRetest = input.bool(true, "Show OR breakout / retest WATCH", group=s4_gORRetest)
bool s4_auditORBreakRetest = input.bool(false, "Show OR RETEST VALID diagnostic", group=s4_gORRetest)
bool s4_auditORVerbose = input.bool(false, "Show detailed OR transitions (BREAK / DEPART / INVALIDATED / REARMED)", group=s4_gORRetest)
bool s4_auditORExact = input.bool(false, "AUDIT: compact OR retest history table", group=s4_gORRetest, tooltip="Shows the most recent four confirmed OR retests in a compact bottom-right table. ET timestamps and completed-bar close prices. No effect on detection.")
float s4_qualityBodyMin = input.float(0.50, "AUDIT: strong retest minimum body / range", minval=0.05, maxval=1.0, step=0.05, group=s4_gORRetest, tooltip="Research classification only. Does not filter WATCH or BUY/SELL.")
int s4_orDepartureTicks = input.int(40, "Minimum OR departure (ticks)", minval=1, maxval=1000, group=s4_gORRetest, tooltip="A later completed candle must close this many ticks beyond the OR boundary before a subsequent retest can qualify.")
```

### S4 \| Breakout retest BUY/SELL (DEV)

``` pine
bool s4_brConfirmOn = input.bool(true, "Enable breakout-retest BUY / SELL (research)", group=s4_gBRConfirm)
bool s4_brConfirmLabels = input.bool(true, "Show confirmed breakout-retest BUY / SELL", group=s4_gBRConfirm)
bool s4_brConfirmAudit = input.bool(false, "Show confirmation PASS / FAIL diagnostic on retests", group=s4_gBRConfirm)
float s4_brWickMin = input.float(0.40, "Minimum directional rejection wick / range", minval=0.05, maxval=1.0, step=0.05, group=s4_gBRConfirm)
bool s4_brVolumeOn = input.bool(true, "Allow volume > previous candle as confirmation", group=s4_gBRConfirm)
bool s4_brWickOn = input.bool(true, "Allow large directional wick as confirmation", group=s4_gBRConfirm)
bool s4_brDelayedOn = input.bool(true, "Enable delayed wick-recovery confirmation (research)", group=s4_gBRConfirm, tooltip="After a valid retest with a qualifying rejection wick but no immediate directional BUY/SELL, wait for a directional close beyond that retest candle's high/low. Closed bars only.")
int s4_brDelayedBars = input.int(2, "Delayed confirmation window (bars)", minval=1, maxval=5, group=s4_gBRConfirm)
```

### S4 \| OR retest / available room (research)

``` pine
bool s4_showRoomOnRetest = input.bool(true, "Show multi-timeframe room on OR retest WATCH", group=s4_gRoom)
float s4_roomMinPoints = input.float(20.0, "Research room reference (NQ points)", minval=0.0, step=5.0, group=s4_gRoom, tooltip="Informational only; never gates WATCH or generates BUY/SELL.")
bool s4_roomUseChart = input.bool(true, "Include confirmed chart-TF pivots", group=s4_gRoom)
bool s4_roomUse2H = input.bool(true, "Include retained confirmed 2H structure", group=s4_gRoom)
string s4_room2HScope = input.string("Nearby FS-style pivots", "2H obstacle scope", options=["Nearby FS-style pivots", "Full confirmed pivot pool"], group=s4_gRoom, tooltip="Nearby mode applies the FS Builds directional rank and spacing selection to confirmed pivots. The purple display also supports 2H retest-memory levels, which are excluded here until timestamped.")
bool s4_roomUsePreviousOR = input.bool(true, "Include previous session's selected OR", group=s4_gRoom)
bool s4_roomAuditPrices = input.bool(false, "AUDIT: show obstacle prices and source", group=s4_gRoom, tooltip="Shows exact first/secondObstacle obstacle prices and candle-close anchor. Confirmed chart pivots become available only after right-side bars close; 2H levels use the existing GA retained confirmed bank.")
bool s4_roomCompactLabels = input.bool(true, "Compact ROOM labels (audit OFF)", group=s4_gRoom, tooltip="Compact label shows nearest eligible obstacle and distance. Turn on AUDIT to see full dates, prices, and the next obstacle.")
bool s4_roomQualityAudit = input.bool(false, "AUDIT: level crossing history (top-right table)", group=s4_gRoom, tooltip="Research-only table for the last three ROOM WATCH events. Crossing counts are price-keyed and are not lifetime freshness.")
int s4_qualityInspect = input.int(0, "AUDIT: inspect WATCH event (0 = latest)", minval=0, maxval=199, group=s4_gRoom, tooltip="0 is latest unique WATCH, 1 is previous, etc. Shows the selected event plus the preceding two. Historical records are retained up to 200 events.")
int s4_roomPivotBars = input.int(3, "Chart-TF pivot left/right bars", minval=1, maxval=15, group=s4_gRoom)
int s4_roomPivotHistory = input.int(40, "Max chart-TF pivots retained", minval=5, maxval=100, group=s4_gRoom)
```

### S4 \| Rejection logic

``` pine
int s4_toleranceTicks = input.int(12, "Level proximity tolerance (ticks)", minval=0, group=s4_gLogic)
float s4_minWatchBody = input.float(0.25, "Minimum WATCH s4_body / range", minval=0, maxval=1, step=0.05, group=s4_gLogic)
float s4_minStrongBody = input.float(0.50, "Minimum STRONG s4_body / range (each bar)", minval=0, maxval=1, step=0.05, group=s4_gLogic)
int s4_departureTicks = input.int(40, "Departure s4_distance before a new retest (ticks)", minval=1, group=s4_gLogic)
bool s4_auditOR15 = input.bool(false, "Audit OR15 departures / retests (DEV)", group=s4_gLogic)
```

### S4 \| Pattern awareness

``` pine
bool s4_useEveningStar = input.bool(true, "Detect potential / confirmed Evening Star", group=s4_gPatterns)
bool s4_useMorningStar = input.bool(true, "Detect potential / confirmed Morning Star", group=s4_gPatterns)
float s4_dojiMaxBody = input.float(0.20, "Middle candle maximum s4_body / range", minval=0.01, maxval=0.5, step=0.01, group=s4_gPatterns)
float s4_starMinFirstBody = input.float(0.50, "First candle minimum s4_body / range", minval=0.1, maxval=1, step=0.05, group=s4_gPatterns)
float s4_starMinThirdBody = input.float(0.50, "Confirmation candle minimum s4_body / range", minval=0.1, maxval=1, step=0.05, group=s4_gPatterns)
```

### S4 \| Display / labels

``` pine
bool s4_showWatch = input.bool(true, "Show WATCH", group=s4_gDisplay)
bool s4_showStrong = input.bool(true, "Show BUY / SELL research triggers", group=s4_gDisplay)
color s4_buyLabelColor = input.color(color.rgb(0, 220, 70), "BUY label — bright green", group=s4_gDisplay)
color s4_sellLabelColor = input.color(color.rgb(255, 35, 35), "SELL label — bright red", group=s4_gDisplay)
bool s4_show2Bar = input.bool(true, "Show STRONG 2BAR", group=s4_gDisplay)
bool s4_showWeak2Bar = input.bool(false, "Show WEAK 2BAR (research)", group=s4_gDisplay)
float s4_twoBarBodyMin = input.float(0.50, "STRONG 2BAR minimum s4_body / range (both bars)", minval=0.05, maxval=1.0, step=0.05, group=s4_gDisplay)
bool s4_showConfluence = input.bool(true, "Show level confluence tags", group=s4_gDisplay)
bool s4_showPatternLabels = input.bool(true, "Show star-pattern WATCH / confirmation", group=s4_gDisplay)
bool s4_showLevelPriceTags = input.bool(true, "Show right-edge previous-day level prices", group=s4_gDisplay)
float s4_labelGapATR = input.float(0.13, "Vertical label spacing (ATR units)", minval=0.05, maxval=2.0, step=0.01, group=s4_gDisplay)
int s4_labelATRLength = input.int(14, "Label spacing ATR length", minval=1, group=s4_gDisplay)
```

### S4 \| Display / key-level lines

``` pine
bool s4_plotSelectedOR = input.bool(true, "Show selected OR boundaries", group=s4_gLines)
bool s4_showORBox = input.bool(true, "Show selected OR shaded box", group=s4_gLines)
int s4_orFillTransp = input.int(90, "Selected OR box transparency", minval=75, maxval=100, group=s4_gLines)
bool s4_plotRTH = input.bool(true, "Show previous ACTIVE-session high / low", group=s4_gLines)
bool s4_plotFull = input.bool(false, "Show previous REFERENCE-session high / low", group=s4_gLines)
bool s4_plotPOC = input.bool(false, "Show previous reference-session approximate POC", group=s4_gLines)
color s4_orGreen = input.color(color.rgb(0, 184, 112), "Selected OR green", group=s4_gLines)
color s4_rthHighColor = input.color(color.rgb(210, 85, 90), "Previous active-session high color", group=s4_gLines)
color s4_rthLowColor = input.color(color.rgb(226, 148, 35), "Previous active-session low color", group=s4_gLines)
color s4_fullColor = input.color(color.rgb(155, 113, 188), "Previous reference-session color", group=s4_gLines)
color s4_pocColor = input.color(color.rgb(199, 164, 49), "POC color", group=s4_gLines)
```

### S4 \| Pattern expansion (DEV 0.2.35)

``` pine
bool s4x_on = input.bool(true, "Enable expanded informational pattern markers", group=s4x_group)
bool s4x_tweezer = input.bool(true, "Perfect Tweezers (2 adjacent candles)", group=s4x_group)
int s4x_tweezerTicks = input.int(1, "Perfect Tweezer tolerance (ticks)", minval=0, maxval=2, group=s4x_group)
int s4x_trendBars = input.int(4, "Tweezer preceding trend lookback", minval=2, maxval=20, group=s4x_group)
float s4x_rejectMin = input.float(0.20, "Tweezer rejection wick / range minimum", minval=0.0, maxval=0.8, step=0.05, group=s4x_group)
bool s4x_engulf = input.bool(true, "Bullish / Bearish Engulfing", group=s4x_group)
bool s4x_harami = input.bool(true, "Bullish / Bearish Harami", group=s4x_group)
bool s4x_threeInside = input.bool(true, "Three Inside Up / Down", group=s4x_group)
bool s4x_struct = input.bool(true, "H&S / Inverse H&S / M / W structures", group=s4x_group)
int s4x_pivotLen = input.int(3, "Structure pivot left/right bars", minval=2, maxval=12, group=s4x_group)
int s4x_matchTicks = input.int(40, "M/W and shoulder similarity (ticks)", minval=1, maxval=400, group=s4x_group)
int s4x_minSwingTicks = input.int(40, "Minimum structure swing depth (ticks)", minval=1, maxval=1000, group=s4x_group)
int s4x_confirmBars = input.int(30, "Maximum bars to break structure neckline", minval=3, maxval=100, group=s4x_group)
bool s4x_failed = input.bool(true, "Investigate failed OR-high breakout (bearish)", group=s4x_group)
int s4x_failWindow = input.int(8, "Failed breakout monitoring window (bars)", minval=2, maxval=30, group=s4x_group)
float s4x_failWick = input.float(0.35, "Failed breakout upper-wick / range", minval=0.05, maxval=0.90, step=0.05, group=s4x_group)
float s4x_failBody = input.float(0.50, "Failed breakout bearish body / range", minval=0.10, maxval=1.0, step=0.05, group=s4x_group)
bool s4x_fbSellOn = input.bool(true, "Enable SELL · FAILED OR HIGH (research)", group=s4x_group, tooltip="Default ON for DEV testing. Requires prior breakout and exhaustion, a strong bearish close below selected OR high, and volume above previous bar.")
bool s4x_fbSellLabels = input.bool(true, "Show failed-breakout SELL marker", group=s4x_group)
bool s4x_fbSellAudit = input.bool(false, "AUDIT: failed-breakout SELL details", group=s4x_group)
```

### S4 \| Failed-breakout outcome audit (DEV 0.2.38)

``` pine
bool s4x_outEnable = input.bool(true, "Track 3-bar and 6-bar SELL MFE/MAE", group=s4x_outGroup)
bool s4x_outLabels = input.bool(false, "Show compact audit after bar 6 closes", group=s4x_outGroup)
bool s4x_outTable = input.bool(true, "Show unified SELL session summary", group=s4x_outGroup)
bool s4x_outReclaimLabels = input.bool(false, "Show OR HIGH RECLAIMED diagnostic", group=s4x_outGroup)
float s4x_outThreshold = input.float(20.0, "Favorable excursion threshold (points)", minval=0.25, step=0.25, group=s4x_outGroup)
int s4x_outMaxRecords = input.int(30, "Maximum concurrent audit records", minval=1, maxval=100, group=s4x_outGroup)
string s4x_outTablePosition = input.string("Top Right", "Summary table position", options=["Top Right", "Top Left", "Bottom Right", "Bottom Left"], group=s4x_outGroup)
```

### S4 \| Late OR-high failure (DEV 0.2.40)

``` pine
bool s4l_enable = input.bool(true, "Enable late OR-high failure research", group=s4l_group)
bool s4l_showWatch = input.bool(true, "Show WATCH LATE OR FAIL", group=s4l_group)
bool s4l_showSell = input.bool(true, "Show SELL LATE OR FAIL", group=s4l_group)
bool s4l_audit = input.bool(false, "AUDIT: late failure confirmation details", group=s4l_group)
int s4l_acceptBars = input.int(6, "Minimum consecutive closes above OR high", minval=2, maxval=40, group=s4l_group)
float s4l_approachPoints = input.float(25.0, "Approach distance above OR high (points)", minval=0.25, step=0.25, group=s4l_group)
int s4l_confirmWindow = input.int(6, "Maximum bars from WATCH to breakdown", minval=1, maxval=20, group=s4l_group)
float s4l_minBody = input.float(0.50, "SELL bearish body / range minimum", minval=0.10, maxval=1.0, step=0.05, group=s4l_group)
bool s4l_allowVolume = input.bool(true, "Allow volume > previous bar", group=s4l_group)
bool s4l_allowUpperWick = input.bool(true, "Allow upper wick confirmation", group=s4l_group)
float s4l_wickMin = input.float(0.35, "Upper wick / range minimum", minval=0.05, maxval=0.90, step=0.05, group=s4l_group)
```

### S4 \| Trade risk profiles

``` pine
string s51_profile = input.string("AUTO — Research Adaptive", "Trade risk profile", options=["AUTO — Research Adaptive","A+ — Conservative","B+ — Balanced","C — Aggressive"], group=s51_group, tooltip="AUTO selects a frozen preset by 5m/15m timeframe and confirmed SELL setup. A+, B+ and C lock the entry profile; target and window remain preset-managed.")
string s58_managedSetup = input.string("Latest signal", "Managed Research Setup", options=["Latest signal","Failed Break","Late OR Fail"], group=s51_group, tooltip="Select the managed research dashboard cohort. Latest signal preserves automatic display switching. This does not filter execution planner signals or alerts.")
bool s58_manual = input.bool(false, "Independent manual research", group=s51_group, tooltip="OFF: research follows the execution preset. ON: advanced research inputs control the research table only; execution remains profile-managed.")
```

### S4 \| Advanced research

``` pine
bool s48_on = input.bool(true, "Enable research", group=s48_group)
string s48_setup = input.string("Failed Break", "Display setup", options=["Failed Break","Late OR Fail"], group=s48_group, active=s58_manual)
int s48_minutes = input.int(30, "Evaluation minutes", options=[30,90], group=s48_group, active=s58_manual)
string s48_mode = input.string("Fixed points", "Target model", options=["Fixed points","Risk multiple"], group=s48_group, active=s58_manual)
string s49_entryModel = input.string("Fixed points", "Entry model", options=["Fixed points","Adaptive risk %"], group=s48_group, active=s58_manual)
float s49_maxRisk = input.float(0.0, "Maximum original risk (0 = off)", minval=0, step=0.25, group=s48_group, active=s58_manual)
bool s49_common = input.bool(false, "Match 30m / 90m cohorts (require full 90m)", group=s48_group, active=s58_manual)
float s48_buffer = input.float(2.0, "Stop wick buffer (points)", minval=0.25, step=0.25, group=s48_group, active=s58_manual)
float s48_cost = input.float(0.0, "Round-trip cost (points)", minval=0, step=0.25, group=s48_group, active=s58_manual)
bool s50_perf = input.bool(true, "Show performance metrics", group=s48_group)
bool s51_focus = input.bool(false, "Show selected profile only", group=s48_group, active=s58_manual)
int s48_start = input.time(timestamp("01 Aug 2026 00:00 -0400"), "Signal date from", group=s48_group)
int s48_end = input.time(timestamp("13 Nov 2026 00:00 -0500"), "Signal date until (exclusive)", group=s48_group)
```

### S4 \| On-chart execution planner

``` pine
float s57_cost = input.float(2.0, "Execution round-trip cost (points)", minval=0, step=0.25, group="S4 | On-chart execution planner")
bool s52_show = input.bool(true, "Show execution planner", group="S4 | On-chart execution planner")
bool s52_zones = input.bool(true, "Show risk / reward zones", group="S4 | On-chart execution planner")
int s52_extension = input.int(5, "Label extension (bars)", minval=1, maxval=20, group="S4 | On-chart execution planner")
int s53_qty = input.int(1, "Contracts", minval=1, maxval=1000, group="S4 | On-chart execution planner")
string s53_display = input.string("Points + net cash", "Projected P&L display", options=["Points only","Points + gross cash","Points + net cash"], group="S4 | On-chart execution planner")
```
