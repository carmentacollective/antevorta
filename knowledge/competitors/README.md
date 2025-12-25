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

### Actual Trading Bots

- **[BTC-Thorp](./btc-thorp.md)** - Volatility trading on Kalshi BTC/ETH hourly markets
- More coming soon...

### Libraries (Reference)

- py-clob-client - Polymarket API client
- dr-manhattan - Another Polymarket library
- aiokalshi - Async Kalshi API client
- predmarket - Prediction market data library

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
