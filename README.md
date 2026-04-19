# Wall Street Analyzer

A Python-based trading signal assistant (renamed to Wall Street Analyzer) that ingests market data, applies multi-timeframe technical analysis, scores trade setups, and delivers actionable alerts via Streamlit dashboard and Telegram/WeCom notifications.

---

## Features

- **Multi-timeframe analysis**: 1m, 5m, 15m, 1h, 1d
- **Indicator engine**: EMA 20/50, RSI 14, MACD, ATR 14, swing high/low
- **Transparent scoring**: rule-based, explainable conviction levels (High / Medium / Low / No-trade)
- **Risk filters**: RR ratio, ATR-based stop check, higher-TF conflict detection
- **Streamlit dashboard**: live signal cards, watchlist overview, history table
- **Notifications**: Telegram + WeCom webhook
- **SQLite persistence**: signal history with status tracking
- **Modular providers**: FMP and Alpha Vantage adapters, easy to extend

---

## Project Structure

```
trading-signal-assistant/
├── app/
│   ├── core/          # Config, logging, enums
│   ├── data/          # Providers, fetcher, cache, models
│   ├── indicators/    # Trend, momentum, volatility, structure
│   ├── analysis/      # Context, scoring, filters, engine
│   ├── signals/       # Signal model, generator, formatter
│   ├── notifications/ # Telegram, WeCom, dispatcher
│   ├── storage/       # SQLite DB, repository
│   ├── scheduler/     # APScheduler jobs
│   └── ui/            # Streamlit dashboard
├── config/
│   ├── app.yaml       # App-level settings
│   ├── watchlist.yaml # Symbols and timeframes
│   └── thresholds.yaml# Scoring weights, risk params
├── tests/             # pytest test suite
├── scripts/           # CLI helpers
├── .env.example       # Environment variable template
└── requirements.txt
```

---

## Setup

### 1. Prerequisites

- Python 3.11+
- A free or paid [Financial Modeling Prep](https://financialmodelingprep.com) API key

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Configure environment

```bash
cp .env.example .env
# Edit .env and fill in your API keys
```

Required keys:
| Variable | Description |
|---|---|
| `FMP_API_KEY` | Financial Modeling Prep API key (primary data source) |
| `TELEGRAM_BOT_TOKEN` | Telegram bot token (optional, for alerts) |
| `TELEGRAM_CHAT_ID` | Telegram chat/channel ID (optional) |
| `WECOM_WEBHOOK_URL` | WeCom robot webhook URL (optional) |

### 4. (Optional) Customize watchlist

Edit [`config/watchlist.yaml`](config/watchlist.yaml) to add/remove symbols or adjust timeframes.

### 5. (Optional) Adjust thresholds

Edit [`config/thresholds.yaml`](config/thresholds.yaml) to tune:
- Minimum risk/reward ratio
- Scoring weights per dimension
- Conviction level thresholds

---

## Running

### Streamlit Dashboard

```bash
streamlit run app/ui/streamlit_app.py
```

Open [http://localhost:8501](http://localhost:8501) in your browser.

### CLI — one-shot signal run

```bash
python scripts/run_analysis.py
```

### Smoke test (no API keys needed)

```bash
python scripts/smoke_test.py
```

### Unit tests

```bash
pytest tests/ -v
```

---

## How It Works

### Data flow

```
Watchlist → DataFetcher → Provider (FMP/AV) → Cache → CandleSeries
                                                          │
                                                 Indicator Pipeline
                                                   (EMA, RSI, MACD, ATR, Swing)
                                                          │
                                                 MarketContext (per TF)
                                                          │
                                            Multi-TF Direction Decision
                                                          │
                                              Scoring → RiskFilters
                                                          │
                                                    TradeSignal
                                              ┌──────────┴──────────┐
                                         Dashboard           Notifications
                                         (Streamlit)    (Telegram / WeCom)
                                                          │
                                                    SQLite Storage
```

### Signal output example

```
========================================
SIGNAL: XAUUSD | LONG | HIGH
========================================
Setup:       trend_continuation_breakout
Confidence:  72%
RR:          2.10

Entry:       1825.50000
Stop Loss:   1820.30000
TP1:         1836.40000
TP2:         1844.25000
Invalidation: Price closes beyond 1820.3 on higher TF candle

TF Alignment:
  5m: up
  15m: up
  1h: up
  1d: up

Reasons:
  • 5m: trend=up, mom=bullish, vol=expanding
  • 1h: trend=up, mom=bullish, vol=neutral
  • Trend score=0.95, momentum=0.88
========================================
```

---

## Extending

### Add a new data provider

1. Create `app/data/providers/my_provider.py` subclassing `BaseProvider`
2. Implement `fetch_candles()` and `_normalize()` 
3. Register in `app/data/fetcher.py`

### Add a new indicator

Add a function to the relevant file in `app/indicators/` and call it from `app/indicators/__init__.py:build_indicator_df()`.

### Add a new notification channel

1. Create `app/notifications/my_channel.py` subclassing `BaseNotifier`
2. Add an instance to `NotificationDispatcher.__init__()`

---

## Roadmap (Phase 2+)

- [ ] Broker integration (paper trading)
- [ ] News sentiment filter
- [ ] PostgreSQL backend
- [ ] Signal outcome tracking (hit TP / hit stop)
- [ ] Backtesting module
- [ ] FastAPI REST layer
- [ ] Docker deployment
