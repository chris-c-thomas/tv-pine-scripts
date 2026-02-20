# 0DTE SPY Scalping Indicator (1-Minute) v1.0

**Platform**: TradingView
**Language**: Pine Script v6
**Chart**: SPY 1-Minute (optimized)
**Theme**: Dark theme (vibrant, high-contrast palette)

---

## Overview

A single-overlay indicator purpose-built for scalping 0DTE (zero days to expiration) SPY options on the 1-minute chart. It combines trend structure, momentum, volatility, and key price levels into a unified decision-support system with real-time signal generation and a heads-up dashboard.

The indicator does not auto-trade. It surfaces high-confluence setups as CALLS or PUTS labels on the chart, backed by a scoring engine that evaluates six independent conditions per bar. All thresholds are configurable.

---

## Table of Contents

- [Features](#features)
- [Installation](#installation)
- [Indicator Components](#indicator-components)
  - [EMA Ribbon](#ema-ribbon)
  - [VWAP and Bands](#vwap-and-bands)
  - [RSI](#rsi)
  - [ADX / DMI](#adx--dmi)
  - [ATR](#atr)
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
  - [Signal Strictness](#signal-strictness)
  - [EMA Lengths](#ema-lengths)
  - [ADX Thresholds](#adx-thresholds)
  - [VWAP Extended Distance](#vwap-extended-distance)
  - [Signal Cooldown](#signal-cooldown)
  - [Signal Time Window](#signal-time-window)
  - [Opening Range Duration](#opening-range-duration)
  - [RSI Thresholds](#rsi-thresholds)
  - [Tuning Profiles](#tuning-profiles)
- [Architecture Notes](#architecture-notes)
- [Known Limitations](#known-limitations)
- [Changelog](#changelog)

---

## Features

- **EMA Ribbon** with dynamic cloud fill that shifts color based on trend alignment (bullish, bearish, mixed).
- **Session-anchored VWAP** with configurable standard deviation bands.
- **RSI, ADX, and ATR** calculated internally and displayed in the dashboard (not plotted as separate panes, preserving chart real estate).
- **Pre-Market High/Low** automatically detected and drawn as horizontal levels once RTH begins.
- **Prior Day High/Low/Close** pulled from the daily timeframe and plotted as dotted reference levels.
- **Opening Range** (configurable duration) captured at RTH open and drawn as dashed levels.
- **Session HOD/LOD** tracked in real time with dynamically updating lines.
- **Regime Classifier** that categorizes the current market state into one of five modes: BULLISH TREND, BEARISH TREND, RANGING, NO TRADE, or TRANSITION.
- **Confluence-based signal engine** that scores six conditions per bar and generates CALLS or PUTS labels when the score meets the configured threshold.
- **VWAP cross markers** (diamond shapes) for quick visual identification of VWAP reclaims and rejections.
- **Real-time dashboard** (table overlay) showing 14 fields of live indicator data with color-coded status text.
- **Dual alert system**: static `alertcondition()` entries for TradingView's standard alert UI, plus dynamic `alert()` calls with interpolated context for webhook/notification pipelines.
- **Bar confirmation gate** to prevent signals from firing on incomplete (still-forming) bars, eliminating phantom signals.
- **Signal cooldown** to prevent label spam during rapid price action.

---

## Installation

1. Open TradingView and navigate to a **SPY 1-minute chart**.
2. Open the **Pine Editor** (bottom panel).
3. Delete any existing code in the editor.
4. Paste the entire contents of `spy_0dte_scalper.pine` into the editor.
5. Click **"Add to chart"** (or "Save" then "Add to chart").
6. The indicator appears as an overlay with the dashboard in the top-right corner.

**Required chart settings**:

- **Extended hours**: Enable "Extended Trading Hours" in chart settings if you want Pre-Market High/Low levels to populate. Without extended hours data, those fields will show "—" in the dashboard and no PM level lines will render.
- **Timezone**: The script is hardcoded to `America/New_York`. Your TradingView chart timezone setting does not affect the indicator's session detection.

---

## Indicator Components

### EMA Ribbon

Three exponential moving averages that form a visual ribbon around price action.

| Parameter | Default | Description |
|-----------|---------|-------------|
| Fast EMA Length | 8 | Fastest-reacting EMA, hugs price tightly |
| Mid EMA Length | 13 | Intermediate smoothing |
| Slow EMA Length | 21 | Trend anchor, slowest to react |
| Show EMA Cloud Fill | true | Toggles the shaded fill between fast and slow EMAs |

**Cloud behavior**:

- **Green fill**: all three EMAs are stacked bullish (fast > mid > slow).
- **Red fill**: all three EMAs are stacked bearish (fast < mid < slow).
- **Yellow fill**: mixed/transitional alignment.

The cloud provides an at-a-glance read on whether the short-term trend structure supports a directional bias. When the cloud is green and price is riding above the ribbon, the path of least resistance is up. Red cloud with price below = bearish structure. Yellow = chop, proceed with caution.

### VWAP and Bands

The Volume Weighted Average Price resets each trading session and serves as the primary institutional reference level for intraday trading.

| Parameter | Default | Description |
|-----------|---------|-------------|
| Show VWAP | true | Toggle VWAP line visibility |
| Show VWAP Bands | true | Toggle standard deviation bands |
| VWAP Band Multiplier | 1.0 | Number of standard deviations for upper/lower bands |
| VWAP Extended Distance | 2.0 | Points from VWAP to classify price as "DISCONNECTED" |

**Band calculation**: uses cumulative volume-weighted variance from session open. The bands represent one standard deviation of price dispersion around VWAP. Price outside the bands suggests overextension and potential mean-reversion risk.

**Dashboard integration**: the VWAP Dist field shows signed distance in points. When absolute distance exceeds the Extended Distance threshold, the status reads "DISCONNECTED" in red, warning against chasing.

### RSI

Relative Strength Index (momentum oscillator). Calculated internally and displayed in the dashboard only -- not plotted on the chart to avoid cluttering the overlay.

| Parameter | Default | Description |
|-----------|---------|-------------|
| RSI Length | 14 | Lookback period |
| Overbought | 70 | Upper threshold |
| Oversold | 30 | Lower threshold |

**Dashboard status values**:

- **OVERBOUGHT** (red): RSI >= 70. Momentum is extended to the upside; CALLS signals are less reliable.
- **OVERSOLD** (green): RSI <= 30. Momentum is extended to the downside; PUTS signals are less reliable.
- **NEUTRAL** (white): RSI between 30-70. Directional room remains.

For 0DTE scalping, RSI is most useful as a filter rather than a standalone signal. The signal engine requires RSI to be within the neutral zone (between oversold and overbought) for both CALLS and PUTS to fire, avoiding entries at exhaustion points.

### ADX / DMI

The Average Directional Index measures trend strength regardless of direction. +DI and -DI measure the directional components.

| Parameter | Default | Description |
|-----------|---------|-------------|
| DI Length | 14 | Lookback for +DI / -DI calculation |
| ADX Smoothing | 14 | Smoothing period applied to ADX |
| No Trend Threshold | 15 | Below this = NO TRADE regime |
| Weak/Strong Threshold | 25 | Dividing line between weak and confirmed trend |
| Strong Trend Threshold | 40 | Above this = strong directional move |

**ADX status mapping**:

| ADX Value | Status | Color | Implication |
|-----------|--------|-------|-------------|
| < 15 | NO TREND | Red | Chop. Avoid trading. Theta decay will eat you alive. |
| 15 - 24.9 | WEAK TREND | Yellow | Directional bias is forming but not confirmed. |
| 25 - 39.9 | TRENDING | Green | Confirmed trend. Signals are higher conviction. |
| >= 40 | STRONG TREND | Green | Strong directional move. Trend-following entries preferred. |

**+DI / -DI**: displayed in the dashboard as a ratio. When +DI > -DI, bullish pressure dominates. When -DI > +DI, bearish pressure dominates. The dashboard cell is colored green or red accordingly.

### ATR

Average True Range measures realized volatility on the 1-minute timeframe.

| Parameter | Default | Description |
|-----------|---------|-------------|
| ATR Length | 14 | Lookback period |

ATR is used in two ways:

1. **Dashboard display**: gives a real-time read on per-bar volatility. Useful for gauging whether current conditions support the expected move size for your option strike selection.
2. **Key level proximity**: the signal engine uses `ATR * 0.75` as the threshold for determining whether price is "near" a key level (PM High/Low, OR, PDH/PDL). This makes the proximity check adaptive to current volatility rather than using a fixed point distance.

### Key Price Levels

Horizontal reference levels drawn on the chart. Each has a dashed or dotted line style and a right-edge label for identification.

#### Pre-Market High / Low

- **Detection**: tracks the highest high and lowest low during the pre-market session (4:00 AM - 9:29 AM ET).
- **Rendering**: dashed horizontal lines drawn once RTH begins, extending to the right edge.
- **Requirement**: the chart must have Extended Trading Hours enabled in chart settings. Without extended hours data, these levels will not populate.

#### Prior Day High / Low / Close

- **Source**: pulled from the daily timeframe via `request.security()` using the prior completed day's data.
- **Rendering**: dotted horizontal lines with right-edge labels (PDH, PDL, PDC).
- **No repainting risk**: the `[1]` offset with `lookahead_on` accesses only the prior completed daily bar, which is fixed.

#### Opening Range

- **Detection**: captures the high and low of the first N minutes of RTH (configurable, default 5 minutes).
- **Rendering**: dashed horizontal lines with right-edge labels (OR HIGH, OR LOW) drawn once the opening range period completes.
- **Usage**: opening range breakouts/breakdowns are classic 0DTE triggers. Price clearing OR HIGH with volume = bullish continuation. Price breaking OR LOW = bearish.

#### Session HOD / LOD

- **Detection**: running high-of-day and low-of-day tracked from RTH open, updated on every bar.
- **Rendering**: solid horizontal lines with right-edge labels (HOD, LOD) that dynamically move as new highs/lows print.
- **Reset**: clears and starts fresh at each RTH open.

### Regime / Mode Detection

A composite classifier that evaluates the current market state by combining ADX trend strength, EMA ribbon alignment, and price position relative to VWAP.

| Mode | Conditions | Signal Behavior |
|------|-----------|-----------------|
| **BULLISH TREND** | ADX >= 25, EMAs stacked bullish, price above VWAP | CALLS allowed, PUTS suppressed |
| **BEARISH TREND** | ADX >= 25, EMAs stacked bearish, price below VWAP | PUTS allowed, CALLS suppressed |
| **RANGING** | ADX 15-25, no clear EMA stack | Both CALLS and PUTS allowed (lower conviction) |
| **NO TRADE** | ADX < 15 | All signals suppressed |
| **TRANSITION** | ADX >= 15 but mixed EMA/VWAP alignment | Both allowed (use caution) |

The regime classifier serves as a top-level gate on signal generation. Its primary purpose is preventing you from taking directional 0DTE trades during choppy, low-ADX conditions where theta decay dominates and price whipsaws generate false signals.

The mode is displayed prominently in the dashboard and updates in real time. Mode changes also trigger alerts.

### Signal Engine

The core decision engine that generates CALLS and PUTS labels on the chart.

#### Scoring System

Each signal direction (CALLS and PUTS) is evaluated independently using six conditions. Each condition that is true adds one point to the score.

**CALLS conditions (bullish)**:

| # | Condition | Rationale |
|---|-----------|-----------|
| 1 | Price is above VWAP or just crossed above | Institutional bias is bullish |
| 2 | EMA ribbon is bullish or price is above mid+slow EMAs | Trend structure supports upside |
| 3 | RSI is between oversold and overbought (30-70) | Momentum has room to run |
| 4 | ADX > No Trend Threshold (15) | There is measurable directional movement |
| 5 | Bullish candle pattern (engulfing or strong body close) | Price action confirmation |
| 6 | Price is near a support level (PM Low, OR Low, PDL) | Favorable risk/reward at structure |

**PUTS conditions (bearish)**:

| # | Condition | Rationale |
|---|-----------|-----------|
| 1 | Price is below VWAP or just crossed below | Institutional bias is bearish |
| 2 | EMA ribbon is bearish or price is below mid+slow EMAs | Trend structure supports downside |
| 3 | RSI is between oversold and overbought (30-70) | Momentum has room to run |
| 4 | ADX > No Trend Threshold (15) | There is measurable directional movement |
| 5 | Bearish candle pattern (engulfing or strong body close) | Price action confirmation |
| 6 | Price is near a resistance level (PM High, OR High, PDH) | Favorable risk/reward at structure |

#### Signal Filters

A raw score meeting the threshold is necessary but not sufficient. The following filters must also pass:

| Filter | Description |
|--------|-------------|
| **Time Window** | Signal must occur within the configured start/end times (default 9:35 AM - 3:45 PM ET). Avoids the first 5 minutes of RTH chop and the last 15 minutes of gamma/pin risk. |
| **Cooldown** | Minimum number of bars since the last signal of the same direction (default 5 bars = 5 minutes on 1-min chart). Prevents label spam during extended moves. |
| **Bar Confirmation** | When enabled (default), signals only fire on confirmed (closed) bars. Prevents phantom signals from incomplete bars that may reverse before close. |
| **Regime Gate** | CALLS suppressed in BEARISH TREND and NO TRADE modes. PUTS suppressed in BULLISH TREND and NO TRADE modes. |

#### Visual Output

When a signal fires, the following visual elements render on that bar:

1. **Label**: "CALLS" (green, below bar) or "PUTS" (red, above bar) with a hover tooltip showing score, RSI, ADX, VWAP distance, and current mode.
2. **Arrow**: small triangle shape (up for CALLS, down for PUTS) as a secondary quick-scan marker.
3. **Background flash**: faint green or red background highlight on the signal bar (toggleable).

#### VWAP Cross Markers

Independent of the signal engine, small diamond shapes appear on the chart whenever price crosses VWAP during RTH:

- **Lime diamond below bar**: bullish VWAP cross (price crossed above).
- **Red diamond above bar**: bearish VWAP cross (price crossed below).

These are informational markers, not trade signals. They highlight potential inflection points worth watching.

### Dashboard

A real-time information table overlaid on the chart. Positioned in the top-right corner by default (configurable).

#### Layout

Two-column table with a header row and 13 data rows. Dark navy background with high-contrast text. Color-coded values provide instant status reads without needing to interpret raw numbers.

#### Fields

| Row | Field | Value | Color Logic |
|-----|-------|-------|-------------|
| Header | SPY 0DTE DASH | — | Magenta accent bar |
| 1 | Price | Current close + bar direction | Green = green bar, Red = red bar |
| 2 | Mode | Regime classification | Green/Red/Yellow/Gray/Cyan per mode |
| 3 | RSI (14) | RSI value + status | Red = overbought, Green = oversold, White = neutral |
| 4 | ADX | ADX value + trend status | Red = no trend, Yellow = weak, Green = trending |
| 5 | VWAP Dist | Signed distance in points + status | Green = near/normal, Red = disconnected |
| 6 | PM High | Distance to PM High + status | Red = resistance (below), Green = cleared (above) |
| 7 | PM Low | Distance to PM Low + status | Green = support (above), Red = broken (below) |
| 8 | ATR (14) | Current ATR value | Cyan (informational) |
| 9 | EMA Trend | Ribbon alignment status | Green = bullish, Red = bearish, Yellow = mixed |
| 10 | Signal | Last signal on current bar | Green = CALLS, Red = PUTS, Gray = none |
| 11 | Day Chg | Percentage change from session open | Green = positive, Red = negative |
| 12 | Rel Vol | Current volume / 20-bar avg volume | Green + "SPIKE" if > 1.5x, Gray otherwise |
| 13 | +DI / -DI | Directional indicator values | Green = +DI dominant, Red = -DI dominant |

### Alerts

#### Static Alerts (alertcondition)

These appear in TradingView's "Create Alert" dropdown when the indicator is on your chart. You manually configure each alert and choose your notification method.

| Alert Name | Trigger |
|------------|---------|
| 0DTE: CALLS Signal | CALLS signal fires |
| 0DTE: PUTS Signal | PUTS signal fires |
| 0DTE: VWAP Cross Bullish | Price crosses above VWAP during RTH |
| 0DTE: VWAP Cross Bearish | Price crosses below VWAP during RTH |
| 0DTE: Mode Change | Regime mode transitions to a different state |

#### Dynamic Alerts (alert)

These fire automatically with rich, interpolated messages containing real-time context. Useful for webhook integrations (Discord bots, Telegram, custom APIs).

**Example CALLS alert message**:

```
0DTE CALLS | SPY 686.42 | RSI 54.3 | ADX 27.8 | VWAP Dist 0.65 | Score 5/6 | Mode: BULLISH TREND
```

**Example mode change alert message**:

```
0DTE MODE CHANGE | RANGING → BULLISH TREND | SPY 687.10
```

Dynamic alerts use `alert.freq_once_per_bar_close` for signals (no duplicates within a bar) and `alert.freq_once_per_bar` for mode changes.

---

## Visual Reference

### Color Map

All colors are selected for maximum contrast against TradingView's dark theme (pure black background).

| Element | Hex | Preview | Notes |
|---------|-----|---------|-------|
| VWAP | `#FFFFFF` | White | Solid line, weight 2 |
| VWAP Bands | `#00BCD4` | Cyan | 50% transparency, circle style |
| EMA 8 (Fast) | `#FFD600` | Bright Yellow | Thin line |
| EMA 13 (Mid) | `#FF9100` | Bright Orange | Thin line |
| EMA 21 (Slow) | `#FF1744` | Hot Pink/Red | Thin line |
| EMA Cloud (Bull) | `#00E676` | Neon Green | 85% transparent fill |
| EMA Cloud (Bear) | `#FF1744` | Neon Red | 85% transparent fill |
| EMA Cloud (Mixed) | `#FFD600` | Yellow | 90% transparent fill |
| PM High | `#FF5252` | Bright Red | Dashed line + right label |
| PM Low | `#69F0AE` | Bright Green | Dashed line + right label |
| Prior Day High | `#FF6E40` | Coral | Dotted line + "PDH" label |
| Prior Day Low | `#64FFDA` | Teal | Dotted line + "PDL" label |
| Prior Day Close | `#B0BEC5` | Gray | Dotted line + "PDC" label |
| OR High | `#E040FB` | Magenta | Dashed line + "OR HIGH" label |
| OR Low | `#B388FF` | Purple | Dashed line + "OR LOW" label |
| Session HOD | `#FFD740` | Amber | Solid line + "HOD" label |
| Session LOD | `#40C4FF` | Light Blue | Solid line + "LOD" label |
| CALLS Signal | `#00E676` | Neon Green | Label + arrow + bg flash |
| PUTS Signal | `#FF1744` | Neon Red | Label + arrow + bg flash |
| VWAP Cross (Bull) | `#76FF03` | Lime | Diamond shape below bar |
| VWAP Cross (Bear) | `#FF1744` | Red | Diamond shape above bar |

### Chart Element Legend

Every plotted line registers in TradingView's built-in legend (top-left, next to ticker info). You can click any legend entry to toggle its visibility. The entries are:

```
EMA 8 (Fast)  |  EMA 13 (Mid)  |  EMA 21 (Slow)  |  VWAP  |  VWAP +1σ  |  VWAP -1σ
CALLS Arrow  |  PUTS Arrow  |  VWAP Cross Bull  |  VWAP Cross Bear  |  Signal Background Flash
```

Horizontal key levels (PM High/Low, PDH/PDL/PDC, OR, HOD/LOD) are drawn with `line.new()` and labeled at the right edge of the chart, so they are identified by their text labels rather than legend entries.

### Dashboard Fields

See the [Dashboard](#dashboard) section for the full field table. The dashboard uses a dark navy background (`#1A1A2E`) with a magenta header bar and light gray metric labels. Value cells are dynamically colored based on the state of each metric.

---

## Tuning Guide

All parameters are exposed as configurable inputs in TradingView's indicator settings panel. This section explains what each tunable controls and how to adjust it for different trading styles.

### Signal Strictness

**Input**: Signal Strictness
**Options**: Relaxed, Normal, Strict
**Default**: Normal

This is the single most impactful setting. It controls the minimum confluence score required for a signal to fire.

| Setting | Min Score | Signals Per Session (typical) | Best For |
|---------|-----------|-------------------------------|----------|
| Relaxed | 3 of 6 | 15-30+ | Aggressive scalpers who want frequent entries and manage risk via position sizing and quick exits. Higher false positive rate. |
| Normal | 4 of 6 | 5-15 | Balanced approach. Reasonable hit rate with sufficient frequency to capture moves. Recommended starting point. |
| Strict | 5 of 6 | 1-5 | Conservative traders who want only the highest-conviction setups. Fewer trades, but each has strong multi-factor confluence. May miss some moves entirely. |

**Recommendation**: start with Normal. If you find yourself getting stopped out frequently on signals, move to Strict. If you feel like you're missing obvious moves, try Relaxed but tighten your risk management.

### EMA Lengths

**Inputs**: Fast EMA Length, Mid EMA Length, Slow EMA Length
**Defaults**: 8, 13, 21

The EMA ribbon defines short-term trend structure. These lengths are Fibonacci-based and work well on the 1-minute chart for capturing momentum shifts over 8-21 minute windows.

| Adjustment | Effect |
|------------|--------|
| Shorter lengths (e.g., 5/8/13) | More responsive ribbon. Cloud flips faster. Signals trigger earlier but with more noise. Better for ultra-fast scalps (30-second to 2-minute holds). |
| Longer lengths (e.g., 13/21/34) | Smoother ribbon. Cloud flips less often. Signals are slower but more reliable. Better for slightly longer scalps (3-10 minute holds). |
| Wider spread (e.g., 5/13/34) | Cloud is wider. Bullish/bearish stacking requires a stronger move to confirm. Fewer mixed-cloud periods. |

**Important**: the EMA stack (bullish = fast > mid > slow) directly feeds the regime classifier and signal scoring. Changing these lengths changes when the mode flips and when signals fire.

### ADX Thresholds

**Inputs**: No Trend Threshold, Weak/Strong Threshold, Strong Trend Threshold
**Defaults**: 15, 25, 40

ADX thresholds control the regime classifier and the minimum directional strength required for signals.

| Parameter | Tuning Impact |
|-----------|---------------|
| No Trend Threshold (15) | **Raising this** (e.g., to 20) makes the NO TRADE regime more aggressive, suppressing signals in marginal conditions. **Lowering it** (e.g., to 10) allows signals in choppier environments. For 0DTE, erring higher is safer because theta decay punishes low-conviction entries. |
| Weak/Strong Threshold (25) | Controls when the regime classifies as TRENDING vs WEAK/RANGING. Raising this requires stronger trends before directional bias kicks in. |
| Strong Trend Threshold (40) | Informational for the dashboard. Does not directly affect signal logic. |

**Recommendation**: if you trade the opening 30 minutes (high-vol environment), the default of 15 works well. If you trade the midday session where SPY often ranges, consider raising the No Trend Threshold to 18-20 to avoid getting chopped up.

### VWAP Extended Distance

**Input**: VWAP Extended Distance (pts)
**Default**: 2.0

Controls when the dashboard flags price as "DISCONNECTED" from VWAP.

| Value | Behavior |
|-------|----------|
| Lower (e.g., 1.0) | Triggers the DISCONNECTED warning earlier. More conservative. Helps prevent chasing extended moves. |
| Higher (e.g., 3.0) | Allows wider moves before warning. Better for trending days where SPY can sustain 2-3+ point runs from VWAP. |

This value is informational only (affects dashboard display). It does not directly gate signals. However, when price is significantly extended from VWAP, mean-reversion risk increases, and any 0DTE entry has higher adverse-move probability.

**Tip**: on a typical day, SPY's 1-minute ATR is roughly 0.15-0.30 points. A 2-point VWAP distance represents ~7-13 ATR units of extension, which is substantial on an intraday basis.

### Signal Cooldown

**Input**: Cooldown Bars Between Signals
**Default**: 5

Minimum number of bars (minutes on 1-min chart) that must pass before another signal of the same direction can fire.

| Value | Behavior |
|-------|----------|
| Lower (1-3) | Signals fire more frequently during sustained moves. Can result in label clusters during strong trends. Useful if you're scaling into positions. |
| Higher (10-20) | Signals fire rarely. Each one represents a distinct setup well-separated in time. Useful if you take one position at a time and hold for several minutes. |
| Default (5) | One signal per 5-minute window per direction. Reasonable balance for most scalping styles. |

**Note**: CALLS and PUTS cooldowns are tracked independently. A CALLS signal does not reset the PUTS cooldown and vice versa.

### Signal Time Window

**Inputs**: Signal Start Time (HHMM ET), Signal End Time (HHMM ET)
**Defaults**: 0935, 1545

Defines the window during which signals are allowed to fire. Times are in Eastern Time (ET), 24-hour format, no colon.

| Adjustment | Rationale |
|------------|-----------|
| Start at 0930 | Include the opening bar. High volatility, high reward, but also high noise and whipsaw risk. |
| Start at 0935 (default) | Skip the first 5 minutes. Avoids the opening auction chaos where 0DTE spreads are widest and fills are worst. |
| Start at 0945 | Skip the first 15 minutes. Wait for the opening range to establish before taking signals. More conservative. |
| End at 1545 (default) | Stop signals 15 minutes before close. Avoids the final gamma/pin risk window where 0DTE options can swing violently on delta hedging flows. |
| End at 1555 | Allow signals into the last 5 minutes. Only for experienced traders who understand pin risk and are comfortable with rapid gamma decay. |
| End at 1500 | Stop signals an hour before close. Very conservative. Avoids the entire late-day volatility regime shift. |

### Opening Range Duration

**Input**: Opening Range Minutes
**Default**: 5

Number of minutes from RTH open to capture for the opening range.

| Value | Usage |
|-------|-------|
| 5 (default) | 5-minute opening range. The most common institutional reference. Breakouts above OR High or below OR Low on volume are high-probability 0DTE triggers. |
| 15 | 15-minute opening range. More conservative, wider range. Breakouts are higher conviction but take longer to set up. |
| 2-3 | Micro opening range. Very tight levels that break quickly. Useful for the most aggressive opening trades but generates more false breakouts. |

### RSI Thresholds

**Inputs**: Overbought, Oversold
**Defaults**: 70, 30

Controls the RSI zone classification and the signal filter that prevents entries at momentum extremes.

The signal engine requires RSI to be between oversold and overbought for both CALLS and PUTS to fire. This prevents buying calls when momentum is already exhausted (overbought) or buying puts when selling is overdone (oversold).

| Adjustment | Effect |
|------------|--------|
| Tighter (65/35) | Signals are filtered out earlier as RSI approaches extremes. Fewer signals, but each has more momentum room. |
| Wider (75/25) | Allows signals closer to extremes. More signals fire, but some may be at points where the move is nearly exhausted. |

### Tuning Profiles

Here are three preset configurations for common trading styles. Apply these by adjusting the inputs in the indicator settings.

#### Profile: Conservative / Sniper

For traders who want 1-5 high-conviction trades per session.

```
Signal Strictness:     Strict
EMA Lengths:           13 / 21 / 34
ADX No Trend:          20
Cooldown:              10
Signal Start:          0945
Signal End:            1530
RSI OB/OS:             65 / 35
```

#### Profile: Balanced (Default)

For traders who want moderate signal frequency with reasonable accuracy.

```
Signal Strictness:     Normal
EMA Lengths:           8 / 13 / 21
ADX No Trend:          15
Cooldown:              5
Signal Start:          0935
Signal End:            1545
RSI OB/OS:             70 / 30
```

#### Profile: Aggressive / Scalper

For experienced traders who want frequent signals and manage risk through position sizing and rapid exits.

```
Signal Strictness:     Relaxed
EMA Lengths:           5 / 8 / 13
ADX No Trend:          12
Cooldown:              3
Signal Start:          0930
Signal End:            1555
RSI OB/OS:             75 / 25
```

---

## Architecture Notes

**Performance**: the script uses `var` declarations for persistent state (lines, labels, level values, cooldown counters) to avoid reallocation on every bar. `request.security()` calls are minimized (one call per prior-day field). The dashboard table is updated only on `barstate.islast` to reduce computation overhead on historical bars.

**Repainting**: all signal logic is gated by `barstate.isconfirmed` by default. This means signals only appear after a bar closes, not during its formation. This eliminates phantom signals but introduces a 1-bar delay on live charts. You can disable this via the "Require Bar Close Confirmation" toggle if you prefer real-time signals and accept the repainting tradeoff.

**Object limits**: Pine Script v6 allows a maximum of 500 labels, 500 lines, and 500 boxes per script. The indicator's `max_labels_count`, `max_lines_count`, and `max_boxes_count` are all set to 500. On a full trading day (~390 RTH bars on 1-min), signal labels, level labels, and HOD/LOD lines are well within these limits. However, if you run the script across multiple days of extended hours data, older labels and lines will be automatically garbage-collected by TradingView (oldest first).

**VWAP bands**: the band calculation uses cumulative variance (`ta.cum`) which resets with the session. On the first bar of each session, variance is zero and bands will overlap VWAP. This is expected behavior and resolves within a few bars.

**Timezone**: all session detection is hardcoded to `America/New_York`. If TradingView ever changes how `time()` handles timezone parameters in future Pine Script versions, this is the first thing to check.

---

## Known Limitations

1. **No cumulative delta / order flow**: Pine Script does not have access to tick-level bid/ask data. The indicator cannot compute cumulative delta, footprint charts, or order flow imbalances. Relative volume is the closest proxy available.

2. **No volume profile**: Pine Script cannot natively compute volume profile (POC, VAH, VAL). These levels would be highly complementary for 0DTE trading but must be added via a separate indicator or TradingView's built-in VP tool.

3. **Pre-market levels require extended hours**: if your chart does not have extended trading hours enabled, PM High/Low will not populate. The dashboard will show "—" and no PM level lines will render.

4. **Single-symbol**: the script is designed for SPY. It will technically work on any symbol, but the default parameters (VWAP extended distance, ATR-based level proximity, dashboard labeling) are calibrated for SPY's price range and volatility characteristics. Significant parameter adjustment would be needed for other instruments.

5. **Signal scoring is equal-weighted**: all six conditions contribute equally to the score. In practice, some conditions (like VWAP position) may be more predictive than others. A future enhancement could weight conditions differently.

6. **No backtest capability**: this is an `indicator()`, not a `strategy()`. It cannot be backtested through TradingView's strategy tester. Converting to a strategy would require defining entry/exit rules, position sizing, and stop/target logic.

---

## Changelog

### v1.0 (2025-02-18)

- Initial release.
- EMA Ribbon with dynamic cloud fill.
- Session-anchored VWAP with standard deviation bands.
- RSI, ADX/DMI, ATR calculations with dashboard display.
- Pre-Market High/Low, Prior Day H/L/C, Opening Range, Session HOD/LOD levels.
- Five-mode regime classifier (BULLISH TREND, BEARISH TREND, RANGING, NO TRADE, TRANSITION).
- Six-condition confluence scoring engine with configurable strictness.
- Signal filtering (time window, cooldown, bar confirmation, regime gate).
- CALLS/PUTS labels with tooltips, arrow shapes, VWAP cross markers, background flash.
- 14-field real-time dashboard with color-coded status indicators.
- Dual alert system (alertcondition + alert with dynamic messages).
- Full input configurability with grouped settings panel.
