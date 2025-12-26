# kalshi-python-unofficial

Lightweight Python wrapper for Kalshi API with WebSocket support.

## Quick Facts

| Attribute  | Value                                                                                 |
| ---------- | ------------------------------------------------------------------------------------- |
| **Repo**   | [humz2k/kalshi-python-unofficial](https://github.com/humz2k/kalshi-python-unofficial) |
| **PyPI**   | Not published (install from GitHub)                                                   |
| **Python** | 3.8+                                                                                  |
| **Lines**  | 526 (entire library)                                                                  |
| **Status** | Active, last update Jan 2025                                                          |

## Why This Library

1. **WebSocket support** - Real-time orderbook and ticker streams
2. **Extremely lightweight** - 526 lines total, easy to audit
3. **Simple API** - Clean namespace pattern (`client.markets.GetMarket()`)
4. **Full coverage** - All endpoints as of Jan 2025

## Code Quality: 3/5

- Simple, readable code
- No type hints (weakness)
- Direct requests wrapper
- Basic error handling
- Clean namespace organization

## API Coverage

Full REST API coverage:

```python
client.exchange.GetExchangeStatus()
client.markets.GetMarkets()
client.markets.GetMarket(ticker)
client.markets.GetOrderbook(ticker)
client.markets.GetTrades(ticker)
client.portfolio.GetBalance()
client.portfolio.GetPositions()
client.portfolio.GetOrders()
client.portfolio.CreateOrder(...)
client.portfolio.CancelOrder(order_id)
```

## WebSocket Support

The killer feature - real-time streaming:

```python
from kalshi_unofficial import KalshiWebSocketClient

ws_client = KalshiWebSocketClient(
    key_id="your-key-id",
    private_key=private_key,
    environment="prod"  # or "demo"
)

# Subscribe to orderbook updates
await ws_client.subscribe_to_orderbook(["TICKER-1", "TICKER-2"])

# Handle messages
async for message in ws_client:
    print(message)  # Real-time orderbook deltas
```

## Installation

```bash
pip install git+https://github.com/humz2k/kalshi-python-unofficial.git
```

Or clone and install locally:

```bash
git clone https://github.com/humz2k/kalshi-python-unofficial.git
cd kalshi-python-unofficial
pip install -e .
```

## Usage Pattern

```python
from kalshi_unofficial import KalshiClient
from cryptography.hazmat.primitives import serialization

# Load private key
with open("private_key.pem", "rb") as f:
    private_key = serialization.load_pem_private_key(f.read(), password=None)

# Initialize client
client = KalshiClient(
    key_id="your-key-id",
    private_key=private_key,
    environment="demo"  # or "prod"
)

# Get balance
balance = client.portfolio.GetBalance()

# Get orderbook
orderbook = client.markets.GetOrderbook("SPORTS-NBA-LAKERS")

# Place order
order = client.portfolio.CreateOrder(
    ticker="SPORTS-NBA-LAKERS",
    side="yes",
    count=10,
    type="limit",
    yes_price=50
)
```

## Dependencies

Minimal:

- `requests` - HTTP client
- `websockets` - WebSocket client
- `cryptography` - RSA-PSS signing

## Limitations

- **No type hints** - IDE won't help you
- **No async HTTP** - Only WebSocket is async
- **No tests** - Use with caution
- **Not on PyPI** - Must install from GitHub

## For Underdog Volatility Bot

Use this specifically for:

- Real-time orderbook streaming via WebSocket
- Detecting price oscillations as they happen
- Low-latency opportunity detection

Pair with `kalshi-py` for typed, async trading operations.

## Verdict

**Use for WebSocket only.** The real-time streaming is essential for volatility
detection, but use `kalshi-py` for actual trading operations where type safety matters.
