# Polymarket

Largest prediction market by volume. Crypto-based on Polygon. US users technically
restricted (requires VPN).

## Authentication

### Two-Tier System

**Level 1 (L1)**: Private key EIP-712 signing - proves wallet ownership **Level 2
(L2)**: HMAC-SHA256 API key - faster request signing

**Read-only access requires no authentication** - market data, prices, order books are
public.

### L1: Wallet Signature

EIP-712 domain:

- name: "ClobAuthDomain"
- version: "1"
- chainId: 137 (Polygon)

Used for: Creating API credentials, signing orders

### L2: API Key

Headers required:

- `POLY_ADDRESS`: Your address
- `POLY_SIGNATURE`: HMAC-SHA256 signature
- `POLY_TIMESTAMP`: Unix timestamp (expires after 30s)
- `POLY_API_KEY`: Your API key
- `POLY_PASSPHRASE`: Your passphrase

Used for: Order management, balance checks, history

### Signature Types

| Wallet Type              | `signature_type` | Description         |
| ------------------------ | ---------------- | ------------------- |
| EOA (MetaMask, hardware) | 0                | Direct private key  |
| Email/Magic.Link         | 1                | Delegated via proxy |
| Browser Wallet Proxy     | 2                | Gnosis Safe proxy   |

### The Funder Address

Polymarket deploys a **proxy wallet** for each user. Your funds live there, not in your
signing wallet.

```
EOA (signing key) --controls--> Proxy Wallet (holds USDC + positions)
```

Find your proxy address below your profile picture on Polymarket.

### Client Setup

```python
from py_clob_client.client import ClobClient

# For email/Magic wallet users
client = ClobClient(
    "https://clob.polymarket.com",
    key="<private-key-from-magic-export>",
    chain_id=137,
    signature_type=1,  # POLY_PROXY
    funder="<proxy-address-from-profile>"
)
client.set_api_creds(client.create_or_derive_api_creds())
```

### Getting Your Private Key

**Email/Magic users**: Export at https://reveal.magic.link/polymarket

**MetaMask users**: Export from wallet settings

### Multi-User Setup (Family Trading)

**No delegation system exists.** Options:

1. **Share private key** (high risk)
   - Family member exports key, gives to bot
   - Full wallet access (can withdraw everything)
   - Trust-based security only

2. **Dedicated trading wallet** (recommended)
   - Create separate Polymarket account just for bot
   - Fund only what you're willing to risk
   - Main account stays isolated

3. **Turnkey integration** (enterprise)
   - Keys in secure enclaves, never exposed
   - Bot requests signatures via API
   - Full audit trail

### Allowances (EOA/MetaMask only)

Before trading with EOA wallets, approve exchange contracts:

- USDC approval
- Conditional token approval

Email/Magic users get automatic allowances.

## SDK

Official: `py-clob-client` (sync-only, some reliability issues) Better for
multi-exchange: `dr-manhattan` (CCXT-style, WebSocket support)

See [libraries/](../libraries/) for detailed analysis.

## Key Facts

- **Chain**: Polygon (chain ID 137)
- **Currency**: USDC.e
- **Settlement**: On-chain, non-custodial
- **Fee**: ~2% on trades
- **US Legal**: No (requires VPN)
- **Volume**: Highest of any prediction market
- **Leaderboard**: Public, good for copy trading strategy
