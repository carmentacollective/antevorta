# py-clob-client - Polymarket Python SDK

**Repository**:
[github.com/Polymarket/py-clob-client](https://github.com/Polymarket/py-clob-client)
**Maintainer**: Polymarket Engineering (engineering@polymarket.com) **Status**: Actively
maintained - official SDK **Stars/Activity**: 486 stars, 179 forks, v0.34.1 released Dec
24, 2025

## Overview

The official Python client for Polymarket's Central Limit Order Book (CLOB). Enables
programmatic trading on Polymarket's prediction markets running on Polygon (chain ID
137). Designed for algorithmic traders, market makers, and anyone building trading
infrastructure.

Key capabilities:

- Read-only market data (no auth required)
- Order placement (limit, market, FOK, GTC, GTD, FAK)
- Order management (cancel single, batch, or all)
- Trade history and position tracking
- RFQ (Request for Quote) functionality
- Builder flow for advanced integrations

## API Coverage

### Fully Supported Endpoints

**Market Data (Level 0 - No Auth)**

- `get_midpoint()` / `get_midpoints()` - Mid-market prices
- `get_price()` / `get_prices()` - Best bid/ask prices
- `get_spread()` / `get_spreads()` - Bid-ask spreads
- `get_order_book()` / `get_order_books()` - Full order book depth
- `get_tick_size()` - Minimum tick size per market
- `get_simplified_markets()` / `get_markets()` - Market listings
- `get_last_trade_price()` / `get_last_trades_prices()` - Recent trades

**Authentication (Level 1)**

- `create_api_key()` / `derive_api_key()` - API credential management
- `create_or_derive_api_creds()` - Convenience method

**Trading (Level 2)**

- `create_order()` / `create_market_order()` - Order creation
- `post_order()` / `post_orders()` - Order submission
- `cancel()` / `cancel_orders()` / `cancel_all()` / `cancel_market_orders()` -
  Cancellations
- `get_orders()` / `get_order()` - Order queries
- `get_trades()` - Trade history
- `get_balance_allowance()` / `update_balance_allowance()` - Balance management
- `get_notifications()` / `drop_notifications()` - User notifications

**RFQ (Request for Quote)**

- Full RFQ client via `client.rfq` property
- Quote creation, acceptance, and management

### What's Missing

