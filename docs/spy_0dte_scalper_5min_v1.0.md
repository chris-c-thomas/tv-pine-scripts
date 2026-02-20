# 0DTE SPY Scalping Indicator (5-Minute) v1.0

**Platform**: TradingView
**Language**: Pine Script v6
**Chart**: SPY 5-Minute (optimized)
**Theme**: Dark theme (vibrant, high-contrast palette)

---

## Overview

A single-overlay indicator purpose-built for scalping 0DTE (zero days to expiration) SPY options on the 5-minute chart. It combines trend structure, momentum, volatility, key price levels, and a multi-timeframe confirmation layer into a unified decision-support system with real-time signal generation and a heads-up dashboard.

This is the 5-minute companion to the 1-minute SPY 0DTE Scalper. It targets swing-scalp trades with 5-25 minute hold times. The 5-minute chart produces ~78 RTH bars per session, so the signal engine is calibrated for higher conviction per signal with lower total signal count compared to the 1-minute variant.

The indicator does not auto-trade. It surfaces high-confluence setups as CALLS or PUTS labels on the chart, backed by a scoring engine that evaluates up to seven independent conditions per bar. All thresholds are configurable.

---

## Table of Contents

- [Features](#features)
- [Key Differences from 1-Minute Variant](#key-differences-from-1-minute-variant)
- [Installation](#installation)
- [Indicator Components](#indicator-components)
  - [EMA Ribbon](#ema-ribbon)
  - [VWAP and Bands](#vwap-and-bands)
  - [RSI](#rsi)
  - [ADX / DMI](#adx--dmi)
  - [ATR](#atr)
  - [Multi-Timeframe Confirmation](#multi-timeframe-confirmation)
  - [Key Price Levels](#key-price-levels)
  - [Regime / Mode Detection](#regime--mode-detection)
  - [Signal Engine](#signal-engine)
  - [Dashboard](#dashboard)
  - [Alerts](#alerts)
- [Visual Reference](#visual-reference)
  - [Color Map](#color-map)
  - [Chart Element Legend](#chart-element-legend)
  - [Dashboard Fields](#dashboard-fields)
- [Tuning Guide](#tuning-guide)
  - [Minimum Signal Score](#minimum-signal-score)
  - [EMA Lengths](#ema-lengths)
  - [ADX Thresholds](#adx-thresholds)
  - [VWAP Extended Distance](#vwap-extended-distance)
  - [Signal Cooldown](#signal-cooldown)
  - [Signal Time Window](#signal-time-window)
  - [Opening Range Duration](#opening-range-duration)
  - [RSI Thresholds](#rsi-thresholds)
  - [Multi-Timeframe Toggle](#multi-timeframe-toggle)
  - [Tuning Profiles](#tuning-profiles)
- [Architecture Notes](#architecture-notes)
- [Known Limitations](#known-limitations)
- [Changelog](#changelog)

---

## Features

- **EMA Ribbon** (9/15/21) with dynamic cloud fill that shifts color based on trend alignment (bullish, bearish, mixed).
- **Session-anchored VWAP** with configurable standard deviation bands.
- **RSI, ADX, and ATR** calculated internally and displayed in the dashboard (not plotted as separate panes, preserving chart real estate).
- **Multi-timeframe confirmation layer** pulling 1-minute RSI and EMA trend alignment via `request.security()` for lower-timeframe context on the 5-minute chart.
- **Pre-Market High/Low** automatically detected and drawn as horizontal levels once RTH begins.
- **Prior Day High/Low/Close** pulled from the daily timeframe and plotted as dotted reference levels.
- **Opening Range** (default 15 minutes / 3 bars) captured at RTH open and drawn as dashed levels.
- **Session HOD/LOD** tracked in real time with dynamically updating lines.
- **Regime Classifier** that categorizes the current market state into one of five modes: BULLISH, BEARISH, RANGING, NO TRADE, or TRANSITION.
- **5+2 confluence scoring engine** with 5 core conditions and 2 optional enhancement conditions (MTF trend + level proximity). Generates CALLS or PUTS labels when the score meets the configurable threshold.
- **Expanded candle pattern detection** optimized for 5-minute bars: engulfing, strong close, hammer/shooting star, inside bar breakout, and 3-bar momentum.
- **VWAP cross markers** (diamond shapes) for quick visual identification of VWAP reclaims and rejections.
- **Real-time dashboard** (table overlay) showing 16 fields of live indicator data including MTF rows, with color-coded status text.
- **Dual alert system**: static `alertcondition()` entries for TradingView's standard alert UI, plus dynamic `alert()` calls with interpolated context including score and MTF status.
- **Bar confirmation gate** to prevent signals from firing on incomplete (still-forming) bars.
- **Signal cooldown** (default 3 bars = 15 minutes) to prevent label spam.

---

## Key Differences from 1-Minute Variant

| Aspect | 1-Minute | 5-Minute | Why |
|--------|----------|----------|-----|
| EMA Lengths | 8/13/21 | 9/15/21 | 5-min bars need slightly wider windows; 9/15/21 is near-Fibonacci and captures 45-105 min swings |
| RSI Length | 14 | 10 | 5-min bars are smoother; RSI(14) is too sluggish. RSI(10) = 50 min lookback |
| RSI OB/OS | 70/30 | 65/35 | 5-min RSI rarely hits extremes intraday; tighter thresholds catch exhaustion earlier |
| ADX No-Trend | 15 | 18 | ADX reads higher on smoother bars; 18 provides a meaningful no-trend gate |
| Opening Range | 5 min (5 bars) | 15 min (3 bars) | 1 bar is useless as a range; 15-min is the institutional standard |
| Signal Cooldown | 5 bars (5 min) | 3 bars (15 min) | Balances spam prevention with ~78 bars/session |
| Signal Window Start | 09:35 | 09:45 | First 3 bars establish OR; signals start after OR completes |
| VWAP Ext Filter | 1.5 ATR | 2.0 ATR | 5-min ATR is larger absolute; wider multiplier avoids premature extension flags |
| Candle Patterns | 4 patterns | 6 patterns | Added inside bar breakout and 3-bar momentum, optimized thresholds |
| Strong Close Threshold | 50% of range | 55% of range | 5-min candles have proportionally larger wicks |
| Hammer/Star Threshold | 2.0x body | 1.8x body | 5-min patterns are slightly less extreme |
| Scoring Model | Flat (all must pass) | 5+2 with score threshold | Flexible scoring allows partial confluence; configurable min score |
| MTF Confirmation | N/A | 1-min RSI + EMA trend | Lower-TF context adds precision without chart clutter |
| Dashboard Rows | 14 | 16 | Added 1m Trend and 1m RSI rows |
| Level Proximity Scoring | Not in signal logic | Explicit condition (#7) | Near-support/resistance scored as enhancement condition |
| Line Extend Distance | 300 bars left | 60 bars left | 5-min sessions are ~78 bars; 300 would extend off-chart |

---

## Installation

1. Open TradingView and navigate to a **SPY 5-minute chart**.
2. Open the **Pine Editor** (bottom panel).
3. Delete any existing code in the editor.
4. Paste the entire contents of `spy_0dte_scalper_5min.pine` into the editor.
5. Click **"Add to chart"** (or "Save" then "Add to chart").
6. The indicator appears as an overlay with the dashboard in the top-right corner.

**Required chart settings**:

- **Extended hours**: Enable "Extended Trading Hours" in chart settings if you want Pre-Market High/Low levels to populate. Without extended hours data, those fields will show "N/A" in the dashboard and no PM level lines will render.
- **Timezone**: The script defaults to `America/New_York` and is configurable. Your TradingView chart timezone setting does not affect the indicator's session detection.

**Companion usage**: this indicator can be used alongside the 1-minute variant. Run the 1-minute version on a 1-minute SPY chart in a separate tab/layout, and this 5-minute version on a 5-minute SPY chart. The 5-minute version already pulls 1-minute data internally via MTF, so running both is optional but provides visual context on the lower timeframe.

---

## Indicator Components

### EMA Ribbon

Three exponential moving averages that form a visual ribbon around price action.

| Parameter | Default | Description |
|-----------|---------|-------------|
| Fast EMA Length | 9 | 9 bars = 45 minutes. Fastest-reacting EMA. |
| Mid EMA Length | 15 | 15 bars = 75 minutes. Intermediate smoothing. |
| Slow EMA Length | 21 | 21 bars = 105 minutes. Trend anchor. |
| Show EMA Cloud Fill | true | Toggles the shaded fill between fast and slow EMAs. |

**Cloud behavior**:

- **Green fill**: all three EMAs are stacked bullish (fast > mid > slow).
- **Red fill**: all three EMAs are stacked bearish (fast < mid < slow).
- **Yellow fill**: mixed/transitional alignment.

On the 5-minute chart, EMA cloud flips represent ~45-105 minute trend shifts. A green cloud with price riding above the ribbon indicates a strong intraday uptrend structure that typically supports 15-30 minute swing-scalp holds.

### VWAP and Bands

The Volume Weighted Average Price resets each trading session and serves as the primary institutional reference level.

| Parameter | Default | Description |
|-----------|---------|-------------|
| Show VWAP | true | Toggle VWAP line visibility |
| Show VWAP Bands | true | Toggle standard deviation bands |
| VWAP Band Multiplier | 1.0 | Number of standard deviations for upper/lower bands |

**Band calculation**: uses cumulative volume-weighted variance from session open. The bands represent one standard deviation of price dispersion around VWAP. On a 5-minute chart, band width builds more gradually than on 1-minute since fewer bars contribute to the cumulative variance in the early session.

### RSI

Relative Strength Index. Calculated internally with a 10-period default (50 minutes on 5-min chart) and displayed in the dashboard only.

| Parameter | Default | Description |
|-----------|---------|-------------|
| RSI Length | 10 | Lookback period (10 bars = 50 minutes) |
| Overbought | 65 | Upper threshold (tighter than 1-min's 70) |
| Oversold | 35 | Lower threshold (tighter than 1-min's 30) |

**Why tighter thresholds**: on a 5-minute chart, RSI is inherently smoother because each bar aggregates more price action. RSI(10) on 5-min rarely reaches 70/30 intraday on SPY. Using 65/35 catches momentum exhaustion at levels that are actually meaningful on this timeframe. The signal engine requires RSI to be within the neutral zone for both CALLS and PUTS to fire.

### ADX / DMI

The Average Directional Index measures trend strength regardless of direction.

| Parameter | Default | Description |
|-----------|---------|-------------|
| ADX Length | 14 | Lookback for ADX/DI calculation (14 bars = 70 minutes) |
| No Trend Threshold | 18 | Below this = NO TRADE / RANGING regime |
| Trending Threshold | 25 | Above this = confirmed trend |
| Strong Trend Threshold | 40 | Above this = strong directional move |

**ADX status mapping**:

| ADX Value | Status | Color | Implication |
|-----------|--------|-------|-------------|
| < 18 | NO TREND | Red | Chop. Avoid trading. Theta decay dominates. |
| 18 - 24.9 | WEAK/RANGING | Yellow | Directional bias forming but not confirmed. |
| 25 - 39.9 | TRENDING | Green | Confirmed trend. Signals are higher conviction. |
| >= 40 | STRONG TREND | Green | Strong directional move. Trend-following entries preferred. |

The raised no-trend threshold (18 vs. 15 on 1-min) reflects that ADX on 5-minute bars reads higher during mild ranging conditions. An ADX of 15 on 5-min often represents genuine chop that would destroy 0DTE positions.

### ATR

Average True Range measures realized volatility on the 5-minute timeframe.

| Parameter | Default | Description |
|-----------|---------|-------------|
| ATR Length | 14 | Lookback period (14 bars = 70 minutes) |

ATR on 5-minute bars is approximately 2.5-4x larger than 1-minute ATR in absolute terms. This is expected (more price movement in 5 minutes). The ATR value is used for VWAP extension filtering, level proximity detection, and is displayed in the dashboard for option strike selection context.

### Multi-Timeframe Confirmation

The 5-minute variant's primary enhancement over the 1-minute version. Pulls 1-minute data via `request.security()` to provide lower-timeframe context without requiring a separate chart.

| Parameter | Default | Description |
|-----------|---------|-------------|
| Enable 1-Min MTF Confirmation | true | Master toggle for all MTF features |
| 1-Min RSI Length | 14 | RSI lookback on the 1-minute timeframe |
| 1-Min Fast EMA Length | 8 | Fast EMA on 1-minute (matches 1-min variant default) |
| 1-Min Slow EMA Length | 21 | Slow EMA on 1-minute (matches 1-min variant default) |

**What gets pulled from the 1-minute chart**:

| Data | Usage |
|------|-------|
| 1-Min RSI | Dashboard display + awareness. When 5-min RSI looks neutral but 1-min RSI is already OB/OS, momentum may be exhausted on the lower timeframe. |
| 1-Min EMA Trend (fast > slow?) | Signal scoring condition #6. When 1-min trend aligns with 5-min direction, the signal scores an extra point. Disagreement between timeframes suggests lower conviction. |
| 1-Min VWAP Cross | Detects if a VWAP cross occurred on the 1-min within the current 5-min bar. Informational for dashboard awareness. |

**Signal integration**: when MTF is enabled, the signal engine adds a 6th scoring condition: "1-min EMA trend confirms the 5-min signal direction." This is additive — it improves signal quality when present but does not gate signals when absent (unless `i_minScore` is set high enough to require it).

**Dashboard integration**: two dedicated rows (1m Trend, 1m RSI) appear at the bottom of the dashboard. When MTF is disabled, these rows show "OFF" in gray.

**Performance note**: each `request.security()` call adds a small computation overhead. Three calls to the 1-minute timeframe are within TradingView's limits and do not materially affect indicator loading time.

### Key Price Levels

Horizontal reference levels drawn on the chart. Each has a dashed or dotted line style and a right-edge label.

#### Pre-Market High / Low

- **Detection**: tracks the highest high and lowest low during the pre-market session (4:00 AM - 9:29 AM ET).
- **Rendering**: dashed horizontal lines drawn once RTH begins, extending 60 bars left (~5 hours on 5-min chart) and 5 bars right.
- **Requirement**: Extended Trading Hours must be enabled in chart settings.

#### Prior Day High / Low / Close

- **Source**: daily timeframe via `request.security()` using the prior completed day.
- **Rendering**: dotted horizontal lines with right-edge labels (PDH, PDL, PDC).
- **No repainting risk**: the `[1]` offset with `lookahead_on` accesses only the prior completed daily bar.

#### Opening Range

- **Detection**: captures the high and low of the first N minutes of RTH. Default is 15 minutes (3 bars on 5-min chart).
- **Bar count**: `orBarsNeeded = math.ceil(i_orMinutes / 5)`. The input is in minutes and automatically converts to the appropriate number of 5-minute bars.
- **Rendering**: dashed horizontal lines with right-edge labels (OR HIGH, OR LOW) drawn once the opening range period completes.
- **Usage**: the 15-minute opening range is an institutional standard. Breakout above OR HIGH with volume on the 5-min chart is a high-probability continuation signal. Failure to break the OR in the first hour often leads to ranging conditions.

#### Session HOD / LOD

- **Detection**: running high-of-day and low-of-day from RTH open, updated on every bar.
- **Rendering**: solid horizontal lines with right-edge labels that dynamically update.
- **Reset**: clears at each RTH open.

### Regime / Mode Detection

A composite classifier combining ADX trend strength, EMA ribbon alignment, and price position relative to VWAP.

| Mode | Conditions | Implication |
|------|-----------|-------------|
| **BULLISH** | ADX >= 18, EMAs stacked bullish, price above VWAP | Favor CALLS entries |
| **BEARISH** | ADX >= 18, EMAs stacked bearish, price below VWAP | Favor PUTS entries |
| **RANGING** | ADX 18-25 or mixed EMA/VWAP | Caution; both directions possible |
| **NO TRADE** | ADX < 18 and price near VWAP | Avoid. Low-conviction chop. |
| **TRANSITION** | ADX >= 18 but EMA/VWAP not aligned | Emerging trend; use caution |

On the 5-minute chart, mode transitions are more significant than on 1-minute. A mode flip from RANGING to BULLISH on 5-min represents a meaningful ~1-hour structural shift. Mode changes trigger alerts.

### Signal Engine

The core decision engine that generates CALLS and PUTS labels on the chart.

#### Scoring System (5+2 Model)

Each signal direction is evaluated using 5 core conditions and 2 optional enhancement conditions. Each true condition adds 1 point to the score.

**CALLS conditions (bullish)**:

| # | Type | Condition | Rationale |
|---|------|-----------|-----------|
| 1 | Core | Price above VWAP or crossing above | Institutional bias is bullish |
| 2 | Core | EMA ribbon bullish or price reclaiming slow EMA | Trend structure supports upside |
| 3 | Core | RSI between 35-65 (neutral zone) | Momentum has room to run |
| 4 | Core | Bullish candle pattern detected | Price action confirmation |
| 5 | Core | ADX above no-trend threshold (18) | Measurable directional movement |
| 6 | Enhancement | 1-min EMA trend is bullish (MTF) | Lower-TF trend confirms direction |
| 7 | Enhancement | Price near support level (within 1.0 ATR) | Favorable risk/reward at structure |

**Candle patterns that satisfy condition #4**:

- Bullish engulfing (close > prior high, open <= prior low)
- Strong bull close (body > 55% of range)
- Hammer (lower wick > 1.8x body)
- Inside bar breakout to the upside (prior bar inside, current breaks above)
- 3-bar bullish momentum (3 consecutive higher closes)

**PUTS conditions**: mirror of CALLS with inverted direction. Uses bearish engulfing, strong bear close, shooting star, inside bar breakdown, and 3-bar bearish momentum.

#### Minimum Score

| `i_minScore` | Typical Signals/Session | Behavior |
|---|---|---|
| 3 | 10-20+ | Very aggressive. Most core conditions don't need to align. High false positive rate. |
| 4 (default) | 5-12 | Balanced. Requires 4 of 5 core conditions, or 3 core + 1-2 enhancements. |
| 5 | 2-6 | Conservative. Requires all 5 core, or 4 core + 1 enhancement. |
| 6 | 0-3 | Strict. Requires 5 core + 1 enhancement. Very few signals. |
| 7 | 0-1 | Sniper. All 7 conditions must be true. May produce zero signals on some sessions. |

#### Signal Filters

Filters must pass before scoring is evaluated:

| Filter | Description |
|--------|-------------|
| **Time Window** | Default 09:45-15:45 ET. Starts after OR completes. Ends 15 min before close. |
| **Cooldown** | 3 bars (15 min) minimum between same-direction signals. |
| **Bar Confirmation** | Signals fire only on confirmed (closed) bars. No phantom signals. |
| **ADX Gate** | When enabled, ADX must be above no-trend threshold. |
| **VWAP Extension** | Signals suppressed when price is > 2.0 ATR from VWAP (overextended). |

#### Visual Output

When a signal fires:

1. **Label**: "CALLS" (green, below bar) or "PUTS" (red, above bar) with a hover tooltip showing score, RSI, ADX, VWAP distance, mode, ATR, EMA status, and MTF data.
2. **Arrow**: small triangle shape as a secondary quick-scan marker.
3. **Background flash**: faint green or red background highlight on the signal bar.

#### VWAP Cross Markers

Diamond shapes on VWAP crosses during RTH:

- **Lime diamond below bar**: bullish VWAP cross (price crossed above).
- **Red diamond above bar**: bearish VWAP cross (price crossed below).

On a 5-minute chart, VWAP crosses are less frequent and more significant than on 1-minute. Each cross represents a 5-minute candle definitively closing above or below the institutional fair value level.

### Dashboard

A real-time information table overlaid in the top-right corner.

#### Layout

Three-column table with a header row and 15 data rows (16 total). Dark navy background with magenta header bar. Color-coded values for instant status reads.

#### Fields

| Row | Field | Value | Color Logic |
|-----|-------|-------|-------------|
| Header | SPY 0DTE Scalper (5m) Dashboard | — | Magenta accent bar |
| 1 | Price | Current close + bar direction | Green = green bar, Red = red bar |
| 2 | Mode | Regime classification + advice | Green/Red/Yellow/Gray per mode |
| 3 | RSI (10) | RSI value + status | Red = OB, Green = OS, White = neutral |
| 4 | ADX | ADX value + trend status | Red/Yellow/Green per strength |
| 5 | VWAP Dist | Signed distance + status | Green = near, Yellow = moderate, Red = extended |
| 6 | PM High | Distance to PM High + status | Red = resistance, Green = cleared |
| 7 | PM Low | Distance to PM Low + status | Green = support, Red = broken |
| 8 | ATR (14) | Current ATR value | Cyan (informational) |
| 9 | EMA Trend | 5-min ribbon alignment | Green = bullish, Red = bearish, Yellow = mixed |
| 10 | Signal | Current bar signal + score | Green = CALLS, Red = PUTS, Gray = none |
| 11 | Day Chg | % change from session open | Green = positive, Red = negative |
| 12 | Rel Vol | Volume / 20-bar avg | Green + "SPIKE" if > 1.5x |
| 13 | DI Spread | +DI / -DI values | Green = bulls, Red = bears |
| 14 | 1m Trend | 1-min EMA alignment (MTF) | Green/Red/Yellow; "OFF" if MTF disabled |
| 15 | 1m RSI | 1-min RSI value + status (MTF) | Same color logic as 5-min RSI row |

### Alerts

#### Static Alerts (alertcondition)

| Alert Name | Trigger |
|------------|---------|
| 0DTE-5m: CALLS Signal | CALLS signal fires |
| 0DTE-5m: PUTS Signal | PUTS signal fires |
| 0DTE-5m: VWAP Cross Bullish | Price crosses above VWAP |
| 0DTE-5m: VWAP Cross Bearish | Price crosses below VWAP |
| 0DTE-5m: Regime Mode Change | Mode transitions to a different state |

#### Dynamic Alerts (alert)

Rich, interpolated messages including score and MTF status. Example:

```
0DTE-5m CALLS | SPY 586.42 | Score 5/7 | RSI 52.3 | ADX 27.8 | VWAP Dist 0.45 | Mode: BULLISH | 1m: BULLISH
```

```
0DTE-5m MODE SHIFT | RANGING -> BULLISH | SPY 587.10
```

---

## Visual Reference

### Color Map

Identical to the 1-minute variant — all colors are selected for maximum contrast against TradingView's dark theme.

| Element | Hex | Notes |
|---------|-----|-------|
| VWAP | `#FFFFFF` | Solid white, weight 2 |
| VWAP Bands | `#00BCD4` | Cyan, circle style |
| EMA 9 (Fast) | `#FFD600` | Bright yellow |
| EMA 15 (Mid) | `#FF9100` | Bright orange |
| EMA 21 (Slow) | `#FF1744` | Hot pink/red |
| EMA Cloud (Bull) | `#00E676` | Neon green, 85% transparent |
| EMA Cloud (Bear) | `#FF1744` | Neon red, 85% transparent |
| EMA Cloud (Mixed) | `#FFD600` | Yellow, 90% transparent |
| PM High | `#FF5252` | Bright red, dashed |
| PM Low | `#69F0AE` | Bright green, dashed |
| Prior Day High | `#FF6E40` | Coral, dotted |
| Prior Day Low | `#64FFDA` | Teal, dotted |
| Prior Day Close | `#B0BEC5` | Gray, dotted |
| OR High | `#E040FB` | Magenta, dashed |
| OR Low | `#B388FF` | Purple, dashed |
| CALLS Signal | `#00E676` | Neon green label + arrow + bg |
| PUTS Signal | `#FF1744` | Neon red label + arrow + bg |
| VWAP Cross (Bull) | `#76FF03` | Lime diamond |
| VWAP Cross (Bear) | `#FF1744` | Red diamond |

### Chart Element Legend

```
EMA 9 (Fast)  |  EMA 15 (Mid)  |  EMA 21 (Slow)  |  VWAP  |  VWAP +1σ  |  VWAP -1σ
CALLS Arrow  |  PUTS Arrow  |  VWAP Cross Bull  |  VWAP Cross Bear  |  Signal Background Flash
```

Key levels (PM, PD, OR, HOD/LOD) are drawn with `line.new()` and identified by right-edge text labels.

### Dashboard Fields

See [Dashboard](#dashboard) for the full 16-field table. The dashboard adapts to MTF on/off state — when MTF is disabled, the bottom two rows show "OFF" in gray rather than disappearing, so the table layout remains stable.

---

## Tuning Guide

All parameters are exposed as configurable inputs. This section covers tuning for the 5-minute timeframe specifically.

### Minimum Signal Score

**Input**: Minimum Signal Score
**Default**: 4
**Range**: 2-7

This is the most impactful setting. It controls how many of the 7 possible conditions must be true for a signal to fire.

| Score | Signals/Session | Best For |
|-------|-----------------|----------|
| 3 | 10-20+ | Aggressive scalpers. High noise. |
| 4 (default) | 5-12 | Balanced. ~4 core conditions must align. |
| 5 | 2-6 | Conservative. Requires near-full core alignment or core + enhancements. |
| 6 | 0-3 | Very selective. Near-perfect confluence required. |
| 7 | 0-1 | Sniper. All conditions including MTF and level proximity. |

**Recommendation**: start at 4. If too many signals are noise, increase to 5. If you're missing moves, ensure MTF is enabled (adds scoring opportunities) before dropping to 3.

### EMA Lengths

**Defaults**: 9, 15, 21

| Adjustment | Effect |
|------------|--------|
| Shorter (5/9/15) | More responsive. Cloud flips represent ~25-75 min shifts. Better for aggressive scalps. |
| Default (9/15/21) | Balanced. Cloud represents ~45-105 min trend windows. |
| Longer (13/21/34) | Smoother. Cloud flips are rare but high-conviction. Better for swing-scalp holds of 15-30 min. |

### ADX Thresholds

**Default No-Trend**: 18

| Threshold | Behavior |
|-----------|----------|
| 14-16 | Permissive. Allows signals during mild chop. Riskier for 0DTE. |
| 18 (default) | Balanced. Filters out most low-conviction environments. |
| 20-22 | Strict. Only allows signals during clear trends. Reduces midday signal count significantly. |

### VWAP Extended Distance

**Input**: Max VWAP Distance (ATR multiples)
**Default**: 2.0

Signals are suppressed when price is more than this many ATR units from VWAP (overextended).

| Value | Behavior |
|-------|----------|
| 1.5 | Conservative. Suppresses earlier. Prevents chasing. |
| 2.0 (default) | Balanced for 5-min ATR magnitudes. |
| 3.0 | Permissive. Allows signals during strong trend runs where VWAP divergence is sustained. |

### Signal Cooldown

**Input**: Signal Cooldown (bars)
**Default**: 3 (= 15 minutes)

| Value | Behavior |
|-------|----------|
| 1-2 (5-10 min) | Signals can fire frequently during sustained moves. Risk of label clusters. |
| 3 (default, 15 min) | One signal per 15-minute window per direction. Balanced for ~78 bars/session. |
| 4-6 (20-30 min) | Very selective. Each signal is well-separated. Good for single-position traders. |

### Signal Time Window

**Default**: 09:45 - 15:45 ET

| Adjustment | Rationale |
|------------|-----------|
| Start 09:35 | Include the OR formation period. High reward but high noise. Only for experienced traders. |
| Start 09:45 (default) | Starts after the 15-min opening range completes. Recommended. |
| Start 10:00 | Wait for the first 30 minutes to establish structure. Very conservative. |
| End 15:45 (default) | Stops 15 min before close. Avoids gamma/pin risk. |
| End 15:30 | Stops 30 min early. Extra-conservative for 0DTE. |
| End 15:55 | Signals nearly to close. Only if you understand pin risk and rapid gamma decay. |

### Opening Range Duration

**Input**: Opening Range Minutes
**Default**: 15

On the 5-minute chart, the input is automatically converted to bars (`ceil(minutes / 5)`).

| Value | Bars | Usage |
|-------|------|-------|
| 5 | 1 | Single bar. Too narrow for meaningful range. Not recommended on 5-min. |
| 15 (default) | 3 | 15-minute OR. Institutional standard. Breakouts are high-probability. |
| 30 | 6 | 30-minute range. Very wide. Fewer breakout opportunities but highest conviction. |

### RSI Thresholds

**Defaults**: 65 (OB) / 35 (OS)

| Adjustment | Effect |
|------------|--------|
| Tighter (60/40) | More aggressive filtering. Signals only fire when RSI has significant room. Fewer signals. |
| Default (65/35) | Balanced for 5-min RSI dynamics. |
| Wider (70/30) | Matches 1-min defaults. Allows signals closer to extremes. More signals but some at exhaustion points. |

### Multi-Timeframe Toggle

**Input**: Enable 1-Min MTF Confirmation
**Default**: true

When enabled, adds 1-min RSI and EMA trend to the dashboard and makes condition #6 (MTF trend alignment) available for scoring.

| Setting | Effect |
|---------|--------|
| ON (default) | Full MTF layer. Condition #6 available. Dashboard shows 1m data. Three `request.security()` calls active. |
| OFF | No MTF data. Conditions #1-5 and #7 only. Dashboard shows "OFF" for MTF rows. Slightly faster load. |

**When to disable**: if you're running the 1-minute variant simultaneously on a separate chart and don't want redundant MTF computation. Or if you prefer a simpler signal model with 5+1 conditions only (core + level proximity).

### Tuning Profiles

#### Profile: Conservative / Swing-Scalp

For traders targeting 1-3 high-conviction trades per session with 15-30 minute holds.

```
EMA Lengths:           13 / 21 / 34
ADX No Trend:          22
Cooldown:              4 bars (20 min)
Signal Start:          0945
Signal End:            1530
RSI OB/OS:             60 / 40
MTF Confirmation:      ON
Min Score:             5
```

#### Profile: Balanced (Default)

For traders targeting 5-12 signals per session with moderate conviction.

```
EMA Lengths:           9 / 15 / 21
ADX No Trend:          18
Cooldown:              3 bars (15 min)
Signal Start:          0945
Signal End:            1545
RSI OB/OS:             65 / 35
MTF Confirmation:      ON
Min Score:             4
```

#### Profile: Aggressive / Scalper

For experienced traders who want frequent signals and manage risk through position sizing and rapid exits.

```
EMA Lengths:           5 / 9 / 15
ADX No Trend:          14
Cooldown:              2 bars (10 min)
Signal Start:          0935
Signal End:            1555
RSI OB/OS:             70 / 30
MTF Confirmation:      OFF
Min Score:             3
```

---

## Architecture Notes

**Performance**: the script uses `var` declarations for persistent state (lines, labels, level values, cooldown counters) to avoid reallocation on every bar. `request.security()` calls total 6: three for MTF (1-min RSI, 1-min EMA fast, 1-min EMA slow), plus three for prior day (H/L/C) and daily open. The dashboard table is updated only on `barstate.islast`.

**Repainting**: all signal logic is gated by `barstate.isconfirmed`. Signals appear after bar close only. MTF data from `request.security()` uses `lookahead=barmerge.lookahead_off` (no forward-looking data) for 1-minute pulls, ensuring no repainting from lower-timeframe data.

**Object limits**: Pine Script v6 allows 500 labels, 500 lines, 200 boxes. On a 5-minute chart, a full RTH session is ~78 bars. Even with signals on every bar (impossible given cooldowns), object limits are never approached. Line extend distances are set to 60 bars (appropriate for 5-min session length) rather than 300 (which would extend far off-chart).

**Opening range bar math**: `orBarsNeeded = math.ceil(i_orMinutes / 5)` converts the minute-based input to 5-minute bars. For the default 15 minutes, this yields 3 bars. Non-multiples of 5 round up (e.g., 12 minutes = 3 bars = 15 minutes effective).

**VWAP bands**: cumulative variance resets with the session. On the first bar, bands overlap VWAP. This resolves within 1-2 bars on the 5-minute chart.

**Timezone**: session detection uses the configurable `i_tz` input, defaulting to `America/New_York`.

---

## Known Limitations

1. **No cumulative delta / order flow**: Pine Script does not provide tick-level bid/ask data. Relative volume is the closest available proxy.

2. **No volume profile**: Pine Script cannot natively compute VP (POC, VAH, VAL). Use TradingView's built-in VP tool or a separate indicator as a complement.

3. **Pre-market levels require extended hours**: if your chart does not have Extended Trading Hours enabled, PM High/Low will not populate.

4. **Single-symbol calibration**: defaults are calibrated for SPY's price range and volatility. Other instruments require parameter adjustment.

5. **Equal-weighted scoring**: all 7 conditions contribute equally to the score. Some conditions (like VWAP position) may be more predictive. Weighted scoring is a potential v2 enhancement.

6. **No backtest capability**: this is an `indicator()`, not a `strategy()`. It cannot be backtested through TradingView's strategy tester.

7. **MTF request.security() limitations**: the 1-minute data pulled by `request.security()` represents the close of the last 1-minute bar, not a real-time tick. Within a forming 5-minute bar, the 1-min data updates every minute (when a new 1-min bar closes), not continuously. This is a TradingView platform limitation.

8. **Opening range granularity**: the OR input is in minutes but the effective range is always rounded up to the nearest 5-minute bar. A 7-minute OR input results in a 10-minute (2-bar) effective OR.

---

## Changelog

### v1.0 (2025-02-18)

- Initial release of 5-minute variant.
- Recalibrated EMA (9/15/21), RSI (10-period, 65/35 thresholds), ADX (18 no-trend), and ATR defaults for 5-minute bar dynamics.
- Multi-timeframe confirmation layer: 1-min RSI, 1-min EMA trend, and 1-min VWAP cross via `request.security()`.
- 5+2 scoring model with configurable minimum score threshold (default 4).
- Expanded candle pattern detection: added inside bar breakout and 3-bar momentum patterns; adjusted strong close (55%) and hammer/star (1.8x) thresholds.
- Explicit level proximity scoring (condition #7): near-support for CALLS, near-resistance for PUTS.
- 16-field dashboard with 1m Trend and 1m RSI rows.
- Opening range bar-count conversion (`ceil(minutes / 5)`) for 5-minute granularity.
- Adjusted line extend distances (60 bars) and label offsets for 5-minute chart proportions.
- Signal tooltips include score, MTF trend, and MTF RSI.
- Dynamic alerts include score and MTF status.
- Three tuning profiles: Conservative, Balanced, Aggressive.
