# Pine Scripts for TradingView

A growing collection of technical trading indicators and strategy tools for the TradingView charting platform, written in Pine Script v6.

---

## Overview

This repository is an evolving toolkit for systematic intraday trading on TradingView. The current focus is SPY 0DTE options scalping and multi-chart directional bias monitoring, with plans to expand into broader market internals, multi-asset strategies, and backtesting frameworks.

Currently, the suite includes a family of SPY 0DTE Scalper indicators spanning three timeframes (1-minute, 5-minute, 15-minute), each independently calibrated for its target holding period, plus a general-purpose Market Monitor for watchlist reconnaissance. All indicators share a common architectural foundation: EMA ribbon overlays, session-anchored VWAP with deviation bands, regime classification, and real-time dashboards with color-coded status fields. They diverge in signal generation approach, parameter calibration, and feature depth based on the characteristics of their target timeframe.

---

## Current Indicators

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

A directional bias overlay designed for multi-chart watchlist monitoring. Combines EMA ribbon, VWAP bands, RSI, ADX/DMI, higher-timeframe EMA confirmation, and relative volume into a composite bias score (-5 to +5) displayed in a compact dashboard. Optimized for running 6-8 instances simultaneously on small chart tiles.

This is not a signal generator. It answers one question: "which way is the wind blowing?" Use it for reconnaissance across a watchlist, then drill into the 0DTE Scalper on whichever charts show strong directional bias.

| Script | Version | Docs |
| ------ | ------- | ---- |
| [market_monitor_v1_0.pine](market_monitor/market_monitor_v1_0.pine) | v1.0 | [docs](docs/market_monitor_v1_0.md) |

---

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

---

## Core Components

Every indicator in the suite shares these foundational components, calibrated per-timeframe:

**Trend Structure** -- EMA Ribbon (3 EMAs) with dynamic cloud fill that shifts color based on alignment state (bullish, bearish, mixed). Forms the visual trend backbone on all chart overlays.

**Institutional Anchor** -- Session-anchored VWAP with configurable standard deviation bands. Auto-disables on non-intraday timeframes. VWAP cross markers (diamond shapes) flag reclaims and rejections.

**Key Price Levels** -- Pre-market high/low, prior day high/low/close, opening range (configurable duration), and real-time session HOD/LOD with dynamically updating lines.

**Regime Classification** -- Five-mode classifier (BULLISH, BEARISH, RANGING, NO TRADE, TRANSITION) based on EMA alignment, ADX strength, and RSI positioning. Provides context for signal interpretation.

**Dashboard** -- Real-time table overlay with color-coded status fields covering all active indicators. Configurable size (Tiny/Small/Normal) and corner positioning for multi-chart layouts.

**Alert System** -- Dual implementation: static `alertcondition()` for TradingView's standard alert UI, plus dynamic `alert()` with interpolated context for webhook and notification pipelines.

### v1.1 Additions (All Timeframes)

**TTM Squeeze** -- Bollinger Band (20, 2.0) compression inside Keltner Channels (20, 1.5). Three states: SQUEEZE ON (compression building), FIRED (breakout beginning), OFF (normal volatility). Momentum histogram via linear regression.

**NYSE TICK Index** -- Real-time market breadth via `request.security("USI:TICK", "1", close)`. Six-tier classification from EXTREME BEAR to EXTREME BULL with configurable thresholds.

---

## Installation

1. Open TradingView and navigate to a **SPY chart** at the appropriate timeframe.
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

**NYSE TICK** (v1.1 only): The TICK index (`USI:TICK`) requires your TradingView data plan to include the relevant exchange. If unavailable, TICK rows show "N/A" and TICK-based signal conditions pass through without blocking (1-min) or simply don't contribute score (5-min, 15-min).

---

## Recommended Layouts

### Single-Timeframe Focus

Run one SPY 0DTE Scalper on a full-screen chart. Best for dedicated scalping sessions where you want maximum visual clarity.

### Multi-Timeframe Stack

Run 2-3 SPY 0DTE Scalper variants side-by-side (e.g., 1-min + 5-min, or 5-min + 15-min). The higher timeframe provides context for entries on the lower timeframe. Note that the 5-min and 15-min variants already pull lower-timeframe data internally via MTF confirmation, so running the lower-timeframe chart alongside is optional but provides additional visual context.

### Watchlist Reconnaissance

Run 6-8 Market Monitor instances on small chart tiles across your watchlist. Use Tiny or Small dashboard size with Top Right positioning. Identify which tickers show strong directional bias (score >= +3 or <= -3), then open a dedicated chart with the appropriate 0DTE Scalper for signal generation.

---

## Performance and Limits

All indicators are optimized for simultaneous multi-instance execution on TradingView.

| Resource | Pine Script v6 Limit | Typical Usage (per instance) |
| -------- | -------------------- | ---------------------------- |
| `request.security()` calls | 40 per script | 4-5 (scalpers), 3 (monitor) |
| Labels | 500 per script | ~50-100 per full session |
| Lines | 500 per script | ~20-40 (level lines) |
| Boxes | 200 per script | ~5-10 (opening range, clouds) |

**Computation**: No loops, no arrays, no maps. Pure vectorized calculations with `var` declarations for persistent state. Dashboard tables update only on `barstate.islast` to minimize historical bar overhead.

