# Libraries & SDKs

API clients, SDKs, and utility libraries for prediction market integration.

## Quick Reference

| Library                                               | Platform   | Rating | Async | Verdict              |
| ----------------------------------------------------- | ---------- | ------ | ----- | -------------------- |
| [kalshi-py](./kalshi-py.md)                           | Kalshi     | 5/5    | Yes   | **Recommended**      |
| [kalshi-unofficial](./kalshi-unofficial.md)           | Kalshi     | 3/5    | WS    | WebSocket only       |
| [kalshi-python](./kalshi-python.md)                   | Kalshi     | 3.5/5  | No    | Private source       |
| [py-clob-client](./py-clob-client.md)                 | Polymarket | 3.5/5  | No    | Use (with care)      |
| [dr-manhattan](./dr-manhattan.md)                     | Multi      | 4/5    | No    | Fork it              |
| [polymarket-agents](./polymarket-agents.md)           | Polymarket | -      | -     | Study patterns       |
| [aiokalshi](./aiokalshi.md)                           | Kalshi     | 4.5/5  | Yes   | Read-only (no trade) |
| [Prediction Market Data](./prediction-market-data.md) | Multi      | -      | -     | Landscape            |

## Kalshi (Recommended Stack)

For the underdog volatility bot, use this combination:

```
kalshi-py          → Trading operations (async, typed, open source)
kalshi-unofficial  → WebSocket for real-time orderbook
```

### Primary: kalshi-py

- **[kalshi-py](./kalshi-py.md)** - Modern, type-safe client with full async support.
  Open source, actively maintained, 100% type hints. **5/5 - Use this.**

### WebSocket: kalshi-unofficial

- **[kalshi-unofficial](./kalshi-unofficial.md)** - Lightweight wrapper (526 lines) with
  WebSocket support for real-time orderbook streaming. Essential for volatility
  detection. **3/5 - Use for WebSocket only.**

### Official SDK (Not Recommended)

- **[kalshi-python](./kalshi-python.md)** - Official Kalshi SDK on PyPI. Works, but
  source code is private (red flag for trading systems). Sync-only, no async. **3.5/5 -
  Avoid.**

### Async-Only (Incomplete)

- **[aiokalshi](./aiokalshi.md)** - Beautiful async client with Pydantic models, but
  read-only (no authentication, no trading). Watch for future development. **4.5/5
  quality, but can't trade.**

## Polymarket

- **[py-clob-client](./py-clob-client.md)** - Official Python client. Production-ready
  but sync-only, some reliability issues with order book data. **3.5/5**

- **[dr-manhattan](./dr-manhattan.md)** - CCXT-style multi-exchange abstraction with
  strategy framework and WebSocket support. Best candidate for forking. **4/5**

## AI/Agent Frameworks

- **[polymarket-agents](./polymarket-agents.md)** - Official Polymarket framework for
  LLM-powered trading agents. 975 stars. Essential reference for our architecture.

## Data Aggregation

- **[Prediction Market Data](./prediction-market-data.md)** - Landscape analysis of
  unified data libraries including predmarket, plus commercial options.

## Strategic Recommendation

**For Kalshi (now):**

1. Use `kalshi-py` for all trading operations
2. Add `kalshi-unofficial` for WebSocket orderbook streaming
3. Build thin wrapper combining both

**For Polymarket (later):**

- Fork `dr-manhattan`, strip deps, build maintained version

## Reference Repositories

Libraries are cloned to `../../reference/` for direct inspection:

```
reference/
├── kalshi-py/                 # Recommended Kalshi client
├── kalshi-python/             # Official starter code (not SDK)
├── kalshi-python-unofficial/  # WebSocket support
├── kalshi-tools/              # Official analysis tools
├── KalshiPythonClient/        # Older OpenAPI client
├── aiokalshi/                 # Async read-only client
├── dr-manhattan/              # Multi-exchange framework
├── py-clob-client/            # Polymarket client
└── ...
```

## See Also

- **[Platforms](../platforms/)** - Authentication details, API setup, rate limits
