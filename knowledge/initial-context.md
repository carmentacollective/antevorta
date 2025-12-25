# Infinite Money Machine - Polymarket Trading Bots

## Context Dump from Limitless & Fireflies

---

## Project Overview

**Vision:** Build an AI-driven trading bot infrastructure for prediction markets
(Polymarket, Kalshi) that generates consistent profits through automated strategies.

**Goal:** $100,000 in earnings by January 2026

**Core Team:**

- **Nick Sullivan** - Architecture, AI/LLM integration, bot infrastructure
- **Kenny Sullivan** - Manual trading strategies, market expertise, strategy validation
- **Torin** - Bot development, trade-copying bot, data collection
- **Conor Sullivan** - Risk management, betting strategy analysis
- **Anna Sullivan** - Coordination, communication (100X WhatsApp group)
- **Russell Sullivan** - Strategy discussion participant

---

## Platforms

### Polymarket

- Currently banned in US (green light to return November 2024, not yet available without
  VPN/crypto)
- API available for bots
- Leaderboard system for tracking top traders
- Maker/taker fee structure (makers get no fees + rewards)
- Categories: Sports, Crypto, Politics

### Kalshi

- Legal in US
- Similar prediction market structure
- Good for low-volume underdogs (college basketball especially)
- Live section shows upcoming games
- Order book visibility critical

---

## Proven Strategies

### Kenny's Manual Strategy (Validated, Profitable)

**Core Concept:** Exploit volatility in low-volume underdog markets pre-game

**How it works:**

1. Focus on heavy underdogs (1-4% odds) in low-volume markets
2. College basketball is ideal - 25+ games daily with massive underdogs
3. Low volume = odds bounce significantly (e.g., 2% → 4% → 2% → 4%)
4. Buy at the lows (1-2%), sell at highs (3-4%)
5. 1% → 2% = 100% return on that position
6. Use **limit orders** exclusively (no fees as a maker)
7. Place "hopeful" orders at 1¢ that occasionally fill

**Key Insights:**

- Order book more valuable than price graphs
- Default "buy in dollars" fills at bad prices - always use limit orders
- Turn off "flip sell" setting
- Don't get too close to game start (exit with 2+ hours remaining)
- Sportsbook odds = true odds; Kalshi volatility creates opportunity

**Risk Profile:**

- Floor is 1¢ (can always sell at 1¢ pregame)
- Never hold through game start for underdogs
- Minimal losses if sold pregame

### Copy-Trading Strategy (Torin's Bot)

**Concept:** Mirror trades of top Polymarket performers in near-real-time

**Current Implementation:**

- Scans leaderboard for top 5 daily/weekly profit makers
- Buys 100 shares when target user buys
- Sells same percentage when target sells or market closes
- 4-second maximum latency to detect and copy trades
- 2% slippage tolerance built in
- Simulated results: $52 profit overnight

**Challenges:**

- API rate limits (~1 second per call)
- Slippage when copying (buying at worse price)
- Some top earners are "whales" with high losses overall
- Winners by percentage vs absolute amount distinction

### Hedging/Arbitrage (Observed in Top Traders)

**Observed Patterns:**

- "Gabagool" on Polymarket: 90% win rate on Bitcoin price prediction
- Buys both YES and NO when combined < 100¢
- Exploits odds mismatches between platforms (Polymarket vs Kalshi)
- 2-5% imbalance = profitable arbitrage opportunity
- Cross-platform scanning tool needed

---

## Technical Architecture (Proposed by Nick)

### LLM-Agent Approach

```
MCP Server ←→ LLM Agent ←→ Trading Platforms
                ↓
         Strategy Formulation
                ↓
         Bot Execution Infrastructure
```

**Key Components:**

1. **MCP Servers** connecting to Polymarket and Kalshi APIs
2. **LLM Agent** for:
   - Strategy analysis and generation
   - Market opportunity identification
   - Decision making on trades
3. **Bot Infrastructure** for execution

**Philosophy from Nick:**

> "Don't over specify steps, give goals and let it do the work. Whatever LLM is reading
> this is going to be smarter than you."

**Model Recommendations:**

- Claude Opus 4.5
- Gemini 3 (destroyed VendBench - $1K → $6K)
- ChatGPT 5.1

---

## Future Development Ideas

### Torin's ML Approach

- Train logistic regression classifier on all Polymarket data
- Predict true probability of any market
- Calculate expected value for buy/no-buy decisions
- Requires significant data collection first

### Arbitrage Scanner

- Real-time comparison across Polymarket, Kalshi, other platforms
- Alert when odds mismatch by 2%+
- Automated cross-platform hedging

### Lindy Integration (Nick's Existing Bot)

- 8-year-old margin lending bot
- $7.5M under management for Gil
- Consistently high single-digit returns
- Could apply similar LLM-agent upgrade

---

## Decisions Made

1. ✅ Use 100X WhatsApp group for coordination (trading-bots thread)
2. ✅ Kenny continues manual validation to prove strategy viability
3. ✅ Torin continues copy-bot refinement
4. ✅ Nick to build LLM-agent infrastructure
5. ✅ Limit orders only (maker = no fees)
6. ✅ Focus on college basketball for Kenny's strategy
7. ✅ Target 14+ days of profitable manual trading before scaling

---

## Key Metrics to Track

- Win rate percentage
- Profit per trade
- Daily/weekly profit
- Number of resting orders
- Order fill rate
- Slippage on copy trades
- Platform fees paid

---

## Action Items (From Fireflies Meeting)

**Nick:**

- [ ] Develop bot infrastructure with LLM as agent
- [ ] Set up MCP server connections to platforms

**Kenny:**

- [ ] Continue manual trading to validate 14-day profitability
- [ ] Train Anna & Clara on Kalshi navigation

**Torin:**

- [ ] Continue refining copy-bot (slippage, user filtering)
- [ ] Data collection for ML classifier
- [ ] Run copy-bot for 1 week validation

**Conor:**

- [ ] Support risk management analysis
- [ ] Kelly criterion application research

---

## Knowledge Folder Structure (Recommended)

```
knowledge/
├── overview.md              # This file
├── platforms/
│   ├── polymarket.md        # API, fees, features
│   └── kalshi.md            # API, fees, features
├── strategies/
│   ├── volatility-trading.md   # Kenny's approach
│   ├── copy-trading.md         # Torin's bot
│   └── arbitrage.md            # Cross-platform
├── architecture/
│   ├── llm-agent.md           # Agent design
│   └── mcp-servers.md         # Platform integrations
├── risk-management/
│   └── kelly-criterion.md
└── team/
    └── responsibilities.md
```

---

## Quotes Worth Remembering

> "Genius is simplicity" - Conor on Kenny's strategy

> "The gap between idea and execution is asymptotically reaching zero" - Nick on AI
> development

> "If you were dumber, you would rely more on AI and that's actually the better
> solution" - Nick to Conor

> "I have not written a single line of code in the entire codebase myself" - Nick on
> AI-first development
