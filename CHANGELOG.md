# Changelog

All notable changes to this project are documented in this file.

---

## v1.1 (2026-02-20) -- All Scalper Variants

- TTM Squeeze detection with three-state classification and momentum histogram.
- NYSE TICK Index integration with six-tier breadth classification.
- Expanded dashboard rows for squeeze and TICK status.
- New alert conditions for squeeze fired events.
- Enhanced dynamic alert messages with squeeze and TICK context.

**1-Minute specific**: Squeeze and TICK added as optional AND-gate filters (default OFF). VWAP cross markers added.

**5-Minute specific**: Squeeze recency and TICK breadth added as scoring conditions #8 and #9 in the 5+4 engine. Level proximity added as scoring condition #7. Expanded candle pattern detection (inside bar breakout, 3-bar momentum).

**15-Minute specific**: Integrated into scoring and directional bias system with timeframe-appropriate calibration.

---

## v1.0 (2026-02-18) -- Initial Release

- EMA Ribbon with dynamic cloud fill.
- Session-anchored VWAP with standard deviation bands.
- RSI, ADX/DMI, ATR calculations with dashboard display.
- Pre-market high/low, prior day H/L/C, opening range, session HOD/LOD levels.
- Five-mode regime classifier.
- Signal engine (AND-gate for 1-min, scoring for 5-min and 15-min).
- CALLS/PUTS labels with tooltips, arrow shapes, background flash.
- Real-time dashboard with color-coded status indicators.
- Dual alert system (alertcondition + dynamic alert).
- Market Monitor v1.0 with composite bias scoring (-5 to +5).
