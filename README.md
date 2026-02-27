# Pine Scripts for TradingView

A growing collection of technical trading indicators and strategy tools for the TradingView charting platform, written in Pine Script v6.

Pine Script Documentation:
- [Pine Script User Manual](https://www.tradingview.com/pine-script-docs/)
- [Pine Script Language Reference Manual](https://www.tradingview.com/pine-script-reference/v6/)
- [PineCoders](https://www.pinecoders.com/)

## Overview

This repository is an evolving toolkit for systematic trading on TradingView. The current focus spans three tiers: SPY 0DTE options scalping (intraday signal generation), multi-chart directional bias monitoring (watchlist reconnaissance), and strategic trend assessment (multi-day/multi-week trend evaluation for any equity, ETF, or index). Plans include broader market internals, multi-asset strategies, and backtesting frameworks.

Currently, the suite includes a family of SPY 0DTE Scalper indicators spanning three timeframes (1-minute, 5-minute, 15-minute), each independently calibrated for its target holding period; a general-purpose Market Monitor and a 5-minute optimized Market Monitor for watchlist reconnaissance; and a Trend Compass family (Daily, 4-Hour) for strategic trend assessment on any liquid instrument. All indicators share a common architectural foundation: EMA ribbon overlays, regime/phase classification, weighted scoring engines, and real-time dashboards with color-coded status fields. They diverge in signal generation approach, parameter calibration, and feature depth based on the characteristics of their target timeframe and use case.

## Indicators

### SPY 0DTE Scalper

A high-precision signal generation system for scalping 0DTE SPY options. Combines trend structure, momentum, volatility, volatility compression (TTM Squeeze), market breadth (NYSE TICK), and key price levels into a unified decision-support overlay with real-time signal generation.

Each timeframe variant is independently calibrated. The signal engine approach differs by timeframe: the 1-minute variant uses a strict AND-gate (all conditions must pass), while the 5-minute and 15-minute variants use a confluence scoring model (conditions contribute points toward a configurable threshold) that allows partial confluence.

| Script | Timeframe | Version | Signal Engine | Hold Time | Docs |
| ------ | --------- | ------- | ------------- | --------- | ---- |
| [spy_0dte_scalper_1min_v1.0.pine](spy_0dte_scalper/spy_0dte_scalper_1min_v1.0.pine) | 1-min | v1.0 | AND-gate | 1-5 min | [docs](docs/spy_0dte_scalper_1min_v1.0.md) |
| [spy_0dte_scalper_1min_v1.1.pine](spy_0dte_scalper/spy_0dte_scalper_1min_v1.1.pine) | 1-min | v1.1 | AND-gate + optional filters | 1-5 min | [docs](docs/spy_0dte_scalper_1min_v1.1.md) |
| [spy_0dte_scalper_5min_v1.0.pine](spy_0dte_scalper/spy_0dte_scalper_5min_v1.0.pine) | 5-min | v1.0 | 5+2 scoring | 5-25 min | [docs](docs/spy_0dte_scalper_5min_v1.0.md) |
| [spy_0dte_scalper_5min_v1.1.pine](spy_0dte_scalper/spy_0dte_scalper_5min_v1.1.pine) | 5-min | v1.1 | 5+4 scoring | 5-25 min | [docs](docs/spy_0dte_scalper_5min_v1.1.md) |
| [spy_0dte_scalper_15min_v1_0.pine](spy_0dte_scalper/spy_0dte_scalper_15min_v1_0.pine) | 15-min | v1.0 | Scoring + bias | 15-60 min | [docs](docs/spy_0dte_scalper_15min_v1_0.md) |
| [spy_0dte_scalper_15min_v1_1.pine](spy_0dte_scalper/spy_0dte_scalper_15min_v1_1.pine) | 15-min | v1.1 | Scoring + bias | 15-60 min | [docs](docs/spy_0dte_scalper_15min_v1_1.md) |

### Market Monitor

Directional bias overlays designed for multi-chart watchlist monitoring. These are not signal generators. They answer one question: "which way is the wind blowing?" Use them for reconnaissance across a watchlist, then drill into the 0DTE Scalper on whichever charts show strong directional bias.

The **general-purpose** variant is timeframe-adaptive (1m through Daily) with a simple equal-weighted bias score. The **5-minute optimized** variant adds weighted scoring, session awareness, market breadth (NYSE TICK), volatility compression detection (TTM Squeeze), ATR regime classification, and expanded reference levels for intraday monitoring workflows.

| Script | Variant | Version | Bias Score Range | Dashboard Rows | Docs |
| ------ | ------- | ------- | ---------------- | -------------- | ---- |
| [market_monitor_v1_0.pine](market_monitor/market_monitor_1min_v1_0.pine) | General Purpose | v1.0 | -5 to +5 (equal) | 12 | [docs](docs/market_monitor_1min_v1_0.md) |
| [market_monitor_5min_v1_0.pine](market_monitor/market_monitor_5min_v1_0.pine) | 5-Minute Optimized | v1.0 | -11 to +11 (weighted) | 18 | [docs](docs/market_monitor_5min_v1_0.md) |

#### Market Monitor Comparison

| Feature | General Purpose (v1.0) | 5-Minute Optimized (v1.0) |
| ------- | --------------------- | ------------------------- |
| Target timeframe | Adaptive (1m - Daily) | 5-minute (primary) |
| Scoring mode | Equal weight (all 1x) | Structural 2x / Confirmation 1x |
| Bias score range | -5 to +5 | -11 to +11 (weighted), -8 to +8 (equal) |
| Session phases | None | Open Drive / Morning / Midday / Power Hour / Close |
| NYSE TICK | Not included | 6-tier classification, bias contribution |
| TTM Squeeze | Not included | BB/KC compression with momentum direction |
| ATR regime | Raw value only | Percentile-ranked (LOW / NORMAL / HIGH / EXTREME) |
| Opening Range | Not included | Configurable duration (default 15 min) |
| Session HOD/LOD | Not included | Real-time tracking with dashboard context |
| Prior Day VWAP | Not included | Latched from previous session close |
| Anchor EMA | Not included | 15m 50 EMA plotted on chart |
| Bias trend | Not tracked | IMPROVING / STABLE / FADING |
| HTF mapping | Auto-adaptive | Fixed to 15m |
| `request.security` calls | 3 | 4 |
| Dashboard rows | 12 | 18 |
| Alert conditions | 6 | 9 |

### Trend Compass

Strategic trend assessment overlays for evaluating multi-day and multi-week trends on any equity, ETF, or index. These are not signal generators and not SPY-specific. They answer: "What is the trend? How strong is it? Is it accelerating or exhausting? Are divergences forming?" Use them to establish macro bias before deploying intraday tools.

The **Daily** variant is the primary strategic view, pulling Weekly context internally. The **4-Hour** variant provides faster feedback on trend shifts within the current multi-day swing, pulling Daily and Weekly context via `request.security()`.

| Script | Timeframe | Version | Score Range | HTF Context | Dashboard Rows | Docs |
| ------ | --------- | ------- | ----------- | ----------- | -------------- | ---- |
| [trend_compass_daily_v1_0.pine](trend_compass/trend_compass_daily_v1_0.pine) | Daily | v1.0 | -11 to +11 (weighted) | Weekly 50 EMA | 17 | [docs](docs/trend_compass_daily_v1_0.md) |
| [trend_compass_4h_v1_0.pine](trend_compass/trend_compass_4h_v1_0.pine) | 4-Hour | v1.0 | -11 to +11 (weighted) | D50 + D200 + W50 EMA | 18 | [docs](docs/trend_compass_4h_v1_0.md) |

#### Trend Compass Comparison

| Feature | Daily (v1.0) | 4-Hour (v1.0) |
| ------- | ------------ | -------------- |
| EMA Ribbon | 10/21/50 (local) | 10/21/50 (local, 4H bars) |
| Anchor EMA | 200 (local daily) | Daily 200 EMA (via `request.security`) |
| HTF layers | Weekly 50 EMA | Daily 50 + Daily 200 + Weekly 50 |
| Key levels | Prior Week + Prior Month H/L/C | Prior Day + Prior Week H/L/C |
| 52-Week H/L | Local `ta.highest(252)` | Via Daily `request.security` |
| Divergence pivots | 5L/3R, 15-bar decay | 4L/2R, 20-bar decay |
| ATR/BB percentile lookback | 100 bars (~5 months) | 200 bars (~33 days) |
| EMA compression threshold | 0.5% | 0.3% |
| Fibonacci (optional) | 50-bar lookback | 100-bar lookback |
| Ichimoku Cloud (optional) | Standard 9/26/52/26 | Standard 9/26/52/26 |
| `request.security` calls | 3 | 6 |
| Dashboard rows | 17 | 18 |
| Alert conditions | 12 | 12 |

#### Trend Phase Lifecycle

The Trend Compass introduces a six-state trend phase classifier that maps where a trend sits in its lifecycle:

| Phase | ADX Context | EMA Context | Interpretation |
| ----- | ----------- | ----------- | -------------- |
| EMERGING | ADX < 25, rising | EMAs separating | New trend forming. Early entry opportunity. |
| ACCELERATING | ADX > 20, rising | Slope confirming | Trend gaining momentum. Strongest phase. |
| MATURE | ADX > 30, flat | Aligned but plateauing | Trend intact, momentum fading. Trail stops. |
| EXHAUSTING | ADX declining from > 30 | Slope flattening | Trend losing steam. Prepare for transition. |
| CONSOLIDATING | ADX < 20 | Compressed | Range-bound. No directional bias. |
| REVERSING | Recent crossovers | Crossover detected | Trend change underway. Re-evaluate bias. |

## Timeframe Comparison

Key parameter and architectural differences across the SPY 0DTE Scalper variants (v1.1 for each):

| Parameter | 1-Minute | 5-Minute | 15-Minute |
| --------- | -------- | -------- | --------- |
| EMA Lengths | 8 / 13 / 21 | 9 / 15 / 21 | 13 / 21 / 55 |
| RSI Length | 14 | 10 | 10 |
| RSI OB / OS | 70 / 30 | 65 / 35 | 65 / 35 |
| ADX No-Trend Threshold | 15 | 18 | 20 |
| Opening Range | 5 min (5 bars) | 15 min (3 bars) | 30 min (2 bars) |
| Signal Cooldown | 5 bars (5 min) | 3 bars (15 min) | 2 bars (30 min) |
| Signal Window Start | 09:35 | 09:45 | 10:00 |
| VWAP Extended Distance | 1.5 ATR | 2.0 ATR | 2.5 ATR |
| Signal Engine | AND-gate | 5+4 scoring | Scoring + bias |
| Squeeze Integration | AND-gate filter (opt-in) | Scoring condition | Scoring condition |
| TICK Integration | AND-gate filter (opt-in) | Scoring condition | Scoring condition |
| MTF Confirmation | N/A | 1-min RSI + EMA | 5-min RSI + EMA |
| RTH Bars / Session | ~390 | ~78 | ~26 |
| Dashboard Rows | 16 | 18 | 18+ |

## Core Components

Every indicator in the suite shares these foundational components, calibrated per-timeframe:

**Trend Structure** -- EMA Ribbon (3 EMAs) with dynamic cloud fill that shifts color based on alignment state (bullish, bearish, mixed). Forms the visual trend backbone on all chart overlays.

**Institutional Anchor** -- Session-anchored VWAP with configurable standard deviation bands. Auto-disables on non-intraday timeframes. VWAP cross markers (diamond shapes) flag reclaims and rejections.

**Key Price Levels** -- Pre-market high/low, prior day high/low/close, opening range (configurable duration), and real-time session HOD/LOD with dynamically updating lines.

**Regime Classification** -- Five-mode classifier (BULLISH, BEARISH, RANGING, NO TRADE, TRANSITION) based on EMA alignment, ADX strength, and RSI positioning. Provides context for signal interpretation.

**Dashboard** -- Real-time table overlay with color-coded status fields covering all active indicators. Configurable size (Tiny/Small/Normal) and corner positioning for multi-chart layouts.

**Alert System** -- Dual implementation: static `alertcondition()` for TradingView's standard alert UI, plus dynamic `alert()` with interpolated context for webhook and notification pipelines.

### v1.1 Additions (All Scalper Timeframes)

**TTM Squeeze** -- Bollinger Band (20, 2.0) compression inside Keltner Channels (20, 1.5). Three states: SQUEEZE ON (compression building), FIRED (breakout beginning), OFF (normal volatility). Momentum histogram via linear regression.

**NYSE TICK Index** -- Real-time market breadth via `request.security("USI:TICK", "1", close)`. Six-tier classification from EXTREME BEAR to EXTREME BULL with configurable thresholds.

## Installation

1. Open TradingView and navigate to the appropriate chart and timeframe for the indicator.
   - **SPY 0DTE Scalpers**: SPY chart at the matching timeframe (1m, 5m, or 15m).
   - **Market Monitors**: Any ticker at any intraday timeframe (general) or 5-minute (optimized variant).
   - **Trend Compass Daily**: Any ticker on a Daily chart.
   - **Trend Compass 4H**: Any ticker on a 4-Hour chart.
2. Open the **Pine Editor** (bottom panel).
3. Delete any existing code in the editor.
4. Paste the entire contents of the desired `.pine` file.
5. Click **Add to chart**.

### Required Chart Settings

**Extended hours**:

Enable "Extended Trading Hours" in chart settings for pre-market high/low levels. Without extended hours data, PM fields show "N/A" and no PM level lines render.

**Timezone**:

All session-based logic (pre-market detection, RTH boundaries, opening range,
signal time windows) is controlled by the **Timezone** input (`i_tz`) within each
indicator's **Session** settings group. This defaults to `America/New_York`
(Eastern Time), which matches the conventional expression of US equity market
hours (e.g., RTH = 09:30–16:00 ET).

Your TradingView **chart timezone** setting is entirely independent. The chart
timezone — configured under *Symbol > Data Modification > Timezone* — only affects
how timestamps are displayed on the x-axis of your chart. It has no impact on how the indicator
detects sessions, calculates levels, or generates signals.

### Data Requirements

**NYSE TICK** (v1.1 scalpers and Market Monitor 5m): The TICK index (`USI:TICK`) requires your TradingView data plan to include the relevant exchange. If unavailable, TICK rows show "N/A" and TICK-based signal conditions pass through without blocking (1-min) or simply don't contribute score (5-min, 15-min, Market Monitor 5m).

## Recommended Layouts

### Strategic Trend Assessment

Run the Trend Compass Daily on a standalone chart for any ticker you're evaluating. Check trend phase, composite score, and divergence status to establish the macro bias. For faster feedback during active swing trading, add the Trend Compass 4H alongside the Daily. Neither requires SPY — use on NVDA, AAPL, GOOGL, QQQ, IWM, or any liquid instrument.

### Single-Timeframe Focus

Run one SPY 0DTE Scalper on a full-screen chart. Best for dedicated scalping sessions where you want maximum visual clarity.

### Multi-Timeframe Stack

Run 2-3 SPY 0DTE Scalper variants side-by-side (e.g., 1-min + 5-min, or 5-min + 15-min). The higher timeframe provides context for entries on the lower timeframe. Note that the 5-min and 15-min variants already pull lower-timeframe data internally via MTF confirmation, so running the lower-timeframe chart alongside is optional but provides additional visual context.

### Watchlist Reconnaissance

Run 6-8 Market Monitor instances on small chart tiles across your watchlist. Use Tiny or Small dashboard size with Top Right or Bottom Right positioning. Identify which tickers show strong directional bias, then open a dedicated chart with the appropriate 0DTE Scalper for signal generation.

For 5-minute focused monitoring, use the Market Monitor 5m variant — it provides richer context (session phase, squeeze state, TICK breadth, ATR regime, opening range position) at the cost of being optimized for a single timeframe. For timeframe-flexible watchlists spanning 1m through Daily, use the general-purpose Market Monitor.

### Full Pipeline

Trend Compass Daily/4H (macro bias) → Market Monitor watchlist scan (intraday alignment) → 0DTE Scalper on strongest setups (signal execution). Each layer narrows the decision space for the next.

## Performance and Limits

All indicators are optimized for simultaneous multi-instance execution on TradingView.

| Resource | Pine Script v6 Limit | Scalpers (per instance) | Monitor GP (per instance) | Monitor 5m (per instance) | TC Daily (per instance) | TC 4H (per instance) |
| -------- | -------------------- | ----------------------- | ------------------------- | ------------------------- | ----------------------- | -------------------- |
| `request.security()` calls | 40 per script | 4-5 | 3 | 4 | 3 | 6 |
| Labels | 500 per script | ~50-100 per session | 100 max | 200 max | 500 max | 500 max |
| Lines | 500 per script | ~20-40 | 200 max | 300 max | 500 max | 500 max |
| Boxes | 200 per script | ~5-10 | 50 max | 100 max | 100 max | 100 max |

**Computation**: No loops except percentile ranking calculations (ATR and BB width, in Market Monitor 5m and both Trend Compass variants — configurable lookback, default 100-200 bars, lightweight). All other calculations are vectorized with `var` declarations for persistent state. Dashboard tables update only on `barstate.islast` to minimize historical bar overhead.

**Repainting**: All signal logic is gated by `barstate.isconfirmed` by default. Signals appear after bar close, not during formation. `request.security()` calls use `lookahead=barmerge.lookahead_off` for all forward-looking data to prevent future data leakage. Prior day levels use `lookahead_on` which is appropriate for historical reference levels.

## Known Limitations

**No cumulative delta / order flow.** Pine Script lacks access to tick-level bid/ask data. Relative volume and NYSE TICK are the closest available proxies.

**No volume profile.** Pine Script cannot natively compute POC, VAH, VAL. Use TradingView's built-in Volume Profile tool or a separate indicator alongside.

**Single-symbol calibration.** The 0DTE Scalper parameters are calibrated for SPY's price range, volatility, and microstructure. Other instruments require significant parameter adjustment. The Trend Compass and Market Monitor indicators are ticker-agnostic and work on any liquid instrument without recalibration.

**No backtest capability.** These are `indicator()` scripts, not `strategy()` scripts. They cannot be run through TradingView's strategy tester. Conversion would require defining explicit entry/exit rules, position sizing, and stop/target logic.

**Equal-weighted conditions (1-min).** The 1-minute AND-gate treats all conditions as binary pass/fail with no relative weighting. The 5-minute, 15-minute, Market Monitor 5m, and Trend Compass scoring models partially address this by allowing differential contribution via weighted scoring.

**Market Monitor 5m is timeframe-specific.** While it will technically run on other intraday timeframes, session phase detection, OR duration defaults, HTF mapping (fixed to 15m), and scoring parameters are calibrated for 5-minute bars. For other timeframes, use the general-purpose Market Monitor.

**Trend Compass divergence detection has inherent lag.** Pivot-based divergence detection requires N right bars of confirmation before a divergence is confirmed. On the Daily variant (3R default), this means ~3 days of delay. On the 4H variant (2R default), ~8 hours. This is fundamental to pivot-based detection and cannot be eliminated without introducing false signals.

**Trend Compass 52-week H/L requires history.** The Daily variant needs 252 bars of daily history. The 4H variant pulls this from the Daily timeframe via `request.security()`. Very recently listed instruments may not have sufficient data.

## Project Structure

```
├── README.md
├── CHANGELOG.md
├── market_monitor/
│   ├── market_monitor_v1_0.pine
│   └── market_monitor_5min_v1_0.pine
├── spy_0dte_scalper/
│   ├── spy_0dte_scalper_1min_v1_0.pine
│   ├── spy_0dte_scalper_1min_v1_1.pine
│   ├── spy_0dte_scalper_5min_v1_0.pine
│   ├── spy_0dte_scalper_5min_v1_1.pine
│   ├── spy_0dte_scalper_15min_v1_0.pine
│   └── spy_0dte_scalper_15min_v1_1.pine
├── trend_compass/
│   ├── trend_compass_daily_v1_0.pine
│   └── trend_compass_4h_v1_0.pine
└── docs/
    ├── market_monitor_v1_0.md
    ├── market_monitor_5min_v1_0.md
    ├── spy_0dte_scalper_1min_v1.0.md
    ├── spy_0dte_scalper_1min_v1.1.md
    ├── spy_0dte_scalper_5min_v1.0.md
    ├── spy_0dte_scalper_5min_v1.1.md
    ├── spy_0dte_scalper_15min_v1_0.md
    ├── spy_0dte_scalper_15min_v1_1.md
    ├── trend_compass_daily_v1_0.md
    └── trend_compass_4h_v1_0.md
```

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history and release notes.

## Roadmap

Planned additions to the toolkit, roughly ordered by priority. Development order may shift based on what's most useful in practice.

- [ ] **Market Internals Dashboard** -- Standalone lower-pane indicator consolidating NYSE TICK, ADD (advance-decline), VOLD (up/down volume), and VIX term structure into a single "market health" view. Reduces redundant `request.security()` calls across instances and answers whether broad market breadth is confirming or diverging from SPY price action.

- [ ] **Volume Profile Approximation** -- Price-binned volume accumulation over configurable lookback windows to approximate POC (point of control), VAH/VAL (value area high/low), and high-volume node detection. These levels are among the most actionable for 0DTE entries and would complement existing VWAP and prior-day level infrastructure.

- [ ] **QQQ / IWM 0DTE Scalper Variants** -- Adapt the SPY 0DTE Scalper architecture to QQQ and IWM. Not a parameter tweak — each instrument needs independent calibration of EMA lengths, VWAP extension thresholds, RSI ranges, signal engine weights, and TICK symbol mapping (NYSE TICK vs. NASDAQ TICK).

- [ ] **Strategy Backtest Conversions** -- Convert `indicator()` scripts to `strategy()` equivalents with entry/exit rules, position sizing, stop-loss/take-profit logic, and commission modeling for TradingView's strategy tester. Primary challenge: codifying exit logic (currently trader discretion) into deterministic rules without overfitting.

- [ ] **Session Statistics Overlay** -- Companion overlay for end-of-day and intra-session performance stats: signal count, win rate (requires strategy conversion or manual tracking hooks), average bars held, R-multiple distribution, time-of-day signal density, and regime breakdown.

- [ ] **Options-Aware Enhancements** -- Indicators accounting for options-specific dynamics: expected move range (from VIX or ATM straddle pricing), gamma exposure level estimation, and time-decay-aware signal filtering that adjusts aggressiveness based on remaining session time (aggressive early, conservative into close as theta accelerates on 0DTE contracts).

- [ ] **Mean Reversion / Fade Signals** -- Complementary mean-reversion indicator targeting overextended moves: Bollinger Band extremes, RSI divergence, VWAP band rejection, and z-score deviation from intraday TWAP. Particularly useful during RANGING regime when trend-following signals are suppressed.

- [ ] **Multi-Asset Correlation Monitor** -- Real-time correlation and divergence tracking across related instruments (SPY/QQQ, SPY/IWM, SPY/VIX, 10Y yield/equities). Alerts on correlation breakdowns signaling rotation, risk-off events, or tradable divergence setups.

- [ ] **Alert Pipeline Templates** -- Pre-built webhook payload templates and integration guides for routing TradingView alerts to external systems (Discord, Telegram, Slack, custom APIs). Includes JSON payload formatting, conditional routing logic, and downstream automation documentation.

## License

MIT License. See [LICENSE](LICENSE).
