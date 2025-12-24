# aiokalshi - Async-Native Kalshi Client

**Repository**:
[the-odds-company/aiokalshi](https://github.com/the-odds-company/aiokalshi) **Status**:
Active (rapid development) **Platform**: Kalshi **Type**: Async Python client library
**Local Clone**: `../reference/aiokalshi`

## Overview

Async-native, fully type-hinted Python client for Kalshi's REST API. Provides friendly
interface with editor autocomplete, parity with Kalshi's API, and asyncio-first design.

**Key Features**:

- Async/await for concurrent requests
- Full type hints with Pydantic models
- No authentication required for public endpoints
- Mirrors Kalshi's API structure
- WebSocket support in development

## Architecture: Async-First Design

**Core Client Pattern**:

```python
from httpx import AsyncClient
from aiokalshi import Kalshi

async def main():
    async with AsyncClient(follow_redirects=True) as client:
        kalshi = Kalshi(client)

        # All methods are async
        markets = await kalshi.markets.list()
        market = await kalshi.markets.get(markets.markets[0].ticker)
```

**Hierarchical API Structure**:

```python
kalshi.markets.list()           # List markets
kalshi.markets.get(ticker)      # Get one market
kalshi.markets.orderbook.get()  # Market orderbook
kalshi.markets.trades.list()    # Market trades
kalshi.markets.candlesticks.get()  # OHLCV data

kalshi.events.list()            # List events
kalshi.events.get(ticker)       # Get one event
kalshi.events.metadata.list()   # Event metadata

kalshi.series.list()            # List series
kalshi.series.get(ticker)       # Get one series
```

**Mirrors Kalshi's REST API hierarchy** - If you know Kalshi's API, you know aiokalshi.

## API Coverage

### Markets

**List Markets**:

```python
markets = await kalshi.markets.list(
    limit=100,
    cursor=None,
    event_ticker="EVENT-TICKER",
    series_ticker="SERIES-TICKER",
    max_close_ts=1700000000,
    min_close_ts=1600000000,
    status="open",  # open, closed, settled
    tickers="TICKER1,TICKER2"
)

# Returns typed response
for market in markets.markets:
    print(f"{market.ticker}: {market.yes_bid} / {market.yes_ask}")
```

**Get Single Market**:

```python
market = await kalshi.markets.get("MARKET-TICKER")
print(f"{market.ticker}: {market.volume}")
```

**Orderbook**:

```python
orderbook = await kalshi.markets.orderbook.get(
    ticker="MARKET-TICKER",
    depth=5  # Number of price levels
)

print(f"Best bid: {orderbook.yes[0][0]} @ {orderbook.yes[0][1]}")
print(f"Best ask: {orderbook.no[0][0]} @ {orderbook.no[0][1]}")
```

**Trade History**:

```python
trades = await kalshi.markets.trades.list(
    ticker="MARKET-TICKER",
    limit=100,
    cursor=None,
    min_ts=1700000000,
    max_ts=1700086400
)

for trade in trades.trades:
    print(f"{trade.created_time}: {trade.yes_price} @ {trade.count}")
```

**Candlestick Data (OHLCV)**:

```python
candlesticks = await kalshi.markets.candlesticks.get(
    series_ticker="SERIES-TICKER",
    market_ticker="MARKET-TICKER",
    start_ts=1700000000,
    end_ts=1700086400,
    period_interval=60  # 1 hour = 60 minutes
)

for candle in candlesticks.candlesticks:
    print(f"{candle.ts}: O={candle.open} H={candle.high} L={candle.low} C={candle.close} V={candle.volume}")
```

### Events

**List Events**:

```python
events = await kalshi.events.list(
    limit=100,
    cursor=None,
    series_ticker="SERIES-TICKER",
    status="open",  # open, closed, finalized, settled
    with_nested_markets=True
)

for event in events.events:
    print(f"{event.event_ticker}: {event.title}")
```

**Get Single Event**:

```python
event = await kalshi.events.get("EVENT-TICKER")
print(f"{event.event.title}: {len(event.event.markets)} markets")
```

**Event Metadata**:

```python
metadata = await kalshi.events.metadata.list(
    event_ticker="EVENT-TICKER",
    limit=100,
    cursor=None
)
```

### Series

**List Series**:

```python
series_list = await kalshi.series.list(
    limit=100,
    cursor=None,
    category="Science and Technology"  # Optional category filter
)

for series in series_list.series:
    print(f"{series.ticker}: {series.title}")
```

**Get Single Series**:

```python
series = await kalshi.series.get("SERIES-TICKER")
print(f"{series.ticker}: {series.description}")
```

## Type Safety with Pydantic

**Every Response is Typed**:

```python
# Market model
@dataclass
class Market:
    ticker: str
    event_ticker: str
    market_type: str
    title: str
    subtitle: str
    yes_bid: Optional[float]
    yes_ask: Optional[float]
    no_bid: Optional[float]
    no_ask: Optional[float]
    last_price: Optional[float]
    volume: int
    open_time: datetime
    close_time: datetime
    status: str
    # ... all fields from Kalshi API

# Trade model
@dataclass
class Trade:
    ticker: str
    yes_price: float
    no_price: float
    count: int
    taker_side: str  # "yes" or "no"
    created_time: datetime

# Orderbook model
@dataclass
class Orderbook:
    yes: List[Tuple[float, int]]  # [(price, size), ...]
    no: List[Tuple[float, int]]
```

**Autocomplete Works**:

```python
market = await kalshi.markets.get("TICKER")

# Editor suggests: ticker, event_ticker, title, yes_bid, yes_ask, etc.
print(market.yes_bid)
```

## Async Patterns

**Concurrent Requests**:

```python
import asyncio

async def fetch_multiple_markets(tickers: List[str]):
    async with AsyncClient() as client:
        kalshi = Kalshi(client)

        # Fetch all markets concurrently
        tasks = [kalshi.markets.get(ticker) for ticker in tickers]
        markets = await asyncio.gather(*tasks)

        return markets

# Fetch 10 markets in parallel
tickers = ["TICKER1", "TICKER2", ..., "TICKER10"]
markets = await fetch_multiple_markets(tickers)
```

**Streaming Updates**:

```python
async def monitor_orderbook(ticker: str, interval: int = 5):
    async with AsyncClient() as client:
        kalshi = Kalshi(client)

        while True:
            book = await kalshi.markets.orderbook.get(ticker, depth=5)
            print(f"Spread: {book.yes[0][0] - book.no[0][0]}")
            await asyncio.sleep(interval)

# Monitor orderbook every 5 seconds
await monitor_orderbook("MARKET-TICKER")
```

## Code Quality

### Strengths

- ✅ **Async-native** - True async/await, not sync wrapper
- ✅ **Full type hints** - Pydantic models for all responses
- ✅ **Clean API design** - Mirrors Kalshi's structure
- ✅ **No auth required** - Public endpoints work without credentials
- ✅ **Editor autocomplete** - Pydantic enables great DX
- ✅ **httpx-based** - Modern async HTTP client

### Weaknesses

- ❌ **Rapid development** - Breaking changes expected
- ❌ **No authentication** - Missing private endpoints (orders, portfolio)
- ❌ **No WebSocket** - Still in development
- ❌ **Minimal documentation** - README only
- ❌ **No error handling** - Raw httpx exceptions propagate
- ❌ **No rate limiting** - Manual rate limit management

## Unique Innovations

### 1. Async-Native Design 🔥

**Novel aspect**: True async/await, not blocking with async wrapper.

Many "async" clients just wrap sync code:

```python
# Fake async (blocking)
async def fetch_markets(self):
    return await asyncio.to_thread(self._sync_fetch_markets)
```

**aiokalshi is truly async**:

```python
# True async (non-blocking)
async def list(self):
    response = await self.client.get("/markets")  # httpx AsyncClient
    return MarketsResponse(**response.json())
```

Enables true concurrency - fetch 100 markets in parallel without blocking.

### 2. Hierarchical API Mirroring 🔥

**Novel aspect**: API structure mirrors Kalshi's REST endpoints exactly.

```python
# Kalshi API: GET /markets/{ticker}
kalshi.markets.get(ticker)

# Kalshi API: GET /markets/{ticker}/orderbook
kalshi.markets.orderbook.get(ticker)

# Kalshi API: GET /events/{ticker}
kalshi.events.get(ticker)
```

**If you know Kalshi's API docs, you know aiokalshi**. No translation layer.

### 3. Pydantic for Type Safety 🔥

**Novel aspect**: Every response is a validated Pydantic model.

```python
markets = await kalshi.markets.list()

# Type checker knows this is MarketsResponse
# Editor autocompletes: markets.markets, markets.cursor
for market in markets.markets:
    # Type checker knows this is Market
    # Editor autocompletes: market.ticker, market.yes_bid, etc.
    print(market.yes_bid)
```

Catches errors at development time, not runtime.

### 4. Convention: list() vs get() 🔥

**Novel aspect**: Clear naming convention for pagination.

```python
# list() - paginated, returns many
markets = await kalshi.markets.list(limit=100)

# get() - single item
market = await kalshi.markets.get("TICKER")
```

**Consistent across all resources** - markets, events, series, trades.

## What to Learn

**Steal These Patterns**:

1. **Async-first architecture** - True async/await for concurrency
2. **API structure mirroring** - Match official API hierarchy
3. **Pydantic models** - Type-safe responses with autocomplete
4. **list() vs get() convention** - Clear naming for pagination
5. **httpx AsyncClient** - Modern async HTTP library

**Avoid**:

1. **No authentication** - Need private endpoints for trading
2. **Breaking changes** - Stabilize API before 1.0
3. **No error handling** - Add custom exceptions and retries
4. **No rate limiting** - Track and respect limits

**Integration Lessons**:

- **Async unlocks concurrency** - Fetch 100 markets in seconds
- **Type hints improve DX** - Autocomplete makes APIs usable
- **Mirror API structure** - Don't invent new abstractions
- **httpx is the right choice** - Better than aiohttp for REST

## Comparison

**aiokalshi**:

- Async-native
- Kalshi only
- Public endpoints only
- Type-safe with Pydantic

**py-clob-client** (Polymarket):

- Synchronous
- Private + public endpoints
- Type-safe with dataclasses
- Official SDK

**predmarket**:

- Async-native
- Kalshi + Polymarket
- Unified interface
- Less comprehensive per platform

## Verdict

**The best async Kalshi client for read-only use cases**. If you're building async bots
that need market data, this is the cleanest choice.

Missing authentication means you can't trade yet. But for market monitoring,
backtesting, and data analysis, this is excellent.

**Rating**: ⭐⭐⭐ (3/5) - Would be 4/5 with authentication **Maturity**: Alpha (rapid
development, breaking changes expected) **Code Quality**: High (clean async, type-safe)
**Completeness**: Medium (public endpoints only) **Relevance**: High (async + type-safe
is the right architecture)
