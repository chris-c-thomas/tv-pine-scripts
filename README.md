# Pine Scripts for TradingView

Technical trading indicators for the TradingView charting platform, written in Pine Script v6.

## Indicators

### Market Monitor

A directional bias overlay designed for multi-chart watchlist monitoring. Combines EMA ribbon, VWAP bands, RSI, ADX/DMI, and relative volume into a composite bias score (-5 to +5) displayed in a compact dashboard. Optimized for running 6-8 instances simultaneously on small chart tiles.

| Script | Version | Docs |
| ------ | ------- | ---- |
| [market_monitor_v1_0.pine](market_monitor/market_monitor_v1_0.pine) | v1.0 | [docs](docs/market_monitor_v1_0.md) |

### SPY 0DTE Scalper

A high-precision scalping indicator for zero-days-to-expiration SPY options. Features regime classification, opening range detection, pre-market levels, and a multi-condition AND-gate signal engine. V1.1 adds TTM Squeeze detection, NYSE TICK breadth filtering, and VWAP cross markers.

| Script | Timeframe | Version | Docs |
| ------ | --------- | ------- | ---- |
| [spy_0dte_scalper_1min_v1.0.pine](spy_0dte_scalper/spy_0dte_scalper_1min_v1.0.pine) | 1-minute | v1.0 | [docs](docs/spy_0dte_scalper_1min_v1.0.md) |
| [spy_0dte_scalper_1min_v1.1.pine](spy_0dte_scalper/spy_0dte_scalper_1min_v1.1.pine) | 1-minute | v1.1 | [docs](docs/spy_0dte_scalper_1min_v1.1.md) |
| [spy_0dte_scalper_5min_v1.0.pine](spy_0dte_scalper/spy_0dte_scalper_5min_v1.0.pine) | 5-minute | v1.0 | [docs](docs/spy_0dte_scalper_5min_v1.0.md) |
| [spy_0dte_scalper_5min_v1.1.pine](spy_0dte_scalper/spy_0dte_scalper_5min_v1.1.pine) | 5-minute | v1.1 | [docs](docs/spy_0dte_scalper_5min_v1.1.md) |
| [spy_0dte_scalper_15min_v1_0.pine](spy_0dte_scalper/spy_0dte_scalper_15min_v1_0.pine) | 15-minute | v1.0 | [docs](docs/spy_0dte_scalper_15min_v1_0.md) |
| [spy_0dte_scalper_15min_v1_1.pine](spy_0dte_scalper/spy_0dte_scalper_15min_v1_1.pine) | 15-minute | v1.1 | [docs](docs/spy_0dte_scalper_15min_v1_1.md) |

## License

MIT License. See [LICENSE](LICENSE).
