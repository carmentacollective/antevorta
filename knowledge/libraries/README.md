# Libraries & SDKs

API clients, SDKs, and utility libraries for prediction market integration.

## Quick Reference

| Library                                               | Platform   | Rating | Verdict         |
| ----------------------------------------------------- | ---------- | ------ | --------------- |
| [py-clob-client](./py-clob-client.md)                 | Polymarket | 3.5/5  | Use (with care) |
| [dr-manhattan](./dr-manhattan.md)                     | Multi      | 4/5    | Fork it         |
| [aiokalshi](./aiokalshi.md)                           | Kalshi     | 2/5    | Skip            |
| [Prediction Market Data](./prediction-market-data.md) | Multi      | -      | Landscape       |

## Polymarket

- **[py-clob-client](./py-clob-client.md)** - Official Python client. Production-ready
  but sync-only, some reliability issues with order book data. 3.5/5

- **[dr-manhattan](./dr-manhattan.md)** - CCXT-style multi-exchange abstraction with
  strategy framework and WebSocket support. Best candidate for forking. 4/5

## Kalshi

- **[aiokalshi](./aiokalshi.md)** - Async client with nice Pydantic models, but
  read-only (no trading). 2/5 - Skip.

- **kalshi-py** (not analyzed separately) - Better alternative with async + sync
  support, full trading. See prediction-market-data.md.

## Data Aggregation

- **[Prediction Market Data](./prediction-market-data.md)** - Landscape analysis of
  unified data libraries including predmarket, plus commercial options.

## Strategic Recommendation

**Short-term**: Use official SDKs (py-clob-client + kalshi-python) with our own
normalization layer.

**Medium-term**: Fork dr-manhattan, strip unnecessary deps, add Kalshi support, build
our own maintained version.

## Reference Repositories

Libraries are cloned to `../../reference/` for direct inspection.
