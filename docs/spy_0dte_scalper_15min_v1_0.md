# 0DTE SPY Scalping Indicator (15-Minute) v1.0

**Platform**: TradingView
**Language**: Pine Script v6
**Chart**: SPY 15-Minute (optimized)
**Theme**: Dark theme (vibrant, high-contrast palette)

---

## Overview

A single-overlay indicator purpose-built for establishing directional bias and identifying high-conviction swing entries for 0DTE (zero days to expiration) SPY options on the 15-minute chart. It combines trend structure, momentum, volatility, volatility compression detection (TTM Squeeze), market breadth (NYSE TICK with SMA smoothing), key price levels, a multi-timeframe confirmation layer (5-minute), and an anchor EMA for structural context into a unified decision-support system with real-time signal generation and a 21-row heads-up dashboard.

This is the 15-minute companion to the 1-minute and 5-minute SPY 0DTE Scalpers. It targets the highest-conviction directional swings with 15-60 minute hold times. The 15-minute chart produces ~26 RTH bars per session, so the signal engine is calibrated for maximum conviction per signal with the lowest total signal count across all three variants.

**Role in the multi-chart hierarchy**:

- **1-minute**: Micro-entry timing, rapid scalps (1-5 min holds)
- **5-minute**: Swing-scalp trades, moderate conviction (5-25 min holds)
- **15-minute** (this): Directional bias, structural trend context, high-conviction swings (15-60 min holds)

The 15-minute chart is the "big picture" view. Its primary value is establishing whether the session trend is bullish, bearish, ranging, or in transition. Signals on this timeframe represent major structural inflection points. When the 15-minute and 5-minute agree on direction, the 1-minute can be used for precision entry timing.

The indicator does not auto-trade. It surfaces high-confluence setups as CALLS or PUTS labels on the chart, backed by a scoring engine that evaluates up to eleven independent conditions per bar. All thresholds are configurable.

---

## Table of Contents

