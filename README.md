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
