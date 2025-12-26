# kalshi-python

Official Kalshi Python SDK for algorithmic trading.

## Quick Facts

| Attribute   | Value                                                            |
| ----------- | ---------------------------------------------------------------- |
| **Repo**    | [Kalshi GitHub](https://github.com/Kalshi)                       |
| **PyPI**    | `kalshi-python`                                                  |
| **Version** | v2.1.4                                                           |
| **Docs**    | [docs.kalshi.com/python-sdk](https://docs.kalshi.com/python-sdk) |
| **Python**  | 3.8+                                                             |
| **Status**  | Official, maintained                                             |

## What It Does

Full API coverage for Kalshi's prediction market:

- Market discovery and event data
- Order placement and management
- Portfolio and position tracking
- Account management
- WebSocket for real-time updates

## API Coverage

Generated from OpenAPI spec, covers all endpoints:

- `ApiKeys` - API key management
- `Communications` - Notifications
- `Events` - Event discovery
- `Exchange` - Exchange status
- `Markets` - Market data and trading
- `Portfolio` - Positions and P&L
- `Series` - Market series

## WebSocket Support

Real-time streaming available:

```
Production: wss://trading-api.kalshi.com/trade-api/ws/v2
Demo: wss://demo-api.kalshi.co/trade-api/ws/v2
```

**Authentication required** via headers:

- `KALSHI-ACCESS-KEY`
- `KALSHI-ACCESS-SIGNATURE` (RSA-PSS)
- `KALSHI-ACCESS-TIMESTAMP`

## Usage Pattern

```python
from kalshi_python import ApiInstance
from kalshi_python.models import *

# Initialize
config = Configuration()
config.host = "https://trading-api.kalshi.com/trade-api/v2"
api = ApiInstance(config)

# Authenticate
api.login(email, password)

# Get markets
markets = api.get_markets(event_ticker="SPORTS-NBA")

# Place order
order = api.create_order(
    ticker=market_ticker,
    side="yes",
    count=10,
    type="limit",
    yes_price=50,  # cents
)
```

## Verdict

**Use this.** Official SDK with full API coverage. CFTC-regulated platform means stable,
documented API. Good for the underdog volatility strategy since Kalshi has strong sports
markets.

## Considerations

- US-legal (unlike Polymarket without VPN)
- Lower volume than Polymarket = more volatility opportunity
- Demo environment available for testing
- RSA key authentication is more complex than Polymarket's approach
