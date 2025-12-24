# Interest Rate Arbitrage Trader - Mean Reversion Strategy

**Repository**: [leocolab/Interest-Rate-Arbitrage-Trader](https://github.com/leocolab/Interest-Rate-Arbitrage-Trader)
**Status**: Active
**Markets**: Polymarket
**Strategy**: Cross-market arbitrage (bond market vs Polymarket)
**Local Clone**: `../reference/Interest-Rate-Arbitrage-Trader`

## Overview

Compact arbitrage bot (369 lines total) that exploits discrepancies between bond market-implied Fed rate probabilities and Polymarket betting odds using volatility-weighted Z-scores for mean reversion signals.

**Core Thesis**: Bond market incorporates professional analysis faster; Polymarket lags due to retail sentiment. Mean reversion profits from this lag.

## Strategy: Volatility-Aware Mean Reversion

### Cross-Market Probability Comparison

Compares two probability sources:

- **Bond Market** (via Investing.com): Professional traders pricing Fed rate decisions through bond futures
- **Polymarket** (via scraping): Retail/informed prediction market odds

**Spread Calculation**:

```python
currentSpread = bondOddsForNoChange - polyOddsForNoChange
```

### Volatility-Weighted Z-Scores

Uses **volatility-adjusted Z-scores** for signal generation:

```python
# Historical calibration
mean_value = 0  # Assume spreads mean-revert to zero
stdev_value = np.std(data['Spread'])

# Current signal
currentZScore = (currentSpread - mean_value) / stdev_value
currentExpectedVolatility = currentDaysOut * volatility_coefficient
weightedZScore = currentZScore * (1 + (currentExpectedVolatility/32))
```

**Trading Logic**:

```python
if weightedZScore > 1:
    buy = True   # Bet on "No Change" (bond market more bullish)
elif weightedZScore < 1:
    sell = True  # Bet on "Change" (bond market more bearish)
```

**Threshold**: Fixed at Z-score = 1 (no dynamic adjustment despite calculation infrastructure)

**Why This Matters**: A 2-sigma move 50 days out is less significant than 2-sigma move 2 days out. This prevents false signals during high-volatility periods.

### Historical Data-Driven Calibration

Uses 140 historical observations across multiple Fed decision cycles:

```python
# Creating a Cycle ID based on resets at Days_Out = 0
data['Cycle_ID'] = (data['Days_Out'] == 0).cumsum()

# Calculate rolling volatility within each cycle
for cycle in data['Cycle_ID'].unique():
    mask = data['Cycle_ID'] == cycle
    data.loc[mask, 'Rolling_Volatility'] = data.loc[mask, 'Abs_Value'].rolling(window=5).std()
```

Creates a **regime-aware model** that understands each Fed decision cycle has different volatility characteristics.

## Architecture

**Simple Pipeline** (369 total lines):

1. Scrape bond market odds (Investing.com)
2. Scrape Polymarket odds (Selenium + BeautifulSoup)
3. Calculate spread and volatility-weighted Z-score
4. Execute trade if signal exceeds threshold

**Dependencies**:

- `pandas` / `numpy` - Data processing
- `scikit-learn` - Linear regression for volatility prediction
- `BeautifulSoup` - HTML parsing for bond market data
- `Selenium` - Dynamic scraping for Polymarket odds
- `py_clob_client` - Polymarket trading API client
- `requests` - HTTP scraping

**No async/concurrent processing** - sequential execution only.

## Market Integration

**Single Market**: Polymarket only (no multi-exchange arbitrage)

**API Integration**:

```python
from py_clob_client.clob_types import OrderArgs, OrderType
from py_clob_client.order_builder.constants import BUY

client = ClobClient(host, key=key, chain_id=137)  # Polygon chain

order_args = OrderArgs(
    price=price,
    size=1000.0,  # Fixed 1000 token size
    side=BUY,
    token_id=tokenId,
)
signed_order = client.create_order(order_args)
resp = client.post_order(signed_order, OrderType.GTC)
```

**Market Making**: No - this is a **taker strategy** (buys at market prices)

**Configuration** (inputs.py):

```python
betName = 'fed-interest-rates-january-2025'
dateTime = 'Jan 29, 2025 02:00PM ET'
currentRateRange = '4.25 - 4.50'
buyTokenId = ''      # Must be manually configured
sellTokenId = ''     # Must be manually configured
```

**Scraping Approach**:

- **Polymarket**: Selenium with headless Chrome (dynamic JavaScript rendering)
- **Bond Market**: Standard requests + BeautifulSoup (static HTML)

## Risk Management: Minimal

**Position Sizing**: **Hard-coded 1000 tokens** per trade

```python
size=1000.0,  # Fixed forever
```

No dynamic sizing, no Kelly Criterion, no volatility-based position scaling.

**Exposure Limits**: None visible

- No max position tracking
- No portfolio-level risk caps
- No correlation analysis
- No drawdown limits

**Error Handling**: Almost none

```python
if bondOddsForNoChange == -1:
    print('Error scraping bond implied odds')
# No retry logic, no fallback, just prints and continues
```

**Volatility Weighting Constant**: Magic number `32` with no explanation

```python
weighted_z_scores = z_scores * (1 + (expected_volatility/32))
```

## Code Quality

### Strengths

- ✅ Concise (200 lines of actual logic)
- ✅ Clear separation (data sources isolated)
- ✅ Statistical rigor (volatility prediction using historical data)

### Weaknesses

- ❌ **No error handling** - bare exceptions will crash
- ❌ **Magic numbers everywhere** - `rolling_window = 5`, `/ 32`, `zScoreThresh = 1`, `size=1000.0`
- ❌ **No logging** - just `print(resp)`
- ❌ **Web scraping fragility** - hardcoded CSS selectors will break
- ❌ **No tests** - zero test coverage
- ❌ **Manual configuration** - user must find token IDs manually

## Unique Innovations

### 1. Volatility-Aware Signal Weighting 🔥

**Novel aspect**: Dynamically adjusting Z-score thresholds based on days until event.

```python
# Predict expected volatility using linear regression
X = filtered_data['Days_Out'].values.reshape(-1, 1)
y = filtered_data['Rolling_Volatility'].values
model.fit(X, y)
volatility_coefficient = model.coef_[0]

# Scale Z-score by expected volatility
currentExpectedVolatility = currentDaysOut * volatility_coefficient
weightedZScore = currentZScore * (1 + (currentExpectedVolatility/32))
```

Spreads are more volatile farther from the decision date. The bot predicts expected volatility using linear regression on historical rolling volatility.

### 2. Cross-Market Probability Arbitrage 🔥

Most Polymarket bots trade **within Polymarket** (arbitrage across correlated markets). This bot trades **against external pricing** (bond market), exploiting information asymmetry between institutional bond traders and retail prediction market participants.

### 3. Cycle-Aware Historical Calibration 🔥

Rather than guessing parameters, learns from 140 historical observations across multiple Fed decision cycles:

```python
# Creating a Cycle ID based on resets at Days_Out = 0
data['Cycle_ID'] = (data['Days_Out'] == 0).cumsum()

# Calculate rolling volatility within each cycle
for cycle in data['Cycle_ID'].unique():
    mask = data['Cycle_ID'] == cycle
    data.loc[mask, 'Rolling_Volatility'] = data.loc[mask, 'Abs_Value'].rolling(window=5).std()
```

Each Fed decision cycle has different volatility characteristics - this approach respects that.

## What to Learn

**Steal These Patterns**:

1. **Cross-market arbitrage** - Compare prediction market odds against "ground truth" sources (bond markets, options markets, news sentiment)
2. **Volatility-weighted signals** - Don't treat all Z-scores equally; adjust for expected market conditions
3. **Historical calibration** - Use actual historical spread data to calibrate thresholds
4. **Cycle awareness** - Track market "regimes" separately

**Avoid**:

1. **Hard-coded position sizing** - Implement Kelly Criterion or volatility-based sizing
2. **Fragile web scraping** - Use official APIs; build fallbacks
3. **Zero error handling** - Wrap all external calls in try/except with retries
4. **Manual configuration** - Automate token ID discovery
5. **No logging/observability** - Implement structured logging

**Architecture Lessons**:

- **Simple beats complex** for MVP - This 200-line bot actually trades
- **Statistical arbitrage doesn't need LLMs** - Linear regression sufficient for volatility prediction
- **Data quality > model complexity** - Historical data is the edge, not fancy algorithms

## Philosophical Differences

**Interest-Rate-Arbitrage-Trader**:

- Pure statistical arbitrage
- Single market focus (Fed rates)
- Linear regression for volatility
- No AI/LLM

**Antevorta**:

- LLM agents as decision-makers
- Multiple strategies and markets
- Adaptive and experimental

## Verdict

**Production-quality concept, prototype-quality implementation**. The statistical approach is sound; the engineering needs hardening.

The code is a great proof-of-concept showing that cross-market arbitrage works, but needs significant improvements for production deployment.

**Rating**: ⭐⭐⭐ (3/5)
**Maturity**: Prototype
**Code Quality**: Low (needs hardening)
**Innovation**: High (volatility-weighted signals, cross-market arb)
**Relevance**: Medium (good strategy patterns, no AI)
