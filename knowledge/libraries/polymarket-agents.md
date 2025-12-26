# Polymarket Agents Framework

Official Polymarket framework for AI-powered autonomous trading.

## Quick Facts

| Attribute    | Value                                                     |
| ------------ | --------------------------------------------------------- |
| **Repo**     | [Polymarket/agents](https://github.com/Polymarket/agents) |
| **Stars**    | 975                                                       |
| **Language** | Python (with some TypeScript)                             |
| **Purpose**  | LLM-driven trading agents                                 |
| **Status**   | Active, official                                          |

## What It Is

A framework specifically designed for building autonomous trading agents that use LLMs
to make decisions on Polymarket. This is exactly the architecture we're targeting.

## Why It Matters

This is the closest existing implementation to what Antevorta wants to be:

- LLM as decision maker
- Autonomous operation
- Polymarket integration
- Official support from the platform

## Architecture Patterns to Study

1. **Agent Loop**: How they structure the observe → decide → act cycle
2. **LLM Integration**: How they prompt models for trading decisions
3. **Risk Controls**: What guardrails they put around autonomous trading
4. **Market Analysis**: How they feed market data to the LLM

## Verdict

**Essential reference.** Don't reinvent what they've already built. Study this
thoroughly before designing our agent architecture. May be able to use directly or fork
for our needs.

## Investigation TODO

- [ ] Clone and run locally
- [ ] Understand agent loop structure
- [ ] Identify what's reusable vs what we'd change
- [ ] Check if it supports Kalshi or is Polymarket-only

## Relevant to Our Strategies

| Strategy           | Applicability                                  |
| ------------------ | ---------------------------------------------- |
| Copy Trading       | Low - this is for independent decisions        |
| Volatility Trading | High - LLM can analyze opportunities           |
| Arbitrage          | Medium - cross-platform might need custom work |