- **No async support** - Synchronous only (Issue #225 requesting this)
- **No WebSocket streaming** - Must poll for updates
- **No Gamma API integration** - Market metadata requires separate client
- **No GraphQL support** - Historical data queries limited

## Code Quality

### Type Hints

Partial coverage. Uses `dataclasses` for request/response types but many methods return
raw `dict` or `Any`. Core types defined in `clob_types.py`:

```python
@dataclass
class OrderArgs:
    token_id: str
    price: float
    size: float
    side: str
    fee_rate_bps: int = 0
    nonce: int = 0
    expiration: int = 0
    taker: str = ZERO_ADDRESS
```

### Tests

No visible test suite in the repository. Testing burden falls on integration tests
against live API.

### Documentation

Good README with clear examples covering:

- Read-only quickstart
- EOA and proxy wallet setup
- Market/limit order placement
- Order management
- Token allowance setup for MetaMask users

Missing: API reference docs, error handling guide, rate limit documentation.

### Dependencies

Minimal, well-chosen dependencies:

```python
install_requires=[
    "eth-account>=0.13.0",        # Ethereum account management
    "eth-utils>=4.1.1",           # Ethereum utilities
    "poly_eip712_structs>=0.0.1", # EIP-712 signing (Polymarket internal)
    "py-order-utils>=0.3.2",      # Order utilities (Polymarket internal)
    "python-dotenv",              # Environment variable loading
    "py-builder-signing-sdk>=0.0.2", # Builder signing (Polymarket internal)
    "httpx[http2]>=0.27.0",       # Modern HTTP client with HTTP/2
]
```

Uses `httpx` with HTTP/2 support (added v0.29.0) - good for performance.

## Usage Examples

### Read-Only Market Data

```python
from py_clob_client.client import ClobClient
from py_clob_client.clob_types import BookParams

client = ClobClient("https://clob.polymarket.com")

# Get market price
token_id = "21742633143463906290569050155826241533067272736897614950488156847949938836455"
mid = client.get_midpoint(token_id)
price = client.get_price(token_id, side="BUY")
book = client.get_order_book(token_id)

print(f"Mid: {mid}, Best Bid: {price}")
```

### Trading Setup

```python
from py_clob_client.client import ClobClient

client = ClobClient(
    "https://clob.polymarket.com",
    key="<private-key>",
    chain_id=137,
    signature_type=0,  # 0=EOA, 1=Magic/email, 2=browser proxy
    funder="<wallet-address>"
)
client.set_api_creds(client.create_or_derive_api_creds())
```

### Limit Order

```python
from py_clob_client.clob_types import OrderArgs, OrderType
from py_clob_client.order_builder.constants import BUY

order = OrderArgs(
    token_id="<token-id>",
    price=0.45,
    size=100.0,
    side=BUY
)
signed = client.create_order(order)
resp = client.post_order(signed, OrderType.GTC)
```

### Market Order

```python
from py_clob_client.clob_types import MarketOrderArgs, OrderType
from py_clob_client.order_builder.constants import BUY

mo = MarketOrderArgs(
    token_id="<token-id>",
    amount=25.0,  # $25 USD worth
    side=BUY,
    order_type=OrderType.FOK
)
signed = client.create_market_order(mo)
resp = client.post_order(signed, OrderType.FOK)
```

## Integration Effort

### Getting Started: ~2-4 hours

- Install: `pip install py-clob-client`
- Requires Python >= 3.9.10
- Need Polygon wallet with USDC
- EOA users must set token allowances manually (one-time setup)

### Known Gotchas

1. **Stale Order Book Data** (Issue #180)
   - `get_order_book()` sometimes returns "ghost market" (0.01/0.99) even for liquid
     markets
   - `get_price()` returns accurate data - use this instead for price checks

2. **Price Validation Quirks** (Issue #218)
   - API rejects 0.999 prices while frontend allows them
   - Valid range: 0.01 to 0.99

3. **Intermittent Auth Issues** (Issue #190)
   - Periodic 401 errors reported
   - May need to recreate API credentials

4. **FOK Order Decimal Precision** (Issue #121)
   - Maker amount: max 2 decimals
   - Taker amount: max 4 decimals
   - GTC orders are more forgiving

5. **No Async** - Blocking calls only, problematic for high-frequency trading

6. **14 Open Issues** as of Dec 26, 2025 - some fundamental (get_order returning
   nothing, 404s on endpoints)

### Development Velocity

Active maintenance: 10 releases in last 2 months (Oct-Dec 2025), primarily RFQ and
HTTP/2 improvements.

## Alternatives

### polymarket-apis (Community)

[github.com/qualiaenjoyer/polymarket-apis](https://github.com/qualiaenjoyer/polymarket-apis)

- Unified client: CLOB + Gamma + Data + WebSockets + GraphQL
- Pydantic v2 validation
- Better type safety
- Less battle-tested than official client

## Verdict

**Use it** - with caution.

This is the official SDK, actively maintained by Polymarket Engineering. It covers core
trading functionality and has the best chance of staying compatible with API changes.
The release cadence (10 releases in 2 months) shows active development.

However:

- **Wrap critical calls** with retry logic given the intermittent auth issues
- **Prefer `get_price()` over `get_order_book()`** for price data until Issue #180 is
  resolved
- **Consider async wrapper** if building high-frequency systems
- **Pin the version** - rapid releases mean potential breaking changes
- **Monitor GitHub issues** - several fundamental bugs remain open

For Antevorta specifically:

- Good enough for volatility trading and arbitrage scanning (our strategies)
- Copy trading's ~4s latency requirement is challenging with synchronous-only client
- Consider contributing async support or building a thin async wrapper

**Rating**: 3.5/5

Solid foundation, official support, but rough edges. The open issues around stale order
book data and auth problems are concerning for production trading. Would rate 4/5 if
async were supported and the order book reliability issues were fixed.
