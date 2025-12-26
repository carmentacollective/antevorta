# Libraries & SDKs

API clients, SDKs, and utility libraries for prediction market integration.

## Quick Reference

| Library                                               | Platform   | Rating | Verdict         |
| ----------------------------------------------------- | ---------- | ------ | --------------- |
| [py-clob-client](./py-clob-client.md)                 | Polymarket | 3.5/5  | Use (with care) |
| [kalshi-python](./kalshi-python.md)                   | Kalshi     | 4/5    | Use (official)  |
| [dr-manhattan](./dr-manhattan.md)                     | Multi      | 4/5    | Fork it         |
| [polymarket-agents](./polymarket-agents.md)           | Polymarket | -      | Study patterns  |
| [aiokalshi](./aiokalshi.md)                           | Kalshi     | 2/5    | Skip            |
| [Prediction Market Data](./prediction-market-data.md) | Multi      | -      | Landscape       |

## Polymarket

- **[py-clob-client](./py-clob-client.md)** - Official Python client. Production-ready
  but sync-only, some reliability issues with order book data. 3.5/5

- **[dr-manhattan](./dr-manhattan.md)** - CCXT-style multi-exchange abstraction with
  strategy framework and WebSocket support. Best candidate for forking. 4/5

## Kalshi

- **[kalshi-python](./kalshi-python.md)** - Official SDK with full API coverage.
  OpenAPI-generated, all endpoints supported. 4/5 - Use this.

- **[aiokalshi](./aiokalshi.md)** - Async client with nice Pydantic models, but
  read-only (no trading). 2/5 - Skip.

## AI/Agent Frameworks

- **[polymarket-agents](./polymarket-agents.md)** - Official Polymarket framework for
  LLM-powered trading agents. 975 stars. Essential reference for our architecture.

## Data Aggregation

- **[Prediction Market Data](./prediction-market-data.md)** - Landscape analysis of
  unified data libraries including predmarket, plus commercial options.

## Strategic Recommendation

**Short-term**: Use official SDKs (py-clob-client + kalshi-python) with our own
normalization layer.

**Medium-term**: Fork dr-manhattan, strip unnecessary deps, add Kalshi support, build
our own maintained version.

## See Also

- **[Platforms](../platforms/)** - Authentication details, API setup, multi-user
  considerations for Kalshi and Polymarket

## Reference Repositories

Libraries are cloned to `../../reference/` for direct inspection.
