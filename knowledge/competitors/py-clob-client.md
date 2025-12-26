# py-clob-client - Official Polymarket Python SDK

**Repository**:
[Polymarket/py-clob-client](https://github.com/Polymarket/py-clob-client) **Status**:
Active (official Polymarket SDK) **Platform**: Polymarket CLOB **Type**: Official API
client library **Local Clone**: `../reference/py-clob-client`

## Overview

Official Python client for Polymarket's Central Limit Order Book (CLOB). Provides
comprehensive access to market data, order placement, and account management. Maintained
by Polymarket team.

**Key Features**:

- Full CLOB API coverage (public + private endpoints)
- Support for multiple wallet types (EOA, proxy, Magic)
- Market and limit order execution
- Real-time orderbook data
- Type-safe order arguments
- Built-in signature handling

## Architecture

**Core Client Pattern**:

```python
class ClobClient:
    def __init__(self, host, key=None, chain_id=None, signature_type=None, funder=None):
        self.host = host
        self.key = key
        self.chain_id = chain_id  # 137 for Polygon
        self.signature_type = signature_type  # 0=EOA, 1=Magic, 2=Proxy
        self.funder = funder  # Address that holds funds
```

**Wallet Support**:

1. **EOA (Externally Owned Account)** - MetaMask, hardware wallets:

   ```python
   client = ClobClient(HOST, key=PRIVATE_KEY, chain_id=137, signature_type=0)
   ```

2. **Magic Wallet** - Email-based wallets:

   ```python
   client = ClobClient(HOST, key=PRIVATE_KEY, chain_id=137, signature_type=1, funder=PROXY_ADDRESS)
   ```

3. **Proxy Wallets** - Smart contract wallets:
   ```python
   client = ClobClient(HOST, key=PRIVATE_KEY, chain_id=137, signature_type=2, funder=PROXY_ADDRESS)
   ```

**API Credential Derivation**:

```python
# Derive API credentials from private key
api_creds = client.create_or_derive_api_creds()
client.set_api_creds(api_creds)
```

## API Coverage

### Public Endpoints (No Auth Required)

**Server Status**:

```python
ok = client.get_ok()  # Health check
time = client.get_server_time()  # Server timestamp
```

**Market Data**:

```python
# Pricing
mid = client.get_midpoint(token_id)
price = client.get_price(token_id, side="BUY")

# Orderbook
book = client.get_order_book(token_id)
books = client.get_order_books([BookParams(token_id=token_id)])

# Market info
markets = client.get_markets()  # List all markets
market = client.get_market(condition_id)
```

### Private Endpoints (Requires Auth)

**Order Placement**:

Market Order:

```python
from py_clob_client.clob_types import MarketOrderArgs, OrderType
from py_clob_client.order_builder.constants import BUY, SELL

mo = MarketOrderArgs(
    token_id="token_123",
    amount=25.0,  # Dollar amount
    side=BUY,
    order_type=OrderType.FOK  # Fill-or-Kill
)
signed = client.create_market_order(mo)
resp = client.post_order(signed, OrderType.FOK)
```

Limit Order:

```python
from py_clob_client.clob_types import OrderArgs, OrderType

order = OrderArgs(
    price=0.65,  # Price in decimal (0.65 = 65 cents)
    size=100.0,  # Number of shares
    side=BUY,
    token_id="token_123"
)
signed = client.create_order(order)
resp = client.post_order(signed, OrderType.GTC)  # Good-Till-Canceled
```

**Order Management**:

```python
# Cancel order
client.cancel(order_id)

# Cancel all orders
client.cancel_all()

# Cancel orders for specific market
client.cancel_market_orders(market_id)
```

**Account Information**:

```python
# Get positions
positions = client.get_positions()

# Get balance
balance = client.get_balance()

# Get order history
orders = client.get_orders()
```

## Order Types

**Supported Order Types**:

1. **GTC (Good-Till-Canceled)** - Remains active until filled or canceled
2. **FOK (Fill-or-Kill)** - Execute immediately and completely or cancel
3. **GTD (Good-Till-Date)** - Active until specified expiration

**Order Side**:

- `BUY` - Purchase shares
- `SELL` - Sell shares

## Authentication

**Signature Types**:

```python
# signature_type=0: EOA (MetaMask, Ledger, Trezor)
# - Standard Ethereum signatures
# - Requires token allowances set
# - Direct private key control

# signature_type=1: Magic Wallet (email-based)
# - Delegated signing
# - Proxy contract holds funds
# - Requires funder address

# signature_type=2: Proxy/Smart Wallet
# - Contract-based signatures
# - Multi-sig support
# - Requires funder address
```

## Token Allowances (Critical for EOA Users)

**IMPORTANT**: MetaMask and hardware wallet users must set token allowances before
trading:

```python
# Before placing first order, approve CLOB contract to spend USDC
# This is a one-time setup per wallet

# Allowances must be set for:
# 1. Collateral token (USDC)
# 2. Conditional token framework

# This requires direct blockchain interaction (not via py-clob-client)
# Use web3.py or ethers.js to set allowances
```

## Code Quality

### Strengths

- ✅ **Official library** - Maintained by Polymarket
- ✅ **Type safety** - Comprehensive type hints via dataclasses
- ✅ **Clear examples** - Copy-pasteable code in README
- ✅ **Multiple wallet support** - EOA, Magic, Proxy
- ✅ **Comprehensive API coverage** - All CLOB endpoints
- ✅ **Active development** - Regular updates

### Weaknesses

- ❌ **No async support** - Synchronous only (blocks on I/O)
- ❌ **Limited error handling** - Basic exceptions, no retries
- ❌ **No WebSocket client** - REST only (polling required for real-time data)
- ❌ **Sparse documentation** - README is good, but no API docs site
- ❌ **No rate limiting helpers** - Manual rate limit management required

## Usage Patterns

### Read-Only Market Data (No Auth)

```python
from py_clob_client.client import ClobClient

client = ClobClient("https://clob.polymarket.com")

# Get all markets
markets = client.get_markets()

for market in markets[:10]:
    token_id = market['tokens'][0]['token_id']
    mid = client.get_midpoint(token_id)
    print(f"{market['question']}: {mid}")
```

### Automated Trading (With Auth)

```python
from py_clob_client.client import ClobClient
from py_clob_client.clob_types import OrderArgs, OrderType
from py_clob_client.order_builder.constants import BUY

# Initialize
client = ClobClient(
    "https://clob.polymarket.com",
    key=os.getenv("PRIVATE_KEY"),
    chain_id=137,
    signature_type=1,
    funder=os.getenv("FUNDER_ADDRESS")
)
client.set_api_creds(client.create_or_derive_api_creds())

# Place order
order = OrderArgs(price=0.60, size=100, side=BUY, token_id="token_123")
signed = client.create_order(order)
resp = client.post_order(signed, OrderType.GTC)
print(f"Order placed: {resp['id']}")
```

## What to Learn

**Steal These Patterns**:

1. **Multiple signature types** - Support different wallet architectures
2. **API credential derivation** - Derive long-lived creds from private key
3. **Type-safe order args** - Use dataclasses for order parameters
4. **Funder separation** - Distinguish signing key from fund-holding address
5. **Order type enum** - Clean abstraction for GTC/FOK/GTD

**Avoid**:

1. **Synchronous I/O** - Use async for production trading bots
2. **No retry logic** - Add exponential backoff for resilience
3. **Missing rate limiting** - Track and respect rate limits
4. **No WebSocket** - Polling is inefficient for real-time data

**Integration Lessons**:

- **Official SDKs simplify authentication** - Don't roll your own CLOB signing
- **Wallet diversity matters** - Support EOA and proxy wallets
- **Token allowances are critical** - Document clearly for EOA users
- **Type hints improve DX** - Dataclasses make API usage discoverable

## Comparison with Alternatives

**py-clob-client** (official):

- Comprehensive API coverage
- Well-tested and maintained
- Synchronous only
- Polymarket-specific

**dr-manhattan**:

- Unified interface (Polymarket + others)
- Exchange-agnostic code
- Third-party (not official)
- Less comprehensive per platform

**predmarket**:

- Async-native
- Polymarket + Kalshi unified
- Newer, less mature
- Missing some advanced features

## Verdict

**The official choice for Polymarket integration**. If you're building on Polymarket,
this is the reference implementation. Comprehensive, type-safe, and actively maintained.

Limitations are sync-only architecture and missing real-time WebSocket support. For
async workflows or multi-exchange support, consider dr-manhattan or predmarket.

**Rating**: ⭐⭐⭐⭐ (4/5) **Maturity**: Production (official SDK) **Code Quality**:
High (official, type-safe) **Completeness**: Very High (full CLOB coverage)
**Relevance**: Very High (essential for Polymarket integration)
