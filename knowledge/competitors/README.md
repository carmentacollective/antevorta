# Competitive Analysis

This directory contains competitive analysis of open source prediction market trading
bots.

## Purpose

Study existing implementations to:

- Learn proven strategies and risk management patterns
- Identify gaps and opportunities for innovation
- Understand common pitfalls and best practices
- Inform Antevorta's architecture and approach

## Reference Repositories

All analyzed repositories are cloned in `../../reference/` for direct code inspection.

## Bots Analyzed

- **[BTC-Thorp](./btc-thorp.md)** - Volatility trading on Kalshi BTC/ETH hourly markets
- **[Interest Rate Arbitrage](./interest-rate-arbitrage-trader.md)** - Fed rate
  arbitrage on Polymarket
- **[Kal-trix](./kal-trix-prediction-bot.md)** - Multi-model sentiment trading on Kalshi
- **[Kalshi GenAI](./kalshi-genai-trading-bot.md)** - Grok as autonomous trader on
  Kalshi
- **[Polymarket Copy Trading](./polymarket-copy-trading-bot.md)** - Copy trading with
  ~4-8s latency

For API clients and SDKs, see [../libraries/](../libraries/).

## Analysis Structure

Each competitive analysis includes:

- **Overview**: What the bot does
- **Strategy**: Trading logic and decision-making
- **Risk Management**: Position sizing, limits, safeguards
- **Architecture**: Tech stack, deployment, infrastructure
- **Code Quality**: Tests, documentation, maintainability
- **Unique Innovations**: Novel approaches worth learning from
- **What to Learn**: Key takeaways for Antevorta
- **Philosophical Differences**: How they differ from our approach

## How to Use

1. Read the markdown files in this directory for high-level understanding
2. Clone repos from GitHub to `../../reference/` for code inspection
3. Reference specific implementations when building similar features
4. Learn from their mistakes and successes

## Contributing

When adding new competitive analysis:

1. Clone repo to `../../reference/`
2. Create detailed markdown analysis in this directory
3. Follow existing structure for consistency
4. Include code examples and specific insights
5. Rate the bot (1-5 stars) and note maturity level