**Repainting**: All signal logic is gated by `barstate.isconfirmed` by default. Signals appear after bar close, not during formation. `request.security()` calls use `lookahead=barmerge.lookahead_off` for all forward-looking data to prevent future data leakage. Prior day levels use `lookahead_on` which is appropriate for historical reference levels.

---

## Known Limitations

**No cumulative delta / order flow.** Pine Script lacks access to tick-level bid/ask data. Relative volume and NYSE TICK are the closest available proxies.

**No volume profile.** Pine Script cannot natively compute POC, VAH, VAL. Use TradingView's built-in Volume Profile tool or a separate indicator alongside.

**Single-symbol calibration.** All parameters are calibrated for SPY's price range, volatility, and microstructure. Other instruments require significant parameter adjustment.

**No backtest capability.** These are `indicator()` scripts, not `strategy()` scripts. They cannot be run through TradingView's strategy tester. Conversion would require defining explicit entry/exit rules, position sizing, and stop/target logic.

**Equal-weighted conditions (1-min).** The 1-minute AND-gate treats all conditions as binary pass/fail with no relative weighting. The 5-minute and 15-minute scoring models partially address this by allowing differential contribution.

---

## Project Structure

```
├── README.md
├── market_monitor/
│   └── market_monitor_v1_0.pine
├── spy_0dte_scalper/
│   ├── spy_0dte_scalper_1min_v1_0.pine
│   ├── spy_0dte_scalper_1min_v1_1.pine
│   ├── spy_0dte_scalper_5min_v1_0.pine
│   ├── spy_0dte_scalper_5min_v1_1.pine
│   ├── spy_0dte_scalper_15min_v1_0.pine
│   └── spy_0dte_scalper_15min_v1_1.pine
└── docs/
    ├── market_monitor_v1_0.md
    ├── spy_0dte_scalper_1min_v1.0.md
    ├── spy_0dte_scalper_1min_v1.1.md
    ├── spy_0dte_scalper_5min_v1.0.md
    ├── spy_0dte_scalper_5min_v1.1.md
    ├── spy_0dte_scalper_15min_v1_0.md
    └── spy_0dte_scalper_15min_v1_1.md
```

---

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history and release notes.

---

## Roadmap

The following categories represent planned additions to the toolkit. Items are roughly ordered by priority within each category, but development order may shift based on what's most useful in practice.

### Market Internals Dashboard

A dedicated lower-pane or overlay indicator consolidating real-time breadth and internals data into a single view. Planned components include NYSE TICK (already integrated per-indicator, but a standalone composite would reduce redundant `request.security()` calls across instances), NYSE ADD (advance-decline), VOLD (up-volume vs. down-volume), and VIX term structure slope. The goal is a single "market health" indicator that answers whether the broad market is confirming or diverging from SPY price action.

### Volume Profile Approximation

Pine Script cannot access TradingView's native volume profile data, but an approximation using price-binned volume accumulation over configurable lookback windows is feasible. Targeted outputs: POC (point of control), VAH/VAL (value area high/low), and high-volume node detection. These levels are among the most actionable for 0DTE entries and would complement the existing VWAP and prior-day level infrastructure.

### QQQ / IWM 0DTE Scalper Variants

Adapt the SPY 0DTE Scalper architecture to QQQ and IWM, which have meaningfully different volatility profiles, ATR ranges, and microstructure characteristics. This is not a parameter tweak -- each instrument needs independent calibration of EMA lengths, VWAP extension thresholds, RSI ranges, and signal engine weights. TICK symbol mapping (NYSE TICK vs. NASDAQ TICK) also differs.

### Strategy Backtest Conversions

Convert existing `indicator()` scripts to `strategy()` equivalents with defined entry/exit rules, position sizing, stop-loss/take-profit logic, and commission modeling. This enables TradingView's strategy tester for historical performance evaluation. The primary challenge is codifying exit logic (currently left to trader discretion) into deterministic rules without overfitting.

### Session Statistics Overlay

End-of-day and intra-session performance statistics: signal count, win rate (requires strategy conversion or manual tracking hooks), average bars held, R-multiple distribution, time-of-day signal density, and regime breakdown. Designed as a companion overlay that reads signal data from the scalper indicators.

### Options-Aware Enhancements

Indicators that account for options-specific dynamics beyond directional bias. Candidates include expected move range (derived from VIX or ATM straddle pricing), gamma exposure level estimation, and time-decay-aware signal filtering that adjusts aggressiveness based on remaining session time (aggressive early, conservative into close as theta accelerates on 0DTE contracts).

### Mean Reversion / Fade Signals

The current suite is trend-following by design. A complementary mean-reversion indicator would target overextended moves for fade entries: Bollinger Band extremes, RSI divergence, VWAP band rejection, and z-score-based deviation from intraday TWAP. Particularly useful during RANGING regime classification when trend-following signals are suppressed.

### Multi-Asset Correlation Monitor

Real-time correlation and divergence tracking across related instruments (SPY vs. QQQ, SPY vs. IWM, SPY vs. VIX, 10Y yield vs. equity indices). Alerts on correlation breakdowns that may signal rotation, risk-off events, or tradable divergence setups. Would extend the Market Monitor concept from single-instrument bias to cross-asset regime awareness.

### Alert Pipeline Templates

Pre-built webhook payload templates and companion integration guides for routing TradingView alerts to external systems (Discord, Telegram, Slack, custom APIs). Includes JSON payload formatting, conditional routing logic, and documentation for connecting to downstream automation or logging infrastructure.

---

## License

MIT License. See [LICENSE](LICENSE).
