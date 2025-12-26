# Prediction Market Language Ecosystem Analysis

Research Date: December 2025

## Executive Summary

**Python is the dominant language** for prediction market trading infrastructure. It has
stronger official SDK support, 7x more GitHub activity, and virtually all production
trading bots are written in Python. TypeScript has a role for frontend/web integrations
but is secondary for trading bots.

---

## 1. Official SDK Availability

### Polymarket

| Language   | Official SDK                                                                   | Stars | Forks | Status            |
| ---------- | ------------------------------------------------------------------------------ | ----- | ----- | ----------------- |
| **Python** | [py-clob-client](https://github.com/Polymarket/py-clob-client)                 | 480   | 178   | Active, v0.34.0   |
| TypeScript | [clob-client](https://github.com/Polymarket/clob-client)                       | 84    | 322   | Active, v5.1.0    |
| TypeScript | [real-time-data-client](https://github.com/Polymarket/real-time-data-client)   | -     | -     | WebSocket wrapper |
| TypeScript | [builder-relayer-client](https://github.com/Polymarket/builder-relayer-client) | -     | -     | Relayer API       |

**Python has 5.7x more stars** on the main trading SDK. Both are actively maintained.

### Kalshi

| Language   | Official SDK                                                          | PyPI/npm  | Status                      |
| ---------- | --------------------------------------------------------------------- | --------- | --------------------------- |
| **Python** | [kalshi-python](https://pypi.org/project/kalshi-python/)              | v2.1.4    | Official, OpenAPI-generated |
| TypeScript | [kalshi-typescript](https://www.npmjs.com/package/kalshi-typescript)  | v2.1.3    | Official, OpenAPI-generated |
| Go         | [ammario/kalshi](https://github.com/ammario/kalshi)                   | Community | Third-party                 |
| Rust       | [dpeachpeach/kalshi-rust](https://github.com/dpeachpeach/kalshi-rust) | Community | Third-party                 |

**Both languages have official SDKs** for Kalshi (auto-generated from OpenAPI spec).

---

## 2. GitHub Community Activity

### Search Results: Code Files Containing Platform Name

| Platform   | Python Files | TypeScript Files | Ratio           |
| ---------- | ------------ | ---------------- | --------------- |
| Polymarket | 952          | 135              | **7.1x Python** |
| Kalshi     | 832          | 33               | **25x Python**  |

### Production Repositories

Key Python repositories:

- [Polymarket/agents](https://github.com/polymarket/agents) - **975 stars**, official AI
  trading framework
- [nautechsystems/nautilus_trader](https://github.com/nautechsystems/nautilus_trader) -
  Professional trading platform with Polymarket integration
- [warproxxx/poly-maker](https://github.com/warproxxx/poly-maker) - Market making bot
- [Kalshi/tools-and-analysis](https://github.com/Kalshi/tools-and-analysis) - 67 stars,
  official starter code

---

## 3. Production Bot Analysis

### What Language Are Serious Bots Written In?

| Bot Type          | Language          | Example Repos                                                |
| ----------------- | ----------------- | ------------------------------------------------------------ |
| AI Trading Agents | Python            | Polymarket/agents, BlackSky-Jose/PolyMarket-trading-AI-model |
| Market Making     | Python            | warproxxx/poly-maker, poly-maker                             |
| Copy Trading      | TypeScript/Python | Various implementations exist in both                        |
| Spike Detection   | Python            | Trust412/Polymarket-spike-bot-v1                             |
| Arbitrage         | Python            | dexorynLabs/polymarket-kalshi-arbitrage-trading-bot-v1       |
| Data Collection   | Python            | harrodyuan/kalshi-data-collector                             |

**Pattern**: Python dominates for algorithmic trading. TypeScript appears in some
copy-trading bots (likely due to developers coming from web3 frontend background).

### Key Observation

The official [Polymarket/agents](https://github.com/polymarket/agents) repository (975
stars) is **Python-based**. This is Polymarket's own framework for AI trading agents,
which signals Python as the preferred language for autonomous trading.

---

## 4. Web3/Blockchain Ecosystem

Polymarket runs on Polygon (EVM-compatible). The key libraries:

### JavaScript/TypeScript

| Library   | Bundle Size | Stars | Notes                                   |
| --------- | ----------- | ----- | --------------------------------------- |
| ethers.js | ~130 KB     | 7.8k+ | Mature, comprehensive                   |
| viem      | ~27 KB      | 2k+   | Modern, TypeScript-first, tree-shakable |
| web3.js   | ~400 KB     | 18k+  | Oldest, largest community               |

### Python

| Library | Notes                          |
| ------- | ------------------------------ |
| web3.py | Ethereum Foundation maintained |
| py-evm  | Low-level EVM implementation   |

### Ecosystem Health Comparison

**TypeScript/JavaScript**:

- Larger overall Web3 ecosystem (React/Wagmi/RainbowKit stack)
- Better for frontend dApps
- viem is the modern choice, ethers.js more established
- Wagmi + viem is the current best practice for React apps

**Python**:

- web3.py is solid and maintained by Ethereum Foundation
- Better integration with data science (pandas, numpy, ML)
- Preferred for backend/trading infrastructure
- Polymarket's official clients use py-evm for signing

### Polymarket-Specific

Polymarket's Python SDK (`py-clob-client`) handles all the Web3 complexity internally:

- EIP-712 signing
- Polygon transaction submission
- Order book interactions

You don't need to use web3.py directly - the SDK abstracts it.

---

## 5. Why Python Dominates for Trading

1. **Data Science Integration**: pandas, numpy, scipy for analysis
2. **ML/AI Libraries**: LangChain, transformers for AI agents
3. **Backtesting**: More mature quantitative finance libraries
4. **Official Support**: Polymarket's AI agents framework is Python
5. **Trading Community**: Quant finance has Python roots
6. **Jupyter Notebooks**: Interactive strategy development

---

## 6. When to Use TypeScript

TypeScript makes sense for:

- Frontend dashboards displaying market data
- Browser-based trading interfaces
- Copy-trading bots if you're a TS-native developer
- Projects already in the React/Next.js ecosystem

TypeScript does NOT make sense for:

- AI/ML trading strategies (Python ecosystem is far superior)
- Backtesting and quantitative analysis
- Data pipeline work

---

## Recommendation

**For Antevorta: Use Python**

Reasons:

1. Official Polymarket AI agents framework is Python
2. 7x more community code to reference
3. Better ML/LLM integration (LangChain, Claude API, etc.)
4. Kalshi has equal Python support
5. Team's strongest language

**Consider TypeScript only for**:

- Web-based dashboards (if needed later)
- Real-time WebSocket visualization

---

## Sources

### Official Documentation

- [Polymarket CLOB Documentation](https://docs.polymarket.com/developers/CLOB/clients/methods-overview)
- [Kalshi SDK Documentation](https://docs.kalshi.com/sdks/overview)
- [Kalshi TypeScript Quickstart](https://docs.kalshi.com/sdks/typescript/quickstart)

### GitHub Repositories

- [Polymarket GitHub Organization](https://github.com/polymarket) - 91 repos
- [Kalshi GitHub Organization](https://github.com/Kalshi) - 7 repos
- [py-clob-client](https://github.com/Polymarket/py-clob-client) - Python CLOB client
- [clob-client](https://github.com/Polymarket/clob-client) - TypeScript CLOB client
- [Polymarket/agents](https://github.com/Polymarket/agents) - AI trading framework

### Package Registries

- [kalshi-python on PyPI](https://pypi.org/project/kalshi-python/)
- [kalshi-typescript on npm](https://www.npmjs.com/package/kalshi-typescript)

### Web3 Library Comparisons

- [Viem vs Ethers.js - MetaMask](https://metamask.io/news/viem-vs-ethers-js-a-detailed-comparison-for-web3-developers)
- [Web3.js vs Ethers.js Comparison](https://drpc.org/blog/web3-js-vs-ethers-js/)
