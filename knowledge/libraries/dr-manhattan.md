# dr-manhattan - Polymarket Library

**Repository**: [github.com/guzus/dr-manhattan](https://github.com/guzus/dr-manhattan)
**Maintainer**: guzus (solo developer, 4 contributors total) **Status**: Active
development, v0.0.1 released December 7, 2025

## Overview

dr-manhattan is a CCXT-style unified API for prediction markets. While py-clob-client is
Polymarket's official single-exchange SDK, dr-manhattan abstracts across multiple
prediction market platforms (Polymarket, Limitless, Opinion) with a consistent
interface.

**Key Differentiator**: py-clob-client is a low-level Polymarket-only client.
dr-manhattan wraps py-clob-client (it's a dependency!) and adds:

- Multi-exchange abstraction with unified interface
- Strategy framework with built-in market making helpers
- WebSocket support for real-time orderbook updates
- NAV calculation, delta tracking, position management
- Exchange factory pattern for dynamic instantiation

## API Coverage

### Polymarket Features Supported

| Feature                    | py-clob-client | dr-manhattan                             |
| -------------------------- | -------------- | ---------------------------------------- |
| Order placement (GTC)      | Yes            | Yes                                      |
| Order cancellation         | Yes            | Yes                                      |
| Fetch markets              | Yes            | Yes (enhanced with sampling-markets)     |
| Fetch orderbook            | Yes            | Yes                                      |
| Price history              | Limited        | Yes (with intervals: 1m, 1h, 6h, 1d, 1w) |
| WebSocket orderbook        | No             | Yes                                      |
| WebSocket user trades      | No             | Yes                                      |
| Public trades API          | No             | Yes (data-api.polymarket.com)            |
| Market search with filters | No             | Yes (query, tags, liquidity, categories) |
| Crypto hourly markets      | No             | Yes (specialized parser)                 |
| Strategy framework         | No             | Yes                                      |

### Unique Capabilities

1. **Strategy Base Class**: Full trading strategy framework with:
   - BBO (Best Bid/Offer) market making out-of-the-box
   - Delta/NAV tracking
   - Position management with max limits
   - Graceful shutdown with position liquidation

2. **Multi-Exchange**: Same code works across Polymarket, Limitless, and Opinion:

   ```python
   polymarket = dr_manhattan.Polymarket({'private_key': key})
   limitless = dr_manhattan.Limitless({'private_key': key})
   # Same API: exchange.fetch_markets(), exchange.create_order(), etc.
   ```

3. **Exchange Factory**:

   ```python
   exchange = dr_manhattan.create_exchange('polymarket', config)
   ```

4. **Price History with DataFrames**:
   ```python
   df = exchange.fetch_price_history(market, interval="1h", as_dataframe=True)
   ```

## Code Quality

### Type Hints

**Excellent** - Full type annotations throughout:

```python
def create_order(
    self,
    market_id: str,
    outcome: str,
    side: OrderSide,
    price: float,
    size: float,
    params: Optional[Dict[str, Any]] = None,
) -> Order:
```

### Tests

**Minimal** - pytest configured but test coverage appears light. The `tests/` directory
exists but the codebase is 88% Jupyter notebooks (exploration/backtesting) and 12%
Python.

### Documentation

**Good README, no API docs** - The README is comprehensive with architecture diagrams
and usage examples. No generated API documentation or docstrings beyond inline comments.

### Error Handling

**Solid** - Custom exception hierarchy:

- `ExchangeError`, `NetworkError`, `RateLimitError`
- `AuthenticationError`, `InsufficientFunds`, `InvalidOrder`, `MarketNotFound`

Includes retry logic with exponential backoff.

### Code Style

- Uses black (line-length 100) and ruff
- Clean separation: `/base`, `/exchanges`, `/models`, `/strategies`
- Follows CCXT patterns (familiar to crypto developers)

## Comparison to py-clob-client

| Aspect               | py-clob-client           | dr-manhattan                      |
| -------------------- | ------------------------ | --------------------------------- |
| **Purpose**          | Low-level Polymarket SDK | Multi-exchange trading framework  |
| **Maintainer**       | Polymarket (official)    | Independent developer             |
| **Stability**        | Production (v0.28+)      | Early (v0.0.1)                    |
| **Scope**            | Single exchange          | 3+ exchanges                      |
| **Dependencies**     | Minimal                  | Heavy (pandas, boto3, matplotlib) |
| **Strategy Support** | None                     | Built-in framework                |
| **WebSocket**        | None                     | Full support                      |
| **Python Version**   | 3.9+                     | 3.11+                             |
| **Stars**            | ~50                      | 31                                |

### When to Use Each

**Use py-clob-client when:**

- You only trade Polymarket
- You want minimal dependencies
- You need official support
- You're building custom infrastructure

**Use dr-manhattan when:**

- You trade multiple prediction markets
- You want a strategy framework
- You need WebSocket for real-time data
- You're building market making or arbitrage bots

## Concerns

1. **Single Maintainer Risk**: guzus is the primary developer. Bus factor = 1.

2. **Heavy Dependencies**: Pulls in pandas, boto3, pyarrow, matplotlib - overkill for
   simple trading.

3. **Wraps py-clob-client**: Not a replacement - it's a layer on top. You inherit all
   py-clob-client issues plus dr-manhattan's.

4. **Early Stage**: v0.0.1 released December 2025. Limited production testing.

5. **No Kalshi Yet**: Despite being mentioned, Kalshi implementation isn't complete.

## Verdict

**Fork it** - This is exactly what we'd build ourselves for multi-exchange prediction
market trading. The architecture is clean, the abstraction is right, and the strategy
framework is genuinely useful.

However, given the single-maintainer risk and early stage:

1. Fork the repo rather than depending on PyPI
2. Strip unnecessary dependencies (boto3, matplotlib)
3. Add our own test coverage
4. Keep Polymarket and Limitless implementations, remove Opinion if not needed

The Limitless integration alone is valuable - that's a 1000+ line implementation we
don't have to write.

**Rating**: 4/5

Strong architecture, good code quality, fills a real gap. Loses a point for early-stage
maturity and heavy dependencies. Perfect candidate for forking into our own maintained
version.
