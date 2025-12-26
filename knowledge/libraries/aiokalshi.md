# aiokalshi - Async Kalshi Python Client

**Repository**:
[github.com/the-odds-company/aiokalshi](https://github.com/the-odds-company/aiokalshi)
**Maintainer**: [The Odds Company](https://github.com/the-odds-company)
([@ashercn97](https://github.com/ashercn97)) **Status**: Early Development (v0.1.0) -
"Under rapid development. Expect breaking changes." **Stars/Activity**: 16 stars |
Created Oct 8, 2025 | Last commit Oct 9, 2025 (only 4 commits total)

## Overview

aiokalshi is an asyncio-native Python client for Kalshi's REST API. It provides:

- Native async/await support via `httpx.AsyncClient`
- Full type hints with Pydantic models for all responses
- Hierarchical resource-based API matching Kalshi's REST structure
- No authentication required for public endpoints

The async pattern is clean and idiomatic:

```python
from httpx import AsyncClient
from aiokalshi import Kalshi

async def main():
    async with AsyncClient(follow_redirects=True) as client:
        kalshi = Kalshi(client)
        markets = await kalshi.markets.list()
        orderbook = await kalshi.markets.orderbook.get(ticker, depth=5)
```

**Dependencies** (modern stack):

- `httpx>=0.28.1` - async HTTP
- `pydantic>=2.12.0` - typed models
- `structlog>=25.4.0` - structured logging
- `yarl>=1.22.0` - URL handling
- **Requires Python 3.12+**

## API Coverage

### Supported Endpoints (READ-ONLY)

| Resource       | Methods                                     | Notes              |
| -------------- | ------------------------------------------- | ------------------ |
| Markets        | `list()`, `get(ticker)`                     | Paginated listing  |
| Orderbooks     | `get(ticker, depth)`                        | Market depth data  |
| Trades         | `list(ticker, limit)`                       | Historical trades  |
| Candlesticks   | `get(series, ticker, start, end, interval)` | OHLC data          |
| Events         | `list(status, limit)`, `get(event_ticker)`  | Event catalog      |
| Event Metadata | `list(event_ticker)`                        | Event details      |
| Series         | `list(category)`, `get(ticker)`             | Series by category |

### Missing (Critical for Trading Bot)

| Category           | Missing Endpoints                      |
| ------------------ | -------------------------------------- |
| **Authentication** | No auth support yet                    |
| **Portfolio**      | Balance, positions, fills, settlements |
| **Orders**         | Create, cancel, modify, batch orders   |
| **Account**        | API keys, user info                    |
| **WebSocket**      | "In development" per README            |

**Verdict**: Read-only market data client. Cannot trade.

## Code Quality

### Type Hints

- Full type hints throughout
- Pydantic v2 models for all responses
- Editor autocomplete works well

### Tests

- No test directory visible in repository
- No CI/CD configuration found
- No test coverage

### Documentation

- README with usage examples
- No API reference docs
- No docstrings visible from repository structure

### Async Implementation

- Clean async/await patterns
- Uses httpx (modern, well-maintained)
- Follows Python async best practices
- Resource hierarchy pattern (`kalshi.markets.orderbook.get()`)

### Project Structure

```
src/aiokalshi/
    __init__.py
    client.py
    models/      # Pydantic models
    resources/   # API resource modules
```

## Usage Examples

### Basic Market Data

```python
from httpx import AsyncClient
from aiokalshi import Kalshi

async def get_market_data():
    async with AsyncClient(follow_redirects=True) as client:
        kalshi = Kalshi(client)

        # List markets with pagination
        markets = await kalshi.markets.list(limit=100)

        # Get specific market
        market = await kalshi.markets.get("KXBTC-25DEC31")

        # Get orderbook
        orderbook = await kalshi.markets.orderbook.get(
            market.ticker,
            depth=10
        )

        # Get recent trades
        trades = await kalshi.markets.trades.list(
            market.ticker,
            limit=50
        )
```

### Candlestick Data

```python
async def get_ohlc():
    async with AsyncClient(follow_redirects=True) as client:
        kalshi = Kalshi(client)

        candles = await kalshi.markets.candlesticks.get(
            series_ticker="KXBTC",
            market_ticker="KXBTC-25DEC31",
            start_ts=1700000000,
            end_ts=1700086400,
            period_interval=60  # 1 hour
        )
```

## Integration Effort

### Pros

- Clean async API design
- Type safety with Pydantic
- Modern dependency stack (httpx, structlog)
- Mirrors Kalshi API structure

### Cons / Gotchas

- **No trading capability** - read-only
- **No authentication** - can't access portfolio
- **Python 3.12+ required** - may conflict with other dependencies
- **4 commits, 2 months stale** - minimal development activity
- **No license** - legal risk for commercial use
- **No tests** - risky to depend on
- **No WebSocket** - can't get real-time updates

### Adoption Complexity

- **Market data only**: Would need to build all trading functionality ourselves
- **Fork likely required**: To add auth, portfolio, orders
- **Maintenance burden**: Young project with uncertain future

## Comparison to Alternatives

| Library                  | Async | Types    | Trading | Stars | Activity          | Notes             |
| ------------------------ | ----- | -------- | ------- | ----- | ----------------- | ----------------- |
| **aiokalshi**            | Yes   | Pydantic | No      | 16    | 2 months stale    | Read-only         |
| kalshi-python (official) | No    | Swagger  | Yes     | 2     | 3 years stale     | Auto-generated    |
| kalshi-py                | Yes   | attrs    | Yes     | 1     | Active (Nov 2025) | OpenAPI-generated |
| Build our own            | Yes   | Custom   | Yes     | N/A   | N/A               | Full control      |

### Official SDK (kalshi-python)

- Sync-only (no async)
- Auto-generated from Swagger (v2.1.4, Sep 2025)
- Full trading support
- Actively maintained by Kalshi

### kalshi-py by APTy

- Both sync AND async support
- Built daily from OpenAPI spec
- Full trading support including RSA-PSS auth
- Better documentation
- More actively maintained

## Verdict

**Do Not Use** for Antevorta trading bot.

### Reasons:

1. **No trading capability** - fundamental blocker
2. **No authentication** - can't access portfolio or place orders
3. **Abandoned** - only 4 commits, last activity Oct 2025
4. **No license** - legal risk
5. **No tests** - reliability concern
6. **Python 3.12+** - version constraint

### Recommendations:

**Option 1: Use kalshi-py** (Best for rapid development)

- Async + sync support
- Full trading API coverage
- OpenAPI-generated (stays current)
- Active maintenance

**Option 2: Use official kalshi-python + async wrapper**

- Official support
- Full API coverage
- Wrap sync calls with `asyncio.to_thread()`

**Option 3: Build our own async client**

- Full control over implementation
- Optimized for our specific needs
- Can add MCP server integration
- More effort but cleaner integration with Antevorta

### For Antevorta specifically:

Given our MCP-based architecture, **building our own Kalshi MCP server** (similar to our
Polymarket approach) would give us the cleanest integration. We could use aiokalshi's
Pydantic models and async patterns as inspiration, but implement the full trading API
ourselves.

**Rating**: 2/5

Excellent async design principles, but fundamentally incomplete for trading use cases.
