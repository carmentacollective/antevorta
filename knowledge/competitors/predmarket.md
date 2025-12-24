# predmarket - Unified SDK for Kalshi and Polymarket

**Repository**: [ashercn97/predmarket](https://github.com/ashercn97/predmarket)
**Status**: Active (rapid development) **Platforms**: Kalshi, Polymarket **Type**:
Unified async client library **Local Clone**: `../reference/predmarket`

## Overview

Async-native Python library providing unified interface to both Kalshi and Polymarket
prediction markets. Aims to abstract differences while preserving full power of
individual APIs.

**From the same team**: [The Odds Company](https://github.com/orgs/the-odds-company) -
includes aiokalshi and other prediction market tools.

**Key Features**:

- Single install for both Kalshi and Polymarket
- Asyncio-native design
- Identical API for both platforms
- WebSocket support (Polymarket working, Kalshi in development)
- Pydantic models for type safety

## Architecture: Platform Unification

**Core Design** - Same interface, different platforms:

```python
from predmarket import KalshiRest, PolymarketRest
from httpx import AsyncClient

async def main():
    async with AsyncClient() as client:
        # Initialize both with identical API
        kalshi = KalshiRest(client)
        polymarket = PolymarketRest(client)

        # Fetch questions (events) - same method name
        kalshi_questions = await kalshi.fetch_questions()
        polymarket_questions = await polymarket.fetch_questions(limit=10, asc=True)

        # Fetch contracts (markets) - same method name
        kalshi_contracts = await kalshi.fetch_contracts()
        polymarket_contracts = await polymarket.fetch_contracts()
```

**Unified Terminology**:

Instead of learning platform-specific terms:

- Kalshi: Events → Markets → Outcomes
- Polymarket: Markets → Tokens → Outcomes

**predmarket uses**:

- Questions (high-level question being answered)
- Contracts (individual market for specific outcome)

```python
# predmarket terminology
questions = await kalshi.fetch_questions()  # High-level questions
contracts = await kalshi.fetch_contracts()  # Specific tradeable markets
```

## REST API Coverage

### Kalshi REST

**Fetch Questions (Events)**:

```python
kalshi = KalshiRest(client)

questions = await kalshi.fetch_questions(
    limit=100,
    cursor=None,
    status="open"
)

for question in questions.questions:
    print(f"{question.ticker}: {question.title}")
```

**Fetch Contracts (Markets)**:

```python
contracts = await kalshi.fetch_contracts(
    limit=100,
    event_ticker="EVENT-TICKER",
    status="open"
)

for contract in contracts.contracts:
    print(f"{contract.ticker}: {contract.yes_bid} / {contract.yes_ask}")
```

### Polymarket REST

**Fetch Questions (Markets)**:

```python
polymarket = PolymarketRest(client)

questions = await polymarket.fetch_questions(
    limit=10,
    asc=True,  # Polymarket-specific param
    offset=0
)

for question in questions.questions:
    print(f"{question.id}: {question.question}")
```

**Fetch Contracts (Tokens)**:

```python
contracts = await polymarket.fetch_contracts(
    limit=100,
    # Polymarket-specific filtering
)

for contract in contracts.contracts:
    print(f"{contract.token_id}: {contract.outcome}")
```

## WebSocket Support

**Polymarket WebSocket** (working):

```python
from predmarket import PolymarketWS

async def stream_polymarket():
    async with PolymarketWS.connect() as socket:
        polymarket = PolymarketWS(socket)

        # Stream orderbook updates for specific markets
        async for row in polymarket.stream(["TOKEN-ID-1", "TOKEN-ID-2"]):
            # row is a Pydantic model with autocomplete
            print(f"Price update: {row.price}")
```

**Kalshi WebSocket** (in development):

```python
# Planned API (not yet implemented)
from predmarket import KalshiWS

async def stream_kalshi():
    async with KalshiWS.connect() as socket:
        kalshi = KalshiWS(socket)
        async for update in kalshi.stream(["MARKET-TICKER"]):
            print(update)
```

## API Design Philosophy

**Goal**: "Abstract enough to be intuitive, but not lose ANY power of the individual
APIs."

**Example - Platform-Specific Parameters**:

```python
# Polymarket has 'asc' and 'offset' params
poly_questions = await polymarket.fetch_questions(
    limit=10,
    asc=True,     # Polymarket-specific
    offset=20     # Polymarket-specific
)

# Kalshi has 'cursor' and 'event_ticker' params
kalshi_contracts = await kalshi.fetch_contracts(
    limit=100,
    cursor="abc123",         # Kalshi-specific
    event_ticker="EVENT-XYZ"  # Kalshi-specific
)
```

**Unified method names, platform-specific parameters preserved.**

## Type Safety

**Pydantic Models**:

```python
from pydantic import BaseModel

class Question(BaseModel):
    """Unified question model"""
    id: str
    ticker: Optional[str]  # Kalshi has tickers
    title: str
    question: str
    category: Optional[str]
    # ... platform-agnostic fields

class Contract(BaseModel):
    """Unified contract model"""
    id: str
    token_id: Optional[str]  # Polymarket uses token IDs
    ticker: Optional[str]    # Kalshi uses tickers
    outcome: str
    yes_bid: Optional[float]
    yes_ask: Optional[float]
    # ... platform-agnostic fields
```

**Autocomplete works**:

```python
contracts = await kalshi.fetch_contracts()

for contract in contracts.contracts:
    # Editor autocompletes: ticker, outcome, yes_bid, yes_ask, etc.
    print(contract.yes_bid)
```

## Code Quality

### Strengths

- ✅ **Async-native** - True asyncio design
- ✅ **Unified interface** - Same API for both platforms
- ✅ **Type-safe** - Pydantic models throughout
- ✅ **Platform parity** - Platform-specific params preserved
- ✅ **WebSocket support** - Real-time data (Polymarket working)
- ✅ **httpx-based** - Modern async HTTP

### Weaknesses

- ❌ **Rapid development** - Breaking changes expected
- ❌ **Incomplete Kalshi WS** - Still in development
- ❌ **Minimal documentation** - README only
- ❌ **No authentication** - Public endpoints only
- ❌ **No error handling** - Raw exceptions propagate
- ❌ **Less comprehensive** - Missing some advanced features per platform

## Unique Innovations

### 1. Dual-Platform Unification 🔥

**Novel aspect**: Single library for both major prediction markets.

Most libraries are platform-specific:

- py-clob-client (Polymarket only)
- aiokalshi (Kalshi only)

**predmarket unifies both**:

```python
# Same interface for both
kalshi_data = await kalshi.fetch_questions()
poly_data = await polymarket.fetch_questions()
```

Write once, trade anywhere.

### 2. Preserved Platform Power 🔥

**Novel aspect**: Abstraction doesn't sacrifice platform-specific features.

```python
# Use Polymarket's specific sorting
poly = await polymarket.fetch_questions(asc=True, offset=20)

# Use Kalshi's specific filtering
kalshi = await kalshi.fetch_contracts(event_ticker="EVENT-XYZ")
```

**Not a lowest-common-denominator API** - you get full platform capabilities.

### 3. Unified Terminology (Questions + Contracts) 🔥

**Novel aspect**: Common language across platforms.

| Platform   | High-Level | Tradeable | predmarket          |
| ---------- | ---------- | --------- | ------------------- |
| Kalshi     | Event      | Market    | Question → Contract |
| Polymarket | Market     | Token     | Question → Contract |

**Clearer mental model** - "Questions have Contracts you can trade."

### 4. WebSocket + REST in One Library 🔥

**Novel aspect**: Real-time and request/response in single import.

```python
from predmarket import PolymarketRest, PolymarketWS

# Fetch initial data
async with AsyncClient() as client:
    rest = PolymarketRest(client)
    contracts = await rest.fetch_contracts()

# Stream updates
async with PolymarketWS.connect() as socket:
    ws = PolymarketWS(socket)
    async for update in ws.stream(contract_ids):
        print(update)
```

**Single library for complete integration**.

## What to Learn

**Steal These Patterns**:

1. **Dual-platform support** - Single interface for multiple markets
2. **Unified terminology** - Abstract without losing platform specificity
3. **Platform-specific params** - Don't force lowest-common-denominator
4. **REST + WebSocket together** - Complete API coverage in one library
5. **Pydantic for unification** - Type-safe models work across platforms

**Avoid**:

1. **Incomplete features** - Finish Kalshi WebSocket before 1.0
2. **Missing authentication** - Need private endpoints for trading
3. **Rapid breaking changes** - Stabilize API
4. **Sparse docs** - Need comprehensive documentation

**Integration Lessons**:

- **Unification is valuable** - Write once, deploy to multiple markets
- **Don't sacrifice power** - Platform-specific features matter
- **Async + WebSocket** - Essential for prediction market bots
- **Type safety across platforms** - Pydantic enables unified models

## Comparison

**predmarket**:

- Dual-platform (Kalshi + Polymarket)
- Async-native
- Unified interface
- Public endpoints only

**dr-manhattan**:

- Multi-platform (Polymarket, Opinion, Limitless)
- CCXT-style abstractions
- Strategy base class
- More mature

**aiokalshi** + **py-clob-client**:

- Platform-specific
- More comprehensive per platform
- Official (py-clob-client)
- Can't share code across platforms

## Use Cases

**Best For**:

1. **Cross-platform arbitrage** - Same code monitors both Kalshi and Polymarket
2. **Multi-market bots** - Deploy identical strategy to both platforms
3. **Data analysis** - Compare markets across platforms
4. **Async workflows** - Non-blocking concurrent requests

**Not Ideal For**:

1. **Production trading** - Missing authentication and advanced features
2. **Platform-specific optimization** - Use official SDKs for deepest integration

## Verdict

**The right architecture for cross-platform prediction market bots**. If you want to
trade on both Kalshi and Polymarket with a single codebase, this is the cleanest path.

Still early - missing authentication, incomplete WebSocket support, and sparse docs. But
the core abstraction is sound and the async design is correct.

**Rating**: ⭐⭐⭐ (3/5) - Would be 4/5 with authentication and complete WebSocket
support **Maturity**: Alpha (rapid development) **Code Quality**: High (clean async,
type-safe) **Innovation**: Very High (dual-platform unification) **Relevance**: Very
High (essential for multi-platform strategies)
