# Kalshi

US-regulated prediction market exchange. CFTC-regulated, legal for US users without VPN.

## Consumer Onboarding

Non-technical users can sign up without any crypto knowledge.

### Signup Flow

1. **Create account**: Email + password (or Apple ID / Google sign-in)
2. **Verify identity (KYC)**: Upload US government ID + SSN (often instant, up to 48h
   max)
3. **Fund account**: US debit card (2% fee), bank transfer, or wire
4. **Trade**: App or web interface works immediately

### From Consumer to API Access

The consumer login (email/password) is separate from API authentication. Any verified
user can generate API keys:

1. Log into Kalshi (web is easier than app for this)
2. Go to **Profile Settings** → `kalshi.com/account/profile`
3. Find **"API Keys"** section
4. Click **"Create New API Key"**
5. **Download the `.txt` file** - this is the RSA private key
6. Provide to Antevorta (we store encrypted in our database)

Kalshi generates the RSA keypair. The user just clicks and downloads - no crypto
knowledge required. They provide the key once, we store it securely, and the bot trades
on their behalf.

## Authentication

### Method: RSA-PSS Signing

Every request requires three headers:

| Header                    | Value                     |
| ------------------------- | ------------------------- |
| `KALSHI-ACCESS-KEY`       | Your Key ID               |
| `KALSHI-ACCESS-TIMESTAMP` | Timestamp in milliseconds |
| `KALSHI-ACCESS-SIGNATURE` | RSA-PSS SHA256 signature  |

### Signature Generation

**Message format**: `{timestamp_ms}{HTTP_METHOD}{path}` (concatenated, no separators)

**Algorithm**: RSA-PSS with SHA256, salt length = digest length, base64-encoded output

**Gotcha**: Sign path _without_ query params. `/trade-api/v2/portfolio/orders?limit=5`
signs as `/trade-api/v2/portfolio/orders`.

### Getting API Keys

1. Create account at kalshi.com
2. Complete KYC (US ID + SSN + address verification)
3. Profile Settings → API Keys → Create New API Key
4. **Save the private key immediately** - cannot be retrieved later

Two options:

- **Kalshi-generated**: They create keypair, you download private key
- **User-provided**: Upload your own public key (Premier tier only)

### Multi-User Setup (Family Trading)

**No delegation system exists.** Each person must:

1. Have their own verified Kalshi account (individual KYC)
2. Generate their own API keys
3. Share private key with the bot

**Security implication**: API keys have full trading access. No read-only option.

### Rate Limits

| Tier     | Read/sec | Write/sec | How to Get                                        |
| -------- | -------- | --------- | ------------------------------------------------- |
| Basic    | 20       | 10        | Automatic                                         |
| Advanced | 30       | 30        | [Apply](https://kalshi.typeform.com/advanced-api) |
| Premier  | 100      | 100       | 3.75% monthly volume                              |
| Prime    | 400      | 400       | 7.5% monthly volume                               |

Write-limited operations: CreateOrder, CancelOrder, AmendOrder, DecreaseOrder, batch
variants.

### Environments

| Environment | Base URL                                        |
| ----------- | ----------------------------------------------- |
| Production  | `https://api.elections.kalshi.com/trade-api/v2` |
| Demo        | `https://demo-api.kalshi.co/trade-api/v2`       |

Generate separate keys at [demo.kalshi.co](https://demo.kalshi.co) for testing.

## SDK

Official SDK: `kalshi-python` (auto-generated from Swagger) Better alternative:
`kalshi-py` - modern async support, OpenAPI-generated

See [libraries/prediction-market-data.md](../libraries/prediction-market-data.md) for
details.

## Key Facts

- **Chain**: None (centralized exchange)
- **Currency**: USD (real money, bank transfers)
- **Settlement**: Event-based contracts
- **Fee**: 7% of winnings (varies by market)
- **Min deposit**: $5
- **US Legal**: Yes, CFTC regulated
