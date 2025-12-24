# Kalshi GenAI Trading Bot - LLM as Autonomous Trader

**Repository**: [ajwann/kalshi-genai-trading-bot](https://github.com/ajwann/kalshi-genai-trading-bot)
**Status**: Active (production-ready)
**Markets**: Kalshi
**Strategy**: Grok 4.1 as autonomous trading decision-maker
**Local Clone**: `../reference/kalshi-genai-trading-bot`

## Overview

Production-ready bot that uses **Grok 4.1 Fast Reasoning** (xAI) as the autonomous trading decision-maker. The bot finds new markets on Kalshi, sends metadata to Grok, and Grok performs online research via X.com to decide whether to trade.

**Key Innovation**: Treats LLM as the trader, not as an assistant to traders.

**Stats**:

- **Codebase**: 418 lines of Python
- **LLM**: Grok 4.1 Fast Reasoning (with X.com data access)
- **Decision latency**: ~2-4 seconds per market analysis
- **Architecture**: Stateless serverless (GCP Cloud Functions)

## Strategy: LLM-as-Trader Paradigm

### How It Works

**Traditional**: Human decides → Bot executes
**This bot**: **Grok decides → Bot executes**

The human only sets:

- Which markets to analyze (time window)
- Position size (hardcoded to 1 contract)
- When to run (hourly cron)

### Decision Flow

1. **Market Discovery**: Bot finds new markets on Kalshi (created in last 1 hour)
2. **Send to Grok**: For each market, sends metadata to Grok:
   - Title, ticker, subtitle, category
   - Current yes/no prices
3. **Grok Research**: Grok performs **online research** using X.com data access
4. **Structured Response**: Grok returns JSON:
   ```json
   {
     "ticker": "KX-123", // or null if pass
     "explanation": "Odds diverge from polling data."
   }
   ```
5. **Automatic Execution**: Bot places order if Grok recommends

### Role-Based Prompting

```python
system_prompt = (
    "You are an expert prediction market trader, similar to 'Domer'. "
    "You analyze market metadata to find high-probability opportunities. "
    "You need to perform online research to become an expert in a market "
    "before taking a position. "
    "Respond ONLY with a valid RFC 8259 JSON object containing exactly "
    "two keys: 'ticker' (string or null) and 'explanation' (string or null)."
)
```

**Temperature**: 0.1 (deterministic for consistent JSON output)

### Why Grok?

**Two critical strategic reasons**:

1. **X.com Real-Time Data Access**:
   - Recent X ToS changes restrict X data to Grok only
   - Prediction markets are time-sensitive
   - Real-time social data is alpha
   - Enables news/sentiment analysis not available to other models

2. **API Compatibility**:
   - OpenAI-compatible API (easy to swap models)
   - Future-proof design for model switching

### Example Decision

**Input**:

```json
{
  "title": "Will inflation exceed 3% in Q4 2024?",
  "ticker": "INFL-Q4-24",
  "yes_ask": 68,
  "no_ask": 35
}
```

**Grok Research**:

- Searches X.com for recent inflation news
- Analyzes Federal Reserve statements
- Reviews economic indicators
- Compares market price (68%) to analyzed probability

**Output**:

```json
{
  "ticker": "INFL-Q4-24",
  "explanation": "Market overpricing risk. Fed signaling rate cuts,
                  latest CPI down 0.2%. Fair value ~55%. Buy NO."
}
```

## Architecture

**Dual-Mode Execution**:

- **Local**: Standard Python execution with `.env` config
- **GCP Cloud Functions**: Serverless, hourly cron trigger

**Stateless Design**:

- No database
- No position tracking beyond API calls
- Each run is independent
- Scales infinitely on GCP

**Tech Stack**:

- Python 3.13.11
- Minimal dependencies:
  - `requests` - HTTP client
  - `cryptography` - RSA signing for Kalshi auth
  - `functions-framework` - GCP Cloud Functions
  - `google-cloud-secret-manager` - Secure key storage
  - `pytest` - Testing

### Kalshi API Integration

**Authentication** (RSA-PSS signatures):

```python
def _sign_request(self, method: str, path: str, timestamp: str) -> str:
    payload = f"{timestamp}{method}{path}".encode('utf-8')
    signature = self.private_key.sign(
        payload,
        padding.PSS(
            mgf=padding.MGF1(hashes.SHA256()),
            salt_length=padding.PSS.MAX_LENGTH
        ),
        hashes.SHA256()
    )
    return base64.b64encode(signature).decode('utf-8')
```

**Market Discovery**:

```python
def get_active_markets(self, created_after_hours: int = 1):
    # Fetches open markets from last N hours
    # Filters client-side by creation timestamp
    # Returns markets with yes_ask < 100 and no_ask < 100 (viable)
```

**Order Execution**:

```python
def create_market_order(self, ticker: str, side: str = "yes", count: int = 1):
    # Places market buy order
    # Defaults: 1 contract, yes side
    # Auto-cancels on market pause
```

## Risk Management

### Current Implementation (Minimal)

**Position Limits**:

- Hardcoded: **1 contract per market**
- No total capital limits (dangerous!)

**Market Selection**:

- Only trades newly created markets (1-hour lookback)
- Skips markets with existing positions
- Only liquid markets (both sides < 100 cents)

**Execution**:

- Market orders only (no limit orders)
- Always buys yes side (no short positions)
- No stop-loss or take-profit logic

### Missing Critical Safeguards

- ❌ No max capital exposure per trade
- ❌ No portfolio-level position limits
- ❌ No loss limits (daily/weekly/monthly)
- ❌ No circuit breakers for rapid losses
- ❌ No Kelly criterion or position sizing
- ❌ No correlation analysis between positions

**Code Example**:

```python
# Always buys 1 yes contract - no position sizing!
order_response = kalshi.create_market_order(
    ticker, side="yes", count=1, price=market.get('yes_ask')
)
```

## Code Quality

### Strengths

- ✅ Clean structure with single responsibility classes
- ✅ Type hints throughout
- ✅ Comprehensive logging to stdout
- ✅ Error handling with try/except blocks
- ✅ Unit tests for both clients (though limited coverage)

### Weaknesses

- ❌ Hardcoded values (1 contract, always yes side)
- ❌ No retry logic for API failures
- ❌ No rate limiting awareness
- ❌ Missing integration tests
- ❌ No backtesting framework

### Documentation

**Good** - concise, useful docstrings:

```python
def analyze_market(self, market_data: dict) -> dict:
    """
    Sends market data to Grok and asks for a trading decision.
    Returns the JSON parsed response.
    """
```

## Unique Innovations

### 1. Vibe Coding - LLM-Generated Codebase 🔥

The entire project was generated through iterative prompting:

- **13 prompt versions** documented in `/prompts`
- Progressive refinement from v1 to v13
- Final codebase is production-ready from LLM generation

**Evolution visible**:

- v1: Basic concept
- v6: Added comprehensive documentation requirements
- v13: Refined to current implementation

### 2. LLM-as-Trader (not LLM-as-Assistant) 🔥

Instead of using LLMs to assist human traders, **makes the LLM the trader**.

### 3. Role-Based Prompting for Financial Decisions 🔥

Treats LLM as professional trader "Domer" (famous prediction market trader):

```python
"You are an expert prediction market trader, similar to 'Domer'."
```

Anchors the model's behavior to a specific trading persona.

### 4. Online Research Integration 🔥

The prompt explicitly tells Grok to research:

```python
"You need to perform online research to become an expert in a market
before taking a position."
```

Leverages Grok's unique X.com data access for edge discovery.

### 5. Stateless Serverless Trading 🔥

Pure functions, no state:

- No database
- No position tracking beyond API calls
- Each run is independent
- Scales infinitely on GCP

### 6. Configuration-Driven Model Swapping 🔥

Environment variable for model selection:

```bash
GROK_MODEL=grok-4-1-fast-reasoning  # or grok-beta, grok-2, etc.
```

Makes A/B testing different models trivial.

## What to Learn

**What's Brilliant**:

- Clean architecture showing LLM-native decision making
- Real-world example of autonomous AI trading
- Demonstrates X.com data access advantage
- Production-ready code from LLM generation
- Role prompting works - "You are a professional trader like Domer"
- Structured JSON output - Critical for reliable execution
- Low temperature (0.1) - Consistency matters
- Online research capability - Real alpha from real-time data
- Stateless is powerful - Simplifies deployment and scaling

**What's Concerning**:

- **Zero real risk management** - could lose everything quickly
- Fixed 1-contract size ignores position sizing theory
- No backtesting or performance tracking
- Blindly trusts LLM decisions without validation
- No circuit breakers for model failures

**Code Patterns to Adopt**:

```python
# Their clean client pattern
class GrokClient:
    def __init__(self, api_key: str, model: str):
        self.api_key = api_key
        self.model = model

    def analyze_market(self, market_data: dict) -> dict:
        # Structured request/response
        # Error handling
        # JSON validation
```

**What to Build Different**:

1. **Multi-model ensemble** - Don't trust single LLM
2. **Confidence scores** - LLM should express uncertainty
3. **Position sizing** - Kelly criterion based on confidence
4. **Portfolio limits** - Max exposure across all positions
5. **Performance tracking** - Log every decision for backtesting
6. **Human approval loop** - For large trades or low confidence

## Philosophical Differences

**kalshi-genai-trading-bot**:

- LLM is the trader (autonomous)
- Single model (Grok)
- Stateless execution
- 1 contract per trade

**Antevorta**:

- LLM agents as decision-makers (similar!)
- Multi-model ensemble
- Position sizing and risk management
- Adaptive strategies

## Verdict

Remarkably clean reference implementation showing **LLM-as-autonomous-agent** trading in production. The architecture is sound, but risk management is dangerously minimal.

Perfect example of how to structure LLM trading decisions, but needs significant risk management enhancements for production use.

**Rating**: ⭐⭐⭐⭐ (4/5)
**Maturity**: Beta (works but needs risk mgmt)
**Code Quality**: High
**Innovation**: Very High (LLM-as-trader, vibe coding)
**Relevance**: Very High (directly aligned with Antevorta's vision)
