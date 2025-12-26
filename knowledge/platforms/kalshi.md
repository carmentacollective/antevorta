# Kalshi

US-regulated prediction market exchange. CFTC-regulated, legal for US users without VPN.

## Authentication

### Method: RSA-PSS Signing

Every request requires three headers:

| Header                    | Value                     |
| ------------------------- | ------------------------- |
| `KALSHI-ACCESS-KEY`       | Your Key ID               |
| `KALSHI-ACCESS-TIMESTAMP` | Timestamp in milliseconds |
| `KALSHI-ACCESS-SIGNATURE` | RSA-PSS SHA256 signature  |

### Signature Generation

Sign: `timestamp_milliseconds + HTTP_METHOD + path_without_query_params`

```python
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.asymmetric import padding
import base64

def sign_request(private_key, timestamp_ms: str, method: str, path: str) -> str:
    message = f"{timestamp_ms}{method}{path}".encode('utf-8')
    signature = private_key.sign(
        message,
        padding.PSS(
            mgf=padding.MGF1(hashes.SHA256()),
            salt_length=padding.PSS.DIGEST_LENGTH
        ),
        hashes.SHA256()
    )
    return base64.b64encode(signature).decode('utf-8')
```

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
