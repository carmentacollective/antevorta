# Polymarket

Largest prediction market by volume. Crypto-based on Polygon. US users technically
restricted (requires VPN). iOS app launched Dec 2025 with CFTC approval (waitlist,
sports only initially).

## Consumer Onboarding

Non-technical users can sign up without crypto knowledge - Polymarket hides the
blockchain complexity.

### Signup Flow

1. **Create account**: Email (Magic Link) or Google OAuth
2. **Magic wallet created automatically** - user never sees seed phrases
3. **Fund account**: Credit/debit via MoonPay, or Robinhood/Coinbase Connect
4. **Trade**: Web or app interface works immediately

### Funding Options

| Method        | Provider          | Fees   | Notes                         |
| ------------- | ----------------- | ------ | ----------------------------- |
| Credit/Debit  | MoonPay           | 1-4%   | Visa, Mastercard. ~$30 min    |
| Bank Transfer | MoonPay           | Lower  | Direct ACH                    |
| Robinhood     | Robinhood Connect | Varies | Fund from Robinhood balance   |
| Coinbase      | Coinbase Pay      | Varies | Seamless if you have Coinbase |

Behind the scenes: USD → USDC conversion happens automatically. User never touches
crypto directly.

### From Consumer to API Access

More complex than Kalshi because there's a proxy wallet involved.

**Prerequisites:**

1. Complete at least ONE trade via the web UI first (deploys proxy wallet)
2. Know your funder address (shown below profile picture on polymarket.com)

**Export private key:**

1. Go to **https://reveal.magic.link/polymarket**
2. Authenticate through Magic
3. Copy the private key
4. Provide to Antevorta (we store encrypted in our database)

**What we need from each user:**

- Private key (from Magic export)
- Funder address (proxy wallet address from their profile)

Both are required. The private key signs, the funder address holds funds.

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

Use `py-clob-client` with:

- **Endpoint**: `https://clob.polymarket.com`
- **Chain**: Polygon (chain_id 137)
- **Signature type**: `POLY_PROXY` (type 1) for Magic/email wallets
- **Funder**: Your proxy address (from profile page)

API credentials are derived from the private key on first use.

### Getting Your Private Key

**Email/Magic users**: Export at https://reveal.magic.link/polymarket

**MetaMask users**: Export from wallet settings

### Multi-User Setup (Family Trading)

**No delegation system exists.** Each family member:

1. Signs up for Polymarket (email is fine)
2. Funds their account
3. Makes at least one trade via the UI (deploys proxy)
4. Exports private key from Magic
5. Provides key + funder address to Antevorta

We store keys encrypted in our database and trade on their behalf.

**Security note**: The private key has full wallet access including withdrawals. Family
members should only fund what they're comfortable having the bot manage.

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
