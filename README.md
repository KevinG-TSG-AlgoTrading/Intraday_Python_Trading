# Intraday Python Trading — Day Trading Strategies & Scanners

Backtesting code and results for the systematic/algorithmic side of Kevin Gitia's trading — built to support a Seven Points Capital prop-firm application. Each folder below is one strategy: the notebook that detects/backtests it, plus the raw and summary result files the notebook produced.

## Strategies (backtested)

### [`9-EMA-Continuation/`](./9-EMA-Continuation) — 9-EMA Continuation (5-MIN, futures/NVDL)
Locked, frozen signal stack (9-EMA hold + prior-high intact + positive open + VWAP alignment + QQQ/tape filter), morning session only.
- **Official in-sample benchmark (2023–2025):** 155 trades, 65.8% win rate, +$2,802.67 total, +$18.08/trade, max drawdown –$629.69, +28.0% return.
- `9_EMA_Cont_5MIN_FinalFilter.ipynb` — the locked benchmark notebook (no separate results CSV; the numbers above are computed inside the notebook itself).

### [`200-EMA-MOMO/`](./200-EMA-MOMO) — 200-EMA Counter-Trend Opening Drive
Opening-range mean-reversion off the 200-period EMA on the 5-minute chart, 16-ticker equities/leveraged-ETF basket.
- **Standard exit, 2024–2026:** 1,210 trades, 49.4% win rate, +$26,550.21 total, +$21.94/trade.
- **"Blue Sky" exit variant (rejected, not adopted live):** 104 trades, 30.8% win rate, –$2,443.36.
- `200_EMA_MOMO_AlertTracker.ipynb` + `200_EMA_MOMO_backtest_results.csv` (per-trade) + `200_EMA_MOMO_backtest_summary.csv` (per-ticker).

### [`2MIN-Open-Gate/`](./2MIN-Open-Gate) — 2-MIN Open Gate
Opening-drive continuation read against prior close / prior-day high / premarket high on the first few 2-minute candles, 17-ticker basket, SPY-filtered version kept over unfiltered.
- **Full backtest:** 376 trades, 52.1% win rate, +$4,011.83 total, +$10.67/trade.
- `2MIN_OpenGate_LiveScanner.ipynb` + `2MIN_OpenGate_results.csv` (per-trade) + `2MIN_OpenGate_summary.csv` (per-ticker).

### [`Ending-Volume/`](./Ending-Volume) — Ending Volume (volume-climax exhaustion, counter-trend)
Two-layer stop (soft stop + hard brake), VWAP-branch scaled exit ladder. Raw signal validated at 2.73M-signal scale across 516 tickers; full trade-level ladder backtest below.
- **Full ladder backtest, 2023–2026:** 9,537 trades, 29.9% win rate, net –$297.23 — essentially breakeven on the raw signal, before the full-universe study's recommended time-of-day/opening-range filters are applied to this engine.
- `EndingVolume_PenetrationStudy.ipynb` (brake/signal study) + `EndingVolume_LadderEngine.ipynb` (trade-level exit backtest) + `ENDVOL_penetration_summary.csv` + `ENDVOL_ladder_trades.csv` (per-trade) + `ENDVOL_ladder_legs.csv` (per-leg scale-out detail).

## Live tooling

### [`Volume-Discovery-Scanner/`](./Volume-Discovery-Scanner) — real-time volume-discovery scanner & alert dashboard
Not a backtested strategy — the live infrastructure that feeds Ending Volume and the other setups with candidates in real time. Scans a ~600-ticker universe (S&P 500 + Nasdaq 100 + a manually-curated extras list for IPOs/leveraged ETFs) every 2-minute bar, scores each print as current-bar volume ÷ the ticker's own trailing 5-session median at that time slot (median chosen over mean specifically so one outlier day in the lookback can't distort the baseline), and surfaces anything abnormal on a live Tkinter dashboard with two tabs (today's top-10 board, re-ranked every bar; a running log of everything that hit the board that day). High-conviction prints (15x+ baseline volume) also trigger a spoken voice alert (Windows SAPI, with a `pyttsx3` fallback) on a background thread so a blocking text-to-speech call can't stall the next scan cycle.
- **Running log, Aug 24–Sep 22, 2026:** 821 logged alerts across 401 unique tickers; 77 of those cleared the 15x voice-alert threshold. Standout print: GME at 333.6x its baseline on 9/1/2026.
- `Volume_Discovery_Scanner.ipynb` (scanner + dashboard + voice alerting) + `VOL_discovery_log.csv` (every alert logged: timestamp, ticker, score, acceleration, volume, baseline, price, HOD flag).

## Notes
- All notebooks were developed and run in a local Jupyter environment (data sourced via the Alpaca API), not in a hosted/cloud pipeline.
- Numbers above are reproducible directly from the CSVs in each folder — nothing here is hand-typed.
