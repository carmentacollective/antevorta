# Antevorta

AI-driven trading bot infrastructure for prediction markets.

> _Antevorta: Roman goddess of the future, who could see what was to come._
>
> _Also: **ante** — the bet you place before the cards are dealt._

## Vision

Build intelligent bots that exploit market inefficiencies in prediction markets
(Polymarket, Kalshi), targeting $100K in earnings by January 2026.

## Strategy

Three complementary approaches running in parallel:

### 🎯 Volatility Trading

Exploits low-volume underdogs in sports markets by buying at price lows and selling at
highs using limit orders (no-fee maker trades).

### 👥 Copy Trading

Mirrors trades from top Polymarket performers in near real-time with ~4 second latency.

### ⚖️ Arbitrage Scanner

Identifies cross-platform odds mismatches between Polymarket and Kalshi (2-5%
inefficiencies).

## Architecture

- **LLM Agents** - Claude Opus 4.5, Gemini, GPT as autonomous decision-makers
- **MCP Servers** - Platform integrations for Polymarket and Kalshi APIs
- **Bot Infrastructure** - Execution layer for automated trading

Philosophy: Give goals, not steps. Let the LLM work.

## Team

| Role                | Person         |
| ------------------- | -------------- |
| Architecture & AI   | Nick Sullivan  |
| Strategy Validation | Kenny Sullivan |
| Bot Development     | Torin          |
| Risk Management     | Conor Sullivan |
| Coordination        | Anna Sullivan  |

## Status

🌱 Foundational stage - architecture and strategy defined, implementation beginning.

## License

Private - All rights reserved.
