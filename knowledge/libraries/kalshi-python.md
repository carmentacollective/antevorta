# kalshi-python

Official Kalshi Python SDK for algorithmic trading. **Source code is private.**

## Quick Facts

| Attribute   | Value                                                            |
| ----------- | ---------------------------------------------------------------- |
| **PyPI**    | `kalshi-python`                                                  |
| **Version** | v2.1.4 (Sept 2025)                                               |
| **Repo**    | Private (PyPI links to non-existent `Kalshi/exchange-infra`)     |
| **Docs**    | [docs.kalshi.com/python-sdk](https://docs.kalshi.com/python-sdk) |
| **Python**  | 3.9+                                                             |
| **Status**  | Official, maintained, but closed source                          |

## Red Flag: Private Repository

The PyPI metadata points to `https://github.com/Kalshi/exchange-infra` which doesn't
exist publicly. The actual source is not available for review. For a trading system
handling real money, this is a concern:

- Can't review security implementation
- Can't see issue tracker or bug reports
- Can't contribute fixes
- Can't audit for vulnerabilities

## What It Does

Auto-generated from OpenAPI spec. Full API coverage:

- Market discovery and event data
- Order placement and management (create, amend, cancel, batch)
- Portfolio and position tracking
- Account management
- 90+ Pydantic v2 models

## Code Quality (from PyPI extraction)

**Strengths:**

- Pydantic v2 models with `StrictStr`, `StrictInt` validation
- RSA-PSS authentication implemented correctly
- Custom exception hierarchy (BadRequest, Unauthorized, Forbidden, etc.)
- Type hints present throughout

**Weaknesses:**

- Sync-only (no async support)
- Silent error swallowing in some paths
- No retry logic or timeout defaults
- No tests included
- Auto-generated code is verbose

## API Coverage

Generated from OpenAPI spec, covers all endpoints:

- `ApiKeys` - API key management
- `Events` - Event discovery
- `Exchange` - Exchange status
- `Markets` - Market data, orderbook, trades
- `Portfolio` - Positions, balance, fills
- `Orders` - Create, amend, cancel, batch operations

## WebSocket Support

Real-time streaming endpoints exist but SDK doesn't wrap them well:

```
Production: wss://api.elections.kalshi.com/trade-api/ws/v2
Demo: wss://demo-api.kalshi.co/trade-api/ws/v2
```

## Usage Pattern

```python
from kalshi_python import Configuration, ApiClient
from kalshi_python.api import MarketsApi, PortfolioApi

config = Configuration()
config.host = "https://api.elections.kalshi.com/trade-api/v2"

# Load RSA private key
with open("private_key.pem", "r") as f:
    config.private_key_pem = f.read()
config.api_key_id = "your-api-key-id"

client = ApiClient(config)
markets_api = MarketsApi(client)
portfolio_api = PortfolioApi(client)

# Get markets
markets = markets_api.get_markets(event_ticker="SPORTS-NBA")

# Get balance
balance = portfolio_api.get_balance()
```

## Verdict

**Not recommended as primary choice.** While it works, the private source code is a red
flag for production trading. Consider `kalshi-py` instead which is open source with
better code quality.

## When to Use

- Quick prototyping where you trust Kalshi completely
- If you need the absolute latest API coverage (auto-generated from their spec)
- As a reference for endpoint signatures

## Alternatives

See [kalshi-py.md](./kalshi-py.md) for the recommended open-source alternative.
