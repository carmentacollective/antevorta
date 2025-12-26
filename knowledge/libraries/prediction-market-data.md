# Prediction Market Data Libraries - Landscape Analysis

## Executive Summary

The prediction market data ecosystem is fragmented across platform-specific SDKs with
limited unified solutions. **predmarket** is the most promising unified library but is
very early stage. This analysis covers the full landscape.

## predmarket - Unified Prediction Market SDK

**Repository**:
[github.com/ashercn97/predmarket](https://github.com/ashercn97/predmarket)
**Maintainer**: ashersopro2 (individual developer) **Status**: Active development,
expect breaking changes

### Overview

Asyncio-native Python library unifying Kalshi and Polymarket APIs into a single
interface. The only library attempting cross-platform abstraction.

### Data Sources

| Platform   | REST API | WebSocket   | Status      |
| ---------- | -------- | ----------- | ----------- |
| Polymarket | Yes      | Yes         | Working     |
| Kalshi     | Yes      | In Progress | Partial     |
| PredictIt  | No       | No          | Not planned |
| Metaculus  | No       | No          | Not planned |

**Data Types Available:**

- Questions/Events listing
- Contracts/Markets data
- Live prices via REST polling
- Real-time updates via WebSocket (Polymarket only)

### Code Quality

| Metric             | Assessment               |
| ------------------ | ------------------------ |
| **Type Hints**     | Yes - Pydantic models    |
| **Tests**          | Not visible in repo      |
| **Documentation**  | README only, no API docs |
| **Dependencies**   | httpx, pydantic          |
| **Python Version** | 3.12+ required           |

**Weaknesses:**

- 61 stars, 11 commits - very early
- No test suite visible
- WebSocket support incomplete
- No historical data access

### Usage Example

```python
from predmarket import KalshiRest, PolymarketRest
import httpx

async with httpx.AsyncClient() as client:
    polymarket = PolymarketRest(client)
    questions = await polymarket.fetch_questions(limit=10)

    kalshi = KalshiRest(client)
    contracts = await kalshi.fetch_contracts()
```

### Verdict

**Fork and extend.** The architecture is sound but coverage is too thin for production
use. We'd need to add Kalshi WebSocket, historical data, and test coverage.

**Rating**: 2.5/5

## Platform-Specific Alternatives

### kalshi-py - Modern Kalshi Client

**Repository**: [apty.github.io/kalshi-py](https://apty.github.io/kalshi-py/)
**Status**: Active, daily builds from OpenAPI spec

- Both sync AND async support
- Full trading support including RSA-PSS auth
- OpenAPI-generated (stays current)
- Better than official SDK for new projects

**Rating**: 3.5/5

### kalshi-python - Official Kalshi SDK

**Repository**:
[github.com/lowgrind/kalshi-python](https://github.com/lowgrind/kalshi-python)
**Status**: Production, auto-generated from Swagger

- Sync-only (no async)
- Full trading support
- Official but less maintained

**Rating**: 3/5

### PredictIt Libraries

| Library           | Stars | Status | Notes                      |
| ----------------- | ----- | ------ | -------------------------- |
| pyredictit        | Low   | Active | Full wrapper               |
| predictit_markets | Low   | Active | Simple, returns DataFrames |
| predictit-data    | Low   | Active | Enforces 60s rate limit    |

**Note**: PredictIt has limited trading hours and is winding down operations.

### Metaculus - forecasting-tools

**Repository**:
[github.com/Metaculus/forecasting-tools](https://github.com/Metaculus/forecasting-tools)
**Status**: Official, active

Full framework for AI forecasting bots. Use this for Metaculus integration.

**Rating**: 3.5/5

### Manifold Markets

| Library     | Status     | Notes                  |
| ----------- | ---------- | ---------------------- |
| manifoldpy  | Deprecated | "No longer maintained" |
| PyManifold  | WIP        | Basic client           |
| manifoldbot | Active     | Bot-focused            |

**Note**: Manifold recommends using their data dumps over API for analysis.

## Commercial/SaaS Alternatives

### PredictionData.dev

- **Coverage**: Polymarket, Kalshi
- **Data**: 10B+ tick-level updates, 3 years historical
- **Model**: Paid API service
- **Use Case**: When you need deep historical data without building infrastructure

### FinFeedAPI

- **Coverage**: Polymarket, Kalshi, Myriad, Manifold
- **Model**: Unified paid API
- **Use Case**: Production systems wanting managed infrastructure

### Dome (YC 2025)

- **Coverage**: Kalshi, Polymarket
- **Model**: Unified API startup
- **Status**: New, co-founded by Polymarket infrastructure builders
- **Use Case**: Enterprise-grade unified access

## Strategic Recommendations for Antevorta

### Option 1: Multi-SDK Approach (Recommended Short-Term)

Use official SDKs directly:

```python
# polymarket_client.py
from py_clob_client.client import ClobClient

# kalshi_client.py
import kalshi_python

# Our unified interface
class PredictionMarketClient:
    def __init__(self):
        self.polymarket = ClobClient(...)
        self.kalshi = kalshi_python.DefaultApi(...)

    async def get_all_markets(self):
        # Normalize across platforms
        ...
```

**Pros**: Production-proven, official support **Cons**: Maintenance burden for
normalization layer

### Option 2: Fork predmarket (Recommended Medium-Term)

Fork [predmarket](https://github.com/ashercn97/predmarket) and extend:

1. Complete Kalshi WebSocket support
2. Add historical data endpoints
3. Add PredictIt/Metaculus if needed
4. Add comprehensive test suite
5. Production-harden error handling

**Pros**: Clean architecture already in place, asyncio-native **Cons**: Investment
required, solo maintainer upstream

### Option 3: Build Custom (Not Recommended)

Start from scratch with unified architecture.

**Pros**: Complete control **Cons**: Duplicates existing work, high investment

## Final Verdict

| Library               | Use Case                  | Rating |
| --------------------- | ------------------------- | ------ |
| **py-clob-client**    | Polymarket production     | 4.5/5  |
| **kalshi-py**         | Kalshi with modern Python | 3.5/5  |
| **predmarket**        | Unified prototype/fork    | 2.5/5  |
| **forecasting-tools** | Metaculus AI forecasting  | 3.5/5  |

**Recommendation**: Start with official SDKs (py-clob-client + kalshi-python) wrapped in
our own normalization layer. Monitor predmarket development - if it matures, consider
adopting or contributing. For arbitrage scanning across platforms, the multi-SDK
approach gives us the reliability we need for real money trading.