- [Features](#features)
- [Key Differences from 1-Minute and 5-Minute Variants](#key-differences-from-1-minute-and-5-minute-variants)
- [Installation](#installation)
- [Indicator Components](#indicator-components)
  - [EMA Ribbon + Anchor EMA](#ema-ribbon--anchor-ema)
  - [VWAP and Bands](#vwap-and-bands)
  - [RSI](#rsi)
  - [ADX / DMI](#adx--dmi)
  - [ATR](#atr)
  - [TTM Squeeze](#ttm-squeeze)
  - [NYSE TICK Index (Smoothed)](#nyse-tick-index-smoothed)
  - [Multi-Timeframe Confirmation (5-Min)](#multi-timeframe-confirmation-5-min)
  - [Key Price Levels](#key-price-levels)
  - [Regime / Mode Detection](#regime--mode-detection)
  - [Signal Engine](#signal-engine)
  - [Directional Bias Background](#directional-bias-background)
  - [Dashboard](#dashboard)
  - [Alerts](#alerts)
- [Visual Reference](#visual-reference)
  - [Color Map](#color-map)
  - [Chart Element Legend](#chart-element-legend)
  - [Dashboard Fields](#dashboard-fields)
- [Tuning Guide](#tuning-guide)
  - [Minimum Signal Score](#minimum-signal-score)
  - [EMA Lengths](#ema-lengths)
  - [Anchor EMA Length](#anchor-ema-length)
  - [ADX Thresholds](#adx-thresholds)
  - [VWAP Extended Distance](#vwap-extended-distance)
  - [Signal Cooldown](#signal-cooldown)
  - [Signal Time Window](#signal-time-window)
  - [Opening Range Duration](#opening-range-duration)
  - [RSI Thresholds](#rsi-thresholds)
  - [TTM Squeeze Settings](#ttm-squeeze-settings)
  - [NYSE TICK Settings](#nyse-tick-settings)
  - [Multi-Timeframe Toggle](#multi-timeframe-toggle)
  - [Directional Bias Background](#directional-bias-background-1)
  - [Tuning Profiles](#tuning-profiles)
- [Architecture Notes](#architecture-notes)
- [Known Limitations](#known-limitations)
- [Changelog](#changelog)

---

## Features

- **EMA Ribbon** (10/21/34) with dynamic cloud fill that shifts color based on trend alignment (bullish, bearish, mixed). Wider EMA windows tuned for 15-min bar dynamics, capturing 2.5-8.5 hour swings.
- **Anchor EMA** (50-period, default) plotted as a thicker structural reference line. 50 bars on a 15-min chart = ~12.5 hours (~2 full sessions). Integrated as an enhancement scoring condition (#11) for structural trend alignment.
- **Session-anchored VWAP** with configurable standard deviation bands.
- **RSI, ADX, and ATR** calculated internally and displayed in the dashboard (not plotted as separate panes, preserving chart real estate).
- **TTM Squeeze detection** identifying Bollinger Band compression inside Keltner Channels. Three states: SQUEEZE ON (compression building), FIRED (breakout beginning), and OFF (normal volatility). Integrated as an enhancement scoring condition with a 2-bar recency window (~30 min).
- **NYSE TICK Index with SMA smoothing** pulled via `request.security()`. Raw TICK from a single 1-min bar is noisy on a 15-min timeframe. Both raw and 5-period SMA-smoothed TICK are pulled; the smoothed value is used for scoring while both are displayed in the dashboard. Six-tier classification from EXTREME BEAR to EXTREME BULL with lowered thresholds calibrated for the smoothed signal.
- **Multi-timeframe confirmation layer (5-minute)** pulling 5-min RSI, 5-min EMA trend alignment, and 5-min MACD histogram via `request.security()` for lower-timeframe context. MACD is new to the 15-min variant, providing momentum confirmation that complements trend and oscillator data.
- **Pre-Market High/Low** automatically detected and drawn as horizontal levels once RTH begins.
- **Prior Day High/Low/Close** pulled from the daily timeframe and plotted as dotted reference levels.
- **Opening Range** (default 30 minutes / 2 bars) captured at RTH open and drawn as dashed levels. 30-minute OR is the standard institutional reference for identifying the session's initial balance area.
- **Session HOD/LOD** tracked in real time with dynamically updating lines.
- **Session progress indicator** in the dashboard showing current bar count out of ~26 total RTH bars, with EARLY SESSION and LATE SESSION contextual labels.
- **Regime Classifier** that categorizes the current market state into one of five modes: BULLISH, BEARISH, RANGING, NO TRADE, or TRANSITION.
- **5+6 confluence scoring engine** with 5 core conditions and 6 optional enhancement conditions (MTF trend, MTF MACD momentum, level proximity, squeeze recency, TICK breadth, anchor EMA alignment). Generates CALLS or PUTS labels when the score meets the configurable threshold. Max score: 11.
- **Expanded candle pattern detection** optimized for 15-min bars: engulfing, strong close (60% body), hammer/shooting star (1.6x tail), inside bar breakout, 3-bar momentum, and morning star/evening star (3-bar reversal). Morning/evening star patterns are new to the 15-min variant, as 3-bar patterns on 15-min represent ~45 minutes of price action and are more meaningful than on shorter timeframes.
- **Directional bias background** (optional, off by default): subtle full-bar background tint when the regime is BULLISH or BEARISH, providing constant visual context without requiring dashboard attention.
- **VWAP cross markers** (diamond shapes) for quick visual identification of VWAP reclaims and rejections.
- **Real-time dashboard** (table overlay) showing 21 fields of live indicator data including anchor EMA, MTF (trend, RSI, MACD), squeeze, TICK (raw + smoothed), and session progress rows, with color-coded status text.
- **Dual alert system**: static `alertcondition()` entries for TradingView's standard alert UI (including squeeze fired), plus dynamic `alert()` calls with interpolated context including score, anchor EMA, MTF, MACD, squeeze, and TICK status.
- **Bar confirmation gate** to prevent signals from firing on incomplete (still-forming) bars.
- **Signal cooldown** (default 2 bars = 30 minutes) to prevent label spam.

---

## Key Differences from 1-Minute and 5-Minute Variants

| Aspect | 1-Minute | 5-Minute | 15-Minute | Why (15-min) |
|--------|----------|----------|-----------|---------------|
| EMA Lengths | 8/13/21 | 9/15/21 | 10/21/34 | 15-min bars need much wider windows; 10/21/34 captures 2.5-8.5 hr swings and aligns with Fibonacci-adjacent periods |
| Anchor EMA | N/A | N/A | 50-period | NEW: structural trend reference spanning ~2 sessions. Scored as condition #11. |
| RSI Length | 14 | 10 | 9 | 15-min bars are very smooth; RSI(14) is sluggish. RSI(9) = 2.25 hr lookback, responsive to intraday swings |
| RSI OB/OS | 70/30 | 65/35 | 60/40 | 15-min RSI almost never reaches 70/30 intraday; 60/40 catches exhaustion at the structural level |
| ADX No-Trend | 15 | 18 | 20 | ADX reads higher on very smooth bars; 20 provides a meaningful no-trend gate at this timeframe |
| Opening Range | 5 min (5 bars) | 15 min (3 bars) | 30 min (2 bars) | 30-min OR is the institutional standard for session initial balance |
| Signal Cooldown | 5 bars (5 min) | 3 bars (15 min) | 2 bars (30 min) | Only ~26 bars/session; 2-bar cooldown balances spam prevention with limited signal opportunities |
| Signal Window Start | 09:35 | 09:45 | 10:00 | First 2 bars establish OR; signals start after OR completes + 1 buffer bar |
| VWAP Ext Filter | 1.5 ATR | 2.0 ATR | 2.5 ATR | 15-min ATR is larger absolute; wider multiplier avoids premature extension flags on structural moves |
| Candle Patterns | 4 patterns | 6 patterns | 8 patterns | Added morning star / evening star (3-bar reversal); meaningful on 15-min where 3 bars = ~45 min of price action |
| Strong Close Threshold | 50% | 55% | 60% | 15-min candles have proportionally larger wicks; 60% body is genuinely strong |
| Hammer/Star Threshold | 2.0x body | 1.8x body | 1.6x body | 15-min patterns show less extreme tail ratios |
| Scoring Model | AND-gate | 5+4 (max 9) | 5+6 (max 11) | More enhancement conditions available: anchor EMA + MTF MACD |
| MTF Source | N/A | 1-min (RSI + EMA) | 5-min (RSI + EMA + MACD) | 5-min is the natural lower-TF for 15-min context |
| MTF MACD | N/A | N/A | 5-min MACD histogram | NEW: momentum confirmation layer; MACD(12,26,9) on 5-min provides ~1hr/~2.2hr effective lookback |
| TICK Handling | Raw 1-min | Raw 1-min | Raw + SMA(5) smoothed | 15-min needs noise reduction; SMA(5) smoothing on 1-min TICK represents ~5 min average breadth |
| TICK Thresholds | 500/-500/800 | 500/-500/800 | 300/-300/600 | Smoothed TICK produces lower absolute values; thresholds calibrated for the smoothed signal |
| Squeeze Recency | 5 bars (~5 min) | 3 bars (~15 min) | 2 bars (~30 min) | 2 bars = ~30 min recency window; appropriate for 15-min structural plays |
| Level Proximity | N/A (1-min) | 1.0 ATR | 1.5 ATR | 15-min ATR is larger; widened proximity radius for structural level detection |
| Bias Background | N/A | N/A | Optional (off default) | NEW: subtle background tint for constant directional context |
| Session Progress | N/A | N/A | Bar count + Early/Late labels | NEW: awareness of session phase with ~26 bars/session |
| Dashboard Rows | 16 | 18 | 21 | Added anchor EMA, 5m MACD, session progress rows |
| Line Extend | 300 bars | 60 bars | 40 bars | 15-min sessions are ~26 bars; 40 covers ~1.5 sessions |
| Arrow Size | tiny | tiny | small | Larger arrows on the sparser 15-min chart for better visibility |

---

## Installation

1. Open TradingView and navigate to a **SPY 15-minute chart**.
2. Open the **Pine Editor** (bottom panel).
3. Delete any existing code in the editor.
4. Paste the entire contents of `spy_0dte_scalper_15min_v1_0.pine` into the editor.
5. Click **"Add to chart"** (or "Save" then "Add to chart").
6. The indicator appears as an overlay with the dashboard in the top-right corner.

**Required chart settings**:

- **Extended hours**: Enable "Extended Trading Hours" in chart settings if you want Pre-Market High/Low levels to populate. Without extended hours data, those fields will show "N/A" in the dashboard and no PM level lines will render.
- **Timezone**: ensure your chart timezone matches the indicator's timezone input (default: `America/New_York`).

---

## Indicator Components

### EMA Ribbon + Anchor EMA

Three exponential moving averages form the core trend structure with a dynamic cloud fill between the fast and slow EMAs. A fourth long-period "anchor" EMA provides multi-session structural context.

| Parameter | Default | Purpose |
|-----------|---------|---------|
| Fast EMA | 10 | Short-term direction (10 bars = 2.5 hrs) |
| Mid EMA | 21 | Intermediate trend (21 bars = 5.25 hrs) |
| Slow EMA | 34 | Structural trend (34 bars = 8.5 hrs = ~1.3 sessions) |
| Anchor EMA | 50 | Multi-session structure (50 bars = 12.5 hrs = ~2 sessions) |

**Cloud fill states**: green (bullish: fast > mid > slow), red (bearish: fast < mid < slow), yellow (mixed/transitioning). The anchor EMA is plotted as a purple line with `linewidth=2` and is not included in the cloud fill. It serves as a "line in the sand" for structural trend direction and is scored as enhancement condition #11.

### VWAP and Bands

Session-anchored VWAP with manually calculated standard deviation bands using cumulative variance.

| Parameter | Default | Purpose |
|-----------|---------|---------|
| Show VWAP | true | Master toggle |
| Show Bands | true | +/- 1 StdDev bands |
| Band Multiplier | 1.0 | StdDev scaling factor |

VWAP is the primary mean-reversion anchor. Price relationship to VWAP (above/below, distance) is a core scoring condition (#1). VWAP distance in ATR multiples is displayed in the dashboard.

### RSI

| Parameter | Default | Purpose |
|-----------|---------|---------|
| Length | 9 | RSI lookback (9 bars = 2.25 hrs) |
| Overbought | 60 | Upper extreme threshold |
| Oversold | 40 | Lower extreme threshold |

RSI(9) on a 15-min chart is responsive enough to catch intraday exhaustion while smoothing out single-bar noise. The tighter 60/40 thresholds reflect the reality that 15-min RSI rarely reaches 70/30 during a single session. RSI being in the neutral zone (between OS and OB) is a core scoring condition (#3) — signals are suppressed when RSI is at extremes to avoid chasing.

### ADX / DMI

| Parameter | Default | Purpose |
|-----------|---------|---------|
| ADX Length | 14 | ADX/DI lookback period |
| No-Trend | 20 | Below this = no meaningful trend |
| Trending | 25 | Above this = confirmed trend |
| Strong Trend | 40 | Above this = strong directional move |

ADX calibration at 20 for the no-trend threshold reflects the natural smoothing of 15-min bars. ADX above the no-trend threshold is both a signal gate (when enabled) and a core scoring condition (#5).

### ATR

| Parameter | Default | Purpose |
|-----------|---------|---------|
| Length | 14 | ATR lookback (14 bars = 3.5 hrs) |

ATR serves as the volatility normalizer for VWAP distance filtering, level proximity detection, and the extension guard. The 15-min ATR is displayed in the dashboard for manual reference.

### TTM Squeeze

Detects volatility compression by comparing Bollinger Band width to Keltner Channel width.

| Parameter | Default | Purpose |
|-----------|---------|---------|
| BB Length | 20 | Bollinger Band SMA period |
| BB Multiplier | 2.0 | Bollinger Band width |
| KC Length | 20 | Keltner Channel EMA period |
| KC Multiplier | 1.5 | Keltner Channel width (ATR-based) |

**Three states**: SQUEEZE ON (BB inside KC — compression building, volatility coiling), FIRED (BB exits KC — breakout beginning), OFF (normal volatility). A linear regression momentum histogram measures directional energy during and after the squeeze.

The squeeze is integrated as enhancement scoring condition #9. A squeeze that fired within the last 2 bars (~30 minutes) contributes +1 to the signal score. On the 15-min chart, a squeeze represents significant multi-hour compression and its resolution carries more weight than on shorter timeframes.

### NYSE TICK Index (Smoothed)

| Parameter | Default | Purpose |
|-----------|---------|---------|
| TICK Symbol | USI:TICK | NYSE TICK source |
| Bullish Threshold | 300 | Smoothed TICK above this = bullish breadth |
| Bearish Threshold | -300 | Smoothed TICK below this = bearish breadth |
| Extreme Threshold | 600 | Smoothed TICK beyond this = extreme breadth |

**Why smoothing matters on 15-min**: A single 1-min TICK reading captured at the close of a 15-min bar can be wildly unrepresentative of the actual breadth environment during those 15 minutes. The indicator pulls both raw and SMA(5)-smoothed TICK from the 1-min timeframe. The smoothed value represents the "average breadth over the last 5 minutes" and is used for scoring. Both values are displayed in the dashboard as "raw (smoothed)".

The smoothed thresholds are lower than the 1-min/5-min variants (300/-300/600 vs 500/-500/800) because the SMA naturally compresses the range. These thresholds are calibrated so that a sustained bullish or bearish breadth environment will register appropriately.

TICK is integrated as enhancement scoring condition #10.

### Multi-Timeframe Confirmation (5-Min)

| Parameter | Default | Purpose |
|-----------|---------|---------|
| Enable MTF | true | Master toggle for all 5-min pulls |
| 5-Min RSI Length | 10 | RSI on 5-min bars (matches 5-min variant default) |
| 5-Min Fast EMA | 9 | Fast EMA on 5-min bars |
| 5-Min Slow EMA | 21 | Slow EMA on 5-min bars |

Pulls three categories of 5-minute data:

1. **5-Min EMA Trend**: fast vs slow EMA alignment on 5-min bars. Scored as enhancement condition #6.
2. **5-Min RSI**: oscillator state on 5-min bars. Displayed in dashboard; not directly scored (prevents double-counting with the native RSI condition).
3. **5-Min MACD Histogram**: MACD(12,26,9) histogram direction on 5-min bars. NEW to the 15-min variant. Provides momentum confirmation that complements trend alignment. Scored as enhancement condition #7.
4. **5-Min VWAP Cross**: detects recent VWAP crosses on 5-min for dashboard awareness. Not directly scored.

### Key Price Levels

| Level | Source | Style | Purpose |
|-------|--------|-------|---------|
| Pre-Market High/Low | Session tracking | Dashed | PM range boundaries |
| Prior Day High/Low/Close | Daily `request.security()` | Dotted | Prior session reference |
| Opening Range High/Low | First 2 bars (30 min) | Dashed | Session initial balance |
| Session HOD/LOD | Real-time tracking | Solid | Current session extremes |

Line extend distances are set to 40 bars (appropriate for 15-min session length of ~26 bars). Labels are offset 6 bars to the right.

Level proximity is scored as enhancement condition #8. "Near support" for CALLS and "near resistance" for PUTS, using a 1.5 ATR proximity radius (widened from 1.0 ATR on the 5-min variant to account for larger 15-min ATR values).

### Regime / Mode Detection

Five-state classifier based on ADX trend strength, EMA alignment, and VWAP position:

| Mode | Conditions | Dashboard Color | Advice |
|------|-----------|-----------------|--------|
| BULLISH | ADX >= no-trend, EMA bullish, close > VWAP | Green | ACTIVE |
| BEARISH | ADX >= no-trend, EMA bearish, close < VWAP | Red | ACTIVE |
| RANGING | ADX < no-trend, not near VWAP | Yellow | CAUTION |
| TRANSITION | ADX >= trending, EMAs not aligned | Orange | ACTIVE |
| NO TRADE | ADX < no-trend, close near VWAP (< 0.5 ATR) | Gray | AVOID |

On the 15-min chart, mode changes are significant structural events. A mode shift alert fires whenever the regime changes, providing awareness of larger trend transitions.

### Signal Engine

The 15-min signal engine uses a **5+6 confluence scoring model** — 5 core conditions plus 6 optional enhancement conditions, for a maximum score of 11.

**Gate conditions** (must all pass — these are AND-gated prerequisites, not scored):

- Bar is confirmed (closed)
- Within the signal time window (default 10:00-15:45 ET)
- ADX above no-trend threshold (when enabled)
- Price not VWAP-extended (within 2.5 ATR of VWAP)
- Cooldown elapsed (2+ bars since last signal)

**Core conditions** (scored 0 or 1 each):

| # | Condition | CALLS Logic | PUTS Logic |
|---|-----------|-------------|------------|
| 1 | VWAP position | Close > VWAP or reclaiming | Close < VWAP or rejecting |
| 2 | EMA alignment | Fast > Slow or crossing above | Fast < Slow or crossing below |
| 3 | RSI neutral zone | Between OS and OB (40-60) | Between OS and OB (40-60) |
| 4 | Candle pattern | Bullish engulfing, strong close, hammer, inside bar breakout, 3-bar momentum up, morning star | Bearish engulfing, strong close, shooting star, inside bar breakout, 3-bar momentum down, evening star |
| 5 | ADX trend present | ADX >= no-trend threshold | ADX >= no-trend threshold |

**Enhancement conditions** (scored 0 or 1 each):

| # | Condition | CALLS Logic | PUTS Logic |
|---|-----------|-------------|------------|
| 6 | 5m EMA trend | 5-min fast EMA > 5-min slow EMA | 5-min fast EMA < 5-min slow EMA |
| 7 | 5m MACD momentum | 5-min MACD histogram > 0 | 5-min MACD histogram < 0 |
| 8 | Level proximity | Near support (PM Low, OR Low, PDL, LOD) | Near resistance (PM High, OR High, PDH, HOD) |
| 9 | Squeeze fired | Squeeze fired within last 2 bars (~30 min) | Squeeze fired within last 2 bars (~30 min) |
| 10 | TICK breadth | Smoothed TICK > bullish threshold | Smoothed TICK < bearish threshold |
| 11 | Anchor EMA | Close + fast EMA above anchor EMA | Close + fast EMA below anchor EMA |

A signal fires when the gate passes AND the total score >= `i_minScore` (default: 5).

**Candle pattern details (15-min optimized)**:

| Pattern | Key Threshold | Notes |
|---------|---------------|-------|
| Engulfing | Full body engulf | Unchanged from prior variants |
| Strong close | Body > 60% of range | Raised from 55% (5-min), 50% (1-min) |
| Hammer / Shooting star | Tail > 1.6x body | Relaxed from 1.8x (5-min), 2.0x (1-min) |
| Inside bar breakout | Prior bar inside bar[2], breakout close | Same logic as 5-min |
| 3-bar momentum | 3 consecutive directional closes | Same logic as 5-min |
| Morning star / Evening star | 3-bar reversal pattern | NEW: doji/indecision middle bar with reversal confirmation |

### Directional Bias Background

An optional (off by default) subtle background tint that applies when the regime is BULLISH or BEARISH. This provides a constant visual reminder of the current structural bias without requiring dashboard attention. Signal background flashes (brighter green/red on signal bars) take priority.

Enable via Settings > Directional Bias > "Show Directional Bias Background".

### Dashboard

21-row table positioned top-right with three columns: label, value, context.

| Row | Field | Value | Context/Label |
|-----|-------|-------|---------------|
| 0 | Header | "SPY 0DTE Scalper (15m) Dashboard" | — |
| 1 | Price | Current close | Green Day / Red Day |
| 2 | Mode | Regime state | ACTIVE / CAUTION / AVOID |
| 3 | RSI | RSI value | OVERBOUGHT / OVERSOLD / NEUTRAL |
| 4 | ADX | ADX value | NO TREND / WEAK / TRENDING / STRONG |
| 5 | VWAP Dist | Distance from VWAP | EXTENDED / NEAR VWAP |
| 6 | PM High | Distance to PM High | RESISTANCE / CLEARED |
| 7 | PM Low | Distance to PM Low | SUPPORT / BROKEN |
| 8 | ATR | ATR value | — |
| 9 | EMA Trend | Ribbon alignment | BULLISH / BEARISH / MIXED |
| 10 | Anchor EMA | Position relative to anchor | ABOVE / BELOW / NEUTRAL + price |
| 11 | Signal | Last signal direction | Score (e.g., "7/11") |
| 12 | Day Chg | Intraday % change | — |
| 13 | Rel Vol | Volume / 20-bar SMA | SPIKE / ABOVE AVG / BELOW AVG |
| 14 | DI Spread | DI+ / DI- values | BULLS / BEARS |
| 15 | 5m Trend | 5-min EMA alignment | BULLISH / BEARISH / MIXED |
| 16 | 5m RSI | 5-min RSI value | OVERBOUGHT / OVERSOLD / NEUTRAL |
| 17 | 5m MACD | 5-min MACD histogram | BULLISH / BEARISH / FLAT |
| 18 | Squeeze | TTM state | BUILDING / BREAKOUT / n bars ago / NORMAL |
| 19 | TICK | Raw (Smoothed) | EXTREME BULL/BEAR, BULLISH/BEARISH, MILD, FLAT |
| 20 | Session | Bar count / total | EARLY SESSION / LATE SESSION |

### Alerts

**Static alerts** (`alertcondition`):

- CALLS Signal
- PUTS Signal
- VWAP Cross Bullish
- VWAP Cross Bearish
- Regime Mode Change
- Squeeze Fired

**Dynamic alerts** (`alert`): include interpolated context with score (/11), anchor EMA status, MTF trend, MACD direction, squeeze state, and TICK values (raw + smoothed).

---

## Visual Reference

### Color Map

| Element | Color | Hex |
|---------|-------|-----|
| VWAP | White | #FFFFFF |
| VWAP Bands | Cyan | #00BCD4 |
| Fast EMA | Yellow | #FFD600 |
| Mid EMA | Orange | #FF9100 |
| Slow EMA | Red | #FF1744 |
| Anchor EMA | Purple | #7C4DFF |
| Bullish Cloud | Green (85% transparent) | #00E676 |
| Bearish Cloud | Red (85% transparent) | #FF1744 |
| Mixed Cloud | Yellow (90% transparent) | #FFD600 |
| PM High | Bright Red | #FF5252 |
| PM Low | Bright Green | #69F0AE |
| PDH | Deep Orange | #FF6E40 |
| PDL | Teal | #64FFDA |
| PDC | Blue Gray | #B0BEC5 |
| OR High | Purple | #E040FB |
| OR Low | Light Purple | #B388FF |
| CALLS | Green | #00E676 |
| PUTS | Red | #FF1744 |
| Dashboard Header | Purple | #7C4DFF |

### Chart Element Legend

| Element | Style | When Visible |
|---------|-------|--------------|
| EMA Ribbon | 3 solid lines + cloud fill | Always |
| Anchor EMA | Thick purple line | Always |
| VWAP | White line, width 2 | When enabled |
| VWAP Bands | Cyan circles | When enabled |
| PM High/Low | Dashed lines + labels | RTH, when enabled |
| PDH/PDL/PDC | Dotted lines + labels | When enabled |
| OR High/Low | Dashed lines + labels | After OR complete |
| HOD/LOD | Solid lines + labels | RTH, when enabled |
| CALLS Signal | Green triangle (small) + label + bg flash | On signal |
| PUTS Signal | Red triangle (small) + label + bg flash | On signal |
| VWAP Cross | Diamonds above/below bar | On confirmed cross |
| Bias Background | Very subtle green/red tint | When enabled + directional mode |

### Dashboard Fields

All dashboard fields update on the last bar only (`barstate.islast`) for performance.

---

## Tuning Guide

### Minimum Signal Score

The most impactful setting. With 5 core + 6 enhancement conditions (max 11):

| Score Threshold | Expected Behavior |
|-----------------|-------------------|
| 3-4 | Frequent signals; many will lack full confirmation. For experienced traders who manage risk through position sizing. |
| 5 (default) | All core conditions pass. Enhancement conditions provide additional conviction but are not required. |
| 6-7 | Core + 1-2 enhancements. Higher conviction, fewer signals. |
| 8+ | Very high bar. Only fires when most conditions align. Expect 0-3 signals per session. |
| 10-11 | Near-perfect alignment across all layers. May go entire sessions without firing. |

### EMA Lengths

| Setting | Fast / Mid / Slow | Character |
|---------|-------------------|-----------|
| Responsive | 8 / 13 / 21 | Faster reactions, more cloud color changes |
| Default | 10 / 21 / 34 | Balanced structural context |
| Smooth | 13 / 26 / 50 | Very slow, fewer transitions, highest conviction cloud state |

### Anchor EMA Length

| Setting | Effective Lookback | Character |
|---------|-------------------|-----------|
| 34 | ~8.5 hrs (1.3 sessions) | More responsive to recent trend |
| 50 (default) | ~12.5 hrs (2 sessions) | Standard multi-session structure |
| 100 | ~25 hrs (4 sessions) | Weekly structural trend |

### ADX Thresholds

On 15-min bars, ADX naturally reads higher due to smoothing. Recommended ranges:

| Sensitivity | No-Trend | Trending | Strong |
|-------------|----------|----------|--------|
| Sensitive | 16 | 22 | 35 |
| Default | 20 | 25 | 40 |
| Conservative | 24 | 30 | 45 |

### VWAP Extended Distance

Controls how far from VWAP price can be before signals are suppressed.

| Setting | Behavior |
|---------|----------|
| 1.5 ATR | Conservative — suppresses signals when price moves > 1.5 ATR from VWAP |
| 2.5 ATR (default) | Balanced — allows structural moves while filtering extremes |
| 4.0 ATR | Permissive — only filters very extended conditions |

### Signal Cooldown

| Setting | Effective Time | Behavior |
|---------|---------------|----------|
| 1 bar | 15 min | Maximum frequency — allows back-to-back signals |
| 2 bars (default) | 30 min | Balanced — prevents clustering |
| 3 bars | 45 min | Conservative — max ~8 signals per session |

### Signal Time Window

Default: 10:00-15:45 ET. The first two 15-min bars (9:30-10:00) establish the opening range. Signals begin after OR completes. The 15:45 cutoff leaves one 15-min bar before close.

| Profile | Window | Rationale |
|---------|--------|-----------|
| Conservative | 10:15-15:30 | Extra buffer after OR, avoid final 30 min |
| Default | 10:00-15:45 | Standard |
| Extended | 09:45-15:55 | Maximum window, includes OR breakout bar |

### Opening Range Duration

| Setting | Bars | Effective Time |
|---------|------|---------------|
| 15 min | 1 bar | Single bar — minimal but sometimes useful on high-conviction opens |
| 30 min (default) | 2 bars | Institutional standard initial balance |
| 45 min | 3 bars | Extended balance area — more conservative |
| 60 min | 4 bars | Full first hour; common institutional reference |

### RSI Thresholds

| Profile | OB / OS | Behavior |
|---------|---------|----------|
| Tight | 55 / 45 | Very narrow neutral zone; signals only when RSI is mid-range |
| Default | 60 / 40 | Catches intraday exhaustion; wide enough for trending moves |
| Wide | 70 / 30 | Rarely triggers OB/OS on 15-min; effectively disables RSI gating |

### TTM Squeeze Settings

Default BB(20, 2.0) vs KC(20, 1.5) works well across timeframes. The squeeze recency window on 15-min is 2 bars (~30 min), reflecting that a squeeze resolution on this timeframe represents significant multi-hour compression.

### NYSE TICK Settings

The smoothed TICK thresholds should be approximately 60% of the raw thresholds used on the 1-min and 5-min variants, as the SMA(5) compression reduces the effective range.

| Profile | Bull / Bear / Extreme |
|---------|----------------------|
| Sensitive | 200 / -200 / 400 |
| Default | 300 / -300 / 600 |
| Conservative | 400 / -400 / 800 |

### Multi-Timeframe Toggle

When disabled, removes 4 `request.security()` calls (5-min RSI, fast EMA, slow EMA, MACD). Conditions #6 and #7 will never score, effectively reducing max score to 9. VWAP cross detection is also disabled.

### Directional Bias Background

Off by default. When enabled, provides a very subtle (96% transparent) background tint on every bar where the regime is BULLISH or BEARISH. Signal background flashes override the bias background when they fire.

Useful when running the 15-min chart as a "set and glance" bias monitor while focusing attention on the 1-min or 5-min chart.

### Tuning Profiles

#### Profile: Conservative / Trend-Following

For traders who want only the highest-conviction structural signals. Expect 0-3 signals per session.

```
EMA Lengths:           13 / 26 / 50
Anchor EMA:            50
ADX No Trend:          24
Cooldown:              3 bars (45 min)
Signal Start:          1015
Signal End:            1530
RSI OB/OS:             55 / 45
MTF Confirmation:      ON
Squeeze Detection:     ON
TICK Index:            ON
Bias Background:       ON
Min Score:             7
```

#### Profile: Balanced (Default)

For traders targeting 2-6 signals per session with solid conviction.

```
EMA Lengths:           10 / 21 / 34
Anchor EMA:            50
ADX No Trend:          20
Cooldown:              2 bars (30 min)
Signal Start:          1000
Signal End:            1545
RSI OB/OS:             60 / 40
MTF Confirmation:      ON
Squeeze Detection:     ON
TICK Index:            ON
Bias Background:       OFF
Min Score:             5
```

#### Profile: Active Swing Trader

For experienced traders who use the 15-min for swing entries and manage risk with the 5-min/1-min charts.

```
EMA Lengths:           8 / 15 / 26
Anchor EMA:            34
ADX No Trend:          16
Cooldown:              1 bar (15 min)
Signal Start:          0945
Signal End:            1555
RSI OB/OS:             65 / 35
MTF Confirmation:      ON
Squeeze Detection:     OFF
TICK Index:            OFF
Bias Background:       OFF
Min Score:             4
```

---

## Architecture Notes

**Performance**: the script uses `var` declarations for persistent state (lines, labels, level values, cooldown counters) to avoid reallocation on every bar. All dashboard updates occur only on `barstate.islast`. The anchor EMA is a single `ta.ema()` call with negligible overhead. TTM Squeeze calculations (BB, KC, linear regression) use native `ta.*` functions.

**`request.security()` budget**: the script makes 12 `request.security()` calls total:

- 4 for MTF (5-min RSI, 5-min EMA fast, 5-min EMA slow, 5-min MACD histogram)
- 1 for 5-min VWAP cross detection
- 3 for prior day data (high, low, close on daily timeframe)
- 1 for daily open
- 2 for NYSE TICK (1-min raw close, 1-min SMA(5) smoothed)

This is well within TradingView's limit of 40 calls per script. Disabling MTF removes 5 calls; disabling TICK removes 2.

**Repainting**: all signal logic is gated by `barstate.isconfirmed`. Signals appear after bar close only. MTF data from `request.security()` uses `lookahead=barmerge.lookahead_off` (no forward-looking data) for 5-minute and 1-minute pulls, ensuring no repainting from lower-timeframe data.

**Object limits**: Pine Script v6 allows 500 labels, 500 lines, 200 boxes. On a 15-minute chart, a full RTH session is ~26 bars. Even with maximum signal frequency, object limits are never approached. Line extend distances are set to 40 bars (approximately 1.5 sessions) for appropriate visual proportions.

**Opening range bar math**: `orBarsNeeded = math.ceil(i_orMinutes / 15)` converts the minute-based input to 15-minute bars. For the default 30 minutes, this yields 2 bars. Non-multiples of 15 round up (e.g., 20 minutes = 2 bars = 30 minutes effective).

**VWAP bands**: cumulative variance resets with the session. On the first bar, bands overlap VWAP. This resolves within 1-2 bars on the 15-minute chart.

**Timezone**: session detection uses the configurable `i_tz` input, defaulting to `America/New_York`.

**Squeeze momentum**: the linear regression histogram (`ta.linreg`) is calculated but not plotted on the chart (overlay indicator). The momentum value and direction are included in squeeze fired alerts and are available for future enhancements.

**TICK smoothing approach**: rather than smoothing the TICK on the 15-min timeframe (which would only have access to the last 1-min close within each 15-min bar), the SMA(5) is calculated on the 1-min timeframe inside the `request.security()` call. This ensures the smoothing captures the actual 5-minute breadth average, not just the last reading before the 15-min bar closes.

**MACD tuple destructuring**: Pine Script's `ta.macd()` returns a 3-element tuple `[macdLine, signalLine, histogram]`. The `[2]` index extracts the histogram. This is computed inside the `request.security()` call on the 5-min timeframe to avoid needing separate pulls for each MACD component.

---

## Known Limitations

1. **No cumulative delta / order flow**: Pine Script does not provide tick-level bid/ask data. Relative volume and NYSE TICK are the closest available proxies.

2. **No volume profile**: Pine Script cannot natively compute VP (POC, VAH, VAL). Use TradingView's built-in VP tool or a separate indicator as a complement.

3. **Pre-market levels require extended hours**: if your chart does not have Extended Trading Hours enabled, PM High/Low will not populate.

4. **Single-symbol calibration**: defaults are calibrated for SPY's price range and volatility. Other instruments require parameter adjustment.

5. **Equal-weighted scoring**: all 11 conditions contribute equally to the score. Some conditions (like VWAP position or anchor EMA alignment) may be more predictive than others. Weighted scoring is a potential future enhancement.

6. **No backtest capability**: this is an `indicator()`, not a `strategy()`. It cannot be backtested through TradingView's strategy tester.

7. **MTF request.security() limitations**: the 5-minute data pulled by `request.security()` represents the close of the last 5-minute bar, not a real-time tick. Within a forming 15-minute bar, the 5-min data updates every 5 minutes (when a new 5-min bar closes), not continuously. This is a TradingView platform limitation.

8. **Opening range granularity**: the OR input is in minutes but the effective range is always rounded up to the nearest 15-minute bar. A 20-minute OR input results in a 30-minute (2-bar) effective OR.

9. **TICK data availability**: the NYSE TICK symbol (`USI:TICK`) requires your TradingView data plan to include the relevant exchange. If unavailable, the TICK row shows "N/A" and the TICK scoring condition does not contribute to the score. TICK data is only available during RTH (9:30 AM - 4:00 PM ET).

10. **Squeeze recency window is hardcoded**: the 2-bar recency window for squeeze scoring (condition #9) is defined in the source code, not as a configurable input. Modifying the window requires editing the Pine Script source (`sqzBarsSinceFire <= 2`).

11. **TICK smoothing period is hardcoded**: the SMA(5) smoothing period for TICK is defined in the source code. Adjusting the smoothing window requires editing the `request.security()` call.

12. **Sparse signal environment**: with only ~26 RTH bars per session, signals on the 15-min chart are inherently rare. Some sessions may produce zero signals, particularly with higher min score settings. This is by design — the 15-min chart is for directional bias, not rapid entry generation.

13. **Morning star / evening star sensitivity**: the indecision bar threshold (body < 30% of range) may need adjustment depending on market conditions. Very volatile sessions produce fewer clean doji/indecision bars.

---

## Changelog

### v1.0 (2025-02-24)

- Initial release of 15-minute variant.
- Recalibrated EMA (10/21/34), RSI (9-period, 60/40 thresholds), ADX (20 no-trend), and signal parameters for 15-minute bar dynamics.
- **Anchor EMA** (50-period): new structural trend reference spanning ~2 sessions. Plotted as purple line. Integrated as enhancement scoring condition #11.
- **Multi-timeframe confirmation layer (5-minute)**: 5-min RSI, 5-min EMA trend, 5-min MACD histogram, and 5-min VWAP cross via `request.security()`.
- **5-min MACD histogram**: new MTF momentum indicator. MACD(12,26,9) on 5-min provides effective ~1hr/~2.2hr lookback. Scored as enhancement condition #7.
- **SMA-smoothed NYSE TICK**: both raw and SMA(5)-smoothed TICK pulled from 1-min timeframe. Smoothed value used for scoring with calibrated lower thresholds (300/-300/600).
- **5+6 scoring model** with configurable minimum score threshold (default 5). Maximum score: 11. Six enhancement conditions: MTF trend, MTF MACD, level proximity, squeeze recency, TICK breadth, anchor EMA alignment.
- **Expanded candle pattern detection**: added morning star / evening star (3-bar reversal patterns); adjusted strong close (60%) and hammer/star (1.6x) thresholds for 15-min bars.
- **Opening range**: 30-minute default (2 bars), institutional standard initial balance.
- **Directional bias background**: optional subtle background tint when regime is directional (off by default).
- **Session progress indicator**: dashboard row showing bar count out of ~26 total RTH bars with EARLY/LATE SESSION labels.
- **21-field dashboard** with anchor EMA, 5m MACD, and session progress rows. TICK displays both raw and smoothed values.
- Adjusted line extend distances (40 bars) and label offsets for 15-minute chart proportions.
- Signal arrows use `size.small` (vs `size.tiny` on shorter TFs) for visibility on sparser chart.
- Signal tooltips include score, anchor EMA status, MTF trend, MACD, squeeze, and TICK context.
- Dynamic alerts include score (/11), anchor EMA, MTF, MACD, squeeze, and TICK status.
- Three tuning profiles: Conservative, Balanced, Active Swing Trader.
