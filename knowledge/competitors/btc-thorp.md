# BTC-Thorp - Volatility Trading Bot

**Repository**: [ppgrobot/BTC-Thorp](https://github.com/ppgrobot/BTC-Thorp)
**Status**: Active (recent commits)
**Markets**: Kalshi
**Strategy**: BTC/ETH hourly volatility NO contracts
**Local Clone**: `../reference/BTC-Thorp`

## Overview

BTC-Thorp is a quantitative volatility trading bot for Kalshi's hourly crypto markets. It identifies when the market underprices NO contracts on BTC/ETH hourly strikes by comparing market-implied probabilities to a realized volatility model.

**Core Thesis**: Markets misprice short-term crypto volatility. They imply 15-20% probability of 20bp moves when realized volatility shows these moves happen <1% of the time.

## Strategy

### Volatility Model

Uses random walk assumption with normal distribution:

1. **Price Collection** (every 10 seconds):
   - Fetches BTC/ETH prices from Coinbase API
   - Stores in DynamoDB with 7-day TTL
   - Calculates rolling volatility windows: 5m, 7m, 10m, 12m, 15m, 30m, 60m, 90m, 120m

2. **Dynamic Volatility Windows** 🔥 (innovation):
   - At 15 min to settlement: use 15m volatility
   - At 10 min to settlement: use 10m volatility
   - At 5 min to settlement: use 5m volatility
   - Matches time horizon to actual risk period (no sqrt scaling needed)

3. **Probability Calculation**:

   ```python
   # Apply volatility floor (0.15%) and ceiling (10%)
   vol_with_floor = max(vol_std_pct, 0.15%)

   # Scale to time remaining
   vol_scaled = vol_with_floor * sqrt(minutes_remaining / 15)

   # Z-score (standard deviations above current price)
   z = (strike - current_price) / (current_price * vol_scaled)

   # Probability NO wins (BTC stays below strike)
   P(NO) = Φ(z)  # Normal CDF, capped at 99%
   ```

4. **Edge Calculation**:
   ```python
   model_prob = 99.97%  # From volatility model
   market_prob = 81%    # Implied by NO price of 81¢
   edge = 18.97 percentage points
   ```

### Trading Windows

**Early Window (H:30-H:45)**:

- Requires 12% edge minimum
- More time = more uncertainty = bigger edge needed

**Late Window (H:45-H:00)**:

- Requires 4% edge minimum
- Less time = less uncertainty = accept smaller edge

They log "sliding scale" analysis to explore dynamic edge requirements.

### Position Sizing

**Quarter Kelly Criterion**:

```python
# Kelly formula for asymmetric payoffs
b = profit_cents / risk_cents  # e.g., 19/81 = 0.235
p = win_prob
q = 1 - win_prob
kelly_fraction = (b * p - q) / b

# Cap at 20% for BTC, 25% for ETH (originally)
kelly_fraction = min(kelly_fraction, 0.20)

# Share Kelly budget across BTC + ETH
remaining_kelly = MAX_KELLY - (btc_exposure + eth_exposure)
contracts = min(floor(bankroll * kelly_fraction / price), 999)
```

**Shared Position Tracking** 🔥 (innovation):

- Tracks combined BTC + ETH exposure per hour
- Prevents over-leveraging when both bots find opportunities
- TTL-based cleanup (2 hours)

## Risk Management

**Multi-layered approach**:

1. **Strike Selection**:
   - Minimum 30 bps above current price
   - Ensures meaningful edge requirement

2. **Volatility Bounds**:
   - Floor: 0.15% (prevents overconfidence in calm markets)
   - Ceiling: 10% (halts trading when model unreliable)

3. **Edge Requirements**:
   - Early: 12% minimum
   - Late: 4% minimum

4. **Position Sizing**:
   - Quarter Kelly (conservative)
   - Shared across BTC + ETH
   - Max 999 contracts per trade

5. **Price Bounds**:
   - Minimum NO price: 50¢ (too risky below)
   - Maximum NO price: 99¢ (no profit above)
   - Minimum profit: 4%

6. **Model Caps**:
   - Max model probability: 99% (prevents "100% certain" bets)

7. **Balance Checks**:
   - Minimum balance: $1.00
   - Fetches actual Kalshi balance before each trade

## Architecture

**AWS Serverless Stack**:

**DynamoDB Tables**:

- `BTCPriceHistory`: Price + volatility data
- `BTCTradeLog`: Trade history with full context
- `CryptoPositions`: Shared position tracking across BTC + ETH
- `KalshiTradingBudget`: Weather bot daily budget tracking

**Lambda Functions**:

- `BTCPriceCollector`: Runs every minute (6 samples @ 10s intervals)
- `BTCTradingBot`: Runs at H:25 (early) and H:45 (late)
- `PriceHistoryCleanup`: Hourly cleanup

**EventBridge Schedules**:

```python
BTCPriceCollector-EveryMinute: rate(1 minute)
BTCTradingBot-Minute25: cron(25 * * * ? *)
BTCTradingBot-Minute45: cron(45 * * * ? *)
PriceHistoryCleanup-Hourly: rate(1 hour)
```

**Docker Deployment** 🔥:

```bash
# Build for Lambda (linux/amd64)
docker build --platform linux/amd64 -t btc-thorp/btc ./btc

# Push to ECR and deploy
aws lambda create-function \
  --function-name BTCTradingBot \
  --package-type Image \
  --code ImageUri=$IMAGE_URI
```

**Cost**: ~$0.50/month (essentially free tier)

## Code Quality

**Strengths**:

- ✅ Clean separation of concerns
- ✅ Comprehensive error handling
- ✅ Detailed logging at every step
- ✅ Type hints on parameters
- ✅ Modular functions
- ✅ Docker-first deployment
- ✅ Environment variable configuration
- ✅ TTL-based cleanup
- ✅ Excellent file-level documentation

**Weaknesses**:

- ❌ No unit tests
- ❌ No type hints on return values
- ❌ Some magic numbers could be constants
- ❌ No backtesting framework visible

**Documentation**:

- `ARCHITECTURE.md`: System design
- `STRATEGY.md`: Mathematical explanation
- `README.md`: Operational guide
- Inline comments: Explain "why" not "what"

## AI Integration

**None in trading logic** - This is pure quantitative finance:

- Mathematical volatility model
- Kelly criterion for position sizing
- Hard-coded thresholds and risk controls

**Used Claude Code for**:

- Code development and debugging
- Documentation generation
- Infrastructure setup

This is smart: Use AI for engineering productivity, but keep trading decisions deterministic and auditable.

## Unique Innovations

1. **Dynamic Volatility Windows** 🔥
   Match volatility window to time remaining (15m vol for 15m to settlement) instead of scaling single window

2. **Shared Position Tracking** 🔥
   Combined BTC + ETH exposure tracking prevents over-leveraging on correlated opportunities

3. **Trading Window Risk Adjustment** 🔥
   Different edge requirements by time (12% early, 4% late)

4. **High-Frequency Price Collection**
   Every 10 seconds (6 samples/minute) for better volatility estimates

5. **Weather Liquidity Provider** 🔥
   Separate bot that waits for certain outcomes at 6pm, then provides liquidity at 99¢ for rewards

6. **Volatility Floor + Ceiling**
   Prevents overconfidence (floor) and recognizes model breakdown (ceiling)

7. **Docker-First Deployment**
   Container images instead of zip files for Lambda

8. **TTL-Based Lifecycle**
   Data auto-deletes via DynamoDB TTL (no cron jobs)

9. **Sliding Scale Edge Analysis**
   Logs hypothetical trades with different edge requirements (data mining for optimization)

## What to Learn

**Steal These Patterns**:

1. Dynamic volatility windows matching time horizon
2. Shared position tracking across correlated strategies
3. Trading window risk adjustment
4. Comprehensive trade logging (all context)
5. Volatility floor/ceiling guards
6. Model probability caps
7. Docker-first deployment
8. TTL-based data lifecycle
9. Multiple risk layers

**What They Do Well**:

- Clean separation of concerns
- Defensive programming (many safety checks)
- Observable system (great logging)
- Simple, auditable math (no black boxes)
- Multi-layered risk controls

**What They Could Improve**:

- No unit tests
- No backtesting framework
- Manual parameter tuning
- Limited portfolio management
- No dynamic position sizing based on market conditions

## Philosophical Differences

**BTC-Thorp**:

- No AI in trading decisions (pure quant)
- Focus on one simple edge (volatility mispricing)
- Deterministic and auditable

**Antevorta**:

- LLM agents as decision-makers
- Multiple strategies (volatility, copy trading, arbitrage)
- Adaptive and experimental

## Verdict

**Excellent reference material** - clean, well-documented, production-tested. Their risk management framework is particularly strong and worth emulating. The dynamic volatility windows and shared position tracking are innovations we should adopt.

**Rating**: ⭐⭐⭐⭐⭐ (5/5)
**Maturity**: Production
**Code Quality**: High
**Innovation**: High
**Relevance**: Very High
