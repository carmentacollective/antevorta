# Platforms

Prediction market exchanges we integrate with.

| Platform                      | Auth Method       | US Legal | Currency | Multi-User                           |
| ----------------------------- | ----------------- | -------- | -------- | ------------------------------------ |
| [Kalshi](./kalshi.md)         | RSA-PSS signing   | Yes      | USD      | Each needs own account + keys        |
| [Polymarket](./polymarket.md) | EIP-712 + API key | No (VPN) | USDC.e   | Share private key or separate wallet |

## Authentication Reality Check

**Neither platform has OAuth or delegation.** For family members to let a bot trade on
their behalf:

- **Kalshi**: Each person creates account, completes KYC, generates API keys, shares
  private RSA key with bot
- **Polymarket**: Each person exports their Magic wallet private key OR creates a
  separate trading wallet

**Security implication**: Whoever has the keys can do anything - place trades, withdraw
funds. No permission scoping exists.

## Recommendation for Family Setup

Create **dedicated trading wallets** per family member with only the capital they want
the bot to manage. Keep main holdings elsewhere. This limits blast radius if something
goes wrong.
