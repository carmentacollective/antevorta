# kalshi-py

Modern, type-safe Python client for Kalshi. **Recommended for production use.**

## Quick Facts

| Attribute  | Value                                                         |
| ---------- | ------------------------------------------------------------- |
| **Repo**   | [APTy/kalshi-py](https://github.com/APTy/kalshi-py)           |
| **Docs**   | [apty.github.io/kalshi-py](https://apty.github.io/kalshi-py/) |
| **Python** | 3.8+                                                          |
| **Async**  | Yes (both sync and async for every endpoint)                  |
| **Status** | Active, open source                                           |

## Why This Library

1. **Open source** - Full visibility into what's running your trades
2. **Async support** - Both sync and async variants for every endpoint
3. **100% type hints** - attrs-based models, full IDE autocomplete
4. **Professional error handling** - Typed error responses, proper exceptions
5. **Tests included** - 6 test files in the repo
6. **Active maintenance** - Updated December 2024

## Code Quality: 5/5

- Auto-generated from OpenAPI spec but high quality output
- `attrs` for models (lighter than Pydantic, equally typed)
- `httpx` for HTTP (modern, async-native)
- Context managers for resource cleanup
- Both detailed and parsed response variants

## API Coverage

Complete coverage for trading:

| Endpoint     | Sync                     | Async                            |
| ------------ | ------------------------ | -------------------------------- |
| Markets      | `get_markets()`          | `asyncio_get_markets()`          |
| Orderbook    | `get_market_orderbook()` | `asyncio_get_market_orderbook()` |
| Orders       | `create_order()`         | `asyncio_create_order()`         |
| Batch Orders | `batch_create_orders()`  | `asyncio_batch_create_orders()`  |
| Positions    | `get_positions()`        | `asyncio_get_positions()`        |
| Balance      | `get_balance()`          | `asyncio_get_balance()`          |
| Fills        | `get_fills()`            | `asyncio_get_fills()`            |

## Installation

```bash
pip install kalshi-py
```

## Usage Pattern

```python
from kalshi_py import AuthenticatedClient
from kalshi_py.api.portfolio import get_balance, get_positions
from kalshi_py.api.markets import get_market_orderbook, create_order

# Initialize with RSA-PSS auth
client = AuthenticatedClient(
    base_url="https://api.elections.kalshi.com/trade-api/v2",
    api_key_id="your-key-id",
    private_key_path="path/to/private_key.pem"
)

# Sync usage
with client as c:
    balance = get_balance.sync(client=c)
    positions = get_positions.sync(client=c)
    orderbook = get_market_orderbook.sync("TICKER-HERE", client=c)

# Async usage
async with client as c:
    balance = await get_balance.asyncio(client=c)
    positions = await get_positions.asyncio(client=c)
```

## Dependencies

Minimal and modern:

- `httpx` (0.23.0-0.29.0) - Async HTTP client
- `attrs` - Lightweight typed models
- `python-dateutil` - Date parsing
- `cryptography` - RSA-PSS signing

## Limitations

- **No WebSocket support** - Use `kalshi-python-unofficial` for real-time feeds
- **OpenAPI spec issues** - README notes some spec problems from Kalshi's side

## For Underdog Volatility Bot

This is the recommended library for:

- Scanning markets for opportunities
- Placing/canceling limit orders
- Tracking positions and P&L
- Account balance checks

Pair with `kalshi-python-unofficial` for WebSocket orderbook streaming.

## Verdict

**Use this.** Best combination of code quality, type safety, and async support. Open
source means you can audit every line handling your money.
