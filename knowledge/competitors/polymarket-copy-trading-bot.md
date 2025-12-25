# Polymarket Copy Trading Bot

**Repository**:
[GiordanoSouza/polymarket-copy-trading-bot](https://github.com/GiordanoSouza/polymarket-copy-trading-bot)
**Status**: Active (working implementation) **Markets**: Polymarket only **Strategy**:
Copy trading with ~4-8 second latency **Local Clone**:
`../reference/polymarket-copy-trading-bot`

## Overview

Production-ready copy trading bot that mirrors a target trader's positions on Polymarket
with 4-8 second latency. Uses Supabase for real-time event streaming and dual-layer
detection (polling + Realtime) for reliability.

**Key Stats**:

- **Codebase**: 1,424 lines of Python
- **Latency**: ~4-8 seconds from trader action to bot execution
- **Architecture**: Dual-layer (polling + Realtime events)
- **Database**: Supabase (PostgreSQL with Realtime)

## Strategy: Copy Trading Mechanics

### Trader Selection

**Manual selection** - users find traders via
[polymarket.com/leaderboard](https://polymarket.com/leaderboard) and copy their wallet
address to `.env`. The bot blindly copies whatever the target trader does with no
quality filters.

### Dual-Layer Trade Detection

**Layer 1: Polling** (background threads)

- **History Poller**: Fetches trade history every 5 seconds via `/activity` API
- **Position Poller**: Fetches positions every 6 minutes via `/positions` API
- Both insert/update Supabase tables

**Layer 2: Realtime** (Supabase subscriptions)

- Listens to `INSERT` on `historic_trades` table
- Listens to `INSERT` and `UPDATE` on `polymarket_positions` table
- Triggers immediate order execution when changes detected

### Latency Analysis

**Chain**:

1. Trader places order → transaction confirmed (~2-3s Polygon block time)
2. Polling script fetches latest activities (5s interval)
3. Insert into Supabase triggers Realtime event (<100ms)
4. Bot handler receives event and executes order (~500ms)

**Total: ~4-8 seconds** from trader action to bot order execution.

**Bottleneck**: The 5-second polling interval. Could be reduced to sub-second latency
by:

- Switching to WebSocket streams from Polymarket
- Using blockchain event monitoring (watch trader's proxy wallet directly)

### Trade Execution Logic

**Three Event Handlers**:

1. **New Trade (BUY)**:

   ```python
   sized_price = sizing_constraints(usdc_size)
   if sized_price >= 1:  # Minimum $1 trade
       make_order(price, sizing_constraints(size), BUY, token_id)
   ```

2. **New Trade (SELL)** - Proportional Exit:

   ```python
   trader_size = fetch_player_positions(trader_wallet)[0]['size']
   my_size = fetch_player_positions(my_wallet)[0]['size']
   percentage = usdc_size / trader_size
   final_size = percentage * my_size
   make_order(price, final_size, SELL, token_id)
   ```

   **Smart**: If trader exits 50% of position, bot exits 50% too (not just 0.5% due to
   sizing).

3. **Position Update**:
   ```python
   sized_value = sizing_constraints(new_value - old_value)
   if sized_value > 1:
       make_order(avg_price, sized_value, BUY, token_id)
   elif sized_value <= -1:
       make_order(avg_price, sized_value, SELL, token_id)
   ```

## Architecture

**Database Schema** (PostgreSQL via Supabase):

**`historic_trades`** - Every trade action:

```sql
- proxy_wallet, timestamp, condition_id
- type, size, usdc_size, price, side
- asset (token_id), outcome, outcome_index
- transaction_hash (unique identifier)
- unique_key GENERATED ALWAYS AS (transaction_hash || '_' || condition_id || '_' || price)
```

**Smart deduplication**: Generated column prevents duplicate trade processing.

**`polymarket_positions`** - Current holdings:

```sql
PRIMARY KEY (proxy_wallet, asset)
- size, avg_price, initial_value, current_value
- cash_pnl, percent_pnl, realized_pnl
- market metadata (title, slug, outcome, end_date)
```

**Update detection**: Compares old vs new values with 0.1 tolerance to avoid unnecessary
UPDATEs.

**Tech Stack**:

- Python 3.9+
- Supabase >=2.0.0 (async client for DB + Realtime)
- py-clob-client 0.20.0 (official Polymarket SDK)
- websockets >=12.0 (Realtime transport)

## Market Integration: Polymarket

**Data API** (`https://data-api.polymarket.com`):

- `/activity` - Trade history (timestamp, size, price, side, transaction_hash)
- `/positions` - Current holdings (size, P&L, avg_price, current_value)

**CLOB API** (`https://clob.polymarket.com`):

- Used via `py-clob-client`
- Order placement with signed transactions
- Uses Magic wallet (email-based) with proxy address

**Authentication**:

1. User provides private key from Magic wallet (reveal.magic.link/polymarket)
2. Client derives API credentials
3. Signs orders with private key using eth_account
4. Posts to CLOB with API creds + signature

## Risk Management

### Position Sizing - Single Constraint

```python
sizing_whale_pct = 0.005  # Default: 0.5% of trader's size

def sizing_constraints(usdc_size: float) -> float:
    return usdc_size * sizing_whale_pct
```

**Example**:

- Trader bets $10,000 → Bot bets $50 (at 0.5%)
- Trader bets $1,000 → Bot bets $5 (at 0.5%)

### Safeguards

**Minimum Trade Size**:

```python
if sized_price >= 1:  # $1 minimum
    make_order(...)
```

**Configuration Limits** (defined but NOT enforced):

```python
BANKROLL = 1000      # Total capital (not enforced in code!)
STAKE_MIN = 5        # Defined but unused
STAKE_MAX = 20       # Defined but unused
```

**Critical gap**: `BANKROLL`, `STAKE_MIN`, `STAKE_MAX` are loaded but never enforced.
Bot can exceed bankroll if trader makes huge bets.

### Missing Risk Controls

**No implementation** for:

- Total exposure across positions
- Bankroll % per trade enforcement
- Maximum concurrent positions
- Market liquidity checks
- Drawdown limits

## Code Quality

### Strengths

- ✅ Clean configuration management (singleton pattern)
- ✅ Database change detection (0.1 tolerance)
- ✅ Structured event handling (separate INSERT vs UPDATE)
- ✅ Comprehensive logging at every step

### Weaknesses

- ❌ **Error handling - silent failures**:

  ```python
  try:
      resp = client.post_order(signed_order, OrderType.GTC)
      print(resp)
      return resp
  except Exception as e:
      print(f"Error making order: {e}")
      return None  # Order fails silently
  ```

  No retries, no alerting, no fallback.

- ❌ **Hardcoded magic numbers**: `time.sleep(5)`, `time.sleep(6 * 60)`,
  `if sized_price >= 1`
- ❌ **No tests**: Zero test files
- ❌ **Incomplete type hints**: Missing return type annotations
- ❌ **Dead code**: `scripts/listen_to_order.py` entirely commented out (60 lines)

## Unique Innovations

### 1. Dual-Layer Detection (Polling + Realtime) 🔥

Most copy trading bots use either polling OR websockets. This bot uses both:

- Polling ensures no missed trades (robustness)
- Realtime ensures low latency (speed)
- Trade-off: Double API load, but guarantees data integrity

### 2. Database-Centric Architecture 🔥

Most bots process events in-memory. This bot:

- Stores everything in Supabase
- Uses database as event bus via Realtime
- Enables historical analysis, debugging, multiple bot instances

**Benefit**: Can stop/restart bot without losing state.

### 3. Proportional Exit Logic 🔥

```python
percentage_position = usdc_size / size_trader
final_size = percentage_position * size_myself
```

**Smart**: If trader exits 50% of position, bot exits 50% too. Maintains trade alignment
during exits.

### 4. Generated Column Deduplication 🔥

```sql
unique_key VARCHAR(500) GENERATED ALWAYS AS (
    transaction_hash || '_' || condition_id || '_' || price
) STORED;
```

**Elegant**: Database-level deduplication prevents race conditions from concurrent
polling.

### 5. Supabase Realtime as Event System 🔥

Using Supabase Realtime (PostgreSQL logical replication) instead of:

- RabbitMQ/Kafka (overkill)
- Redis pub/sub (requires separate infrastructure)
- Direct websockets (harder to scale)

**Pragmatic choice** for MVP with built-in persistence.

## What to Learn

**Steal These Patterns**:

1. Dual-layer detection (polling + Realtime)
2. Database as event bus
3. Proportional exit scaling
4. Generated column deduplication
5. Supabase for real-time infrastructure

**What Works Well**:

- Low latency (~4-8s) via dual polling + Realtime
- Simple, readable codebase (1,424 LOC)
- Robust data persistence
- Proportional exit logic mirrors trader accurately
- Clean configuration with validation

**Critical Gaps**:

1. **No AI/LLM** - contrary to Antevorta's direction
2. **Minimal risk management** - BANKROLL unused, no position limits enforced
3. **No error recovery** - failed orders disappear
4. **Blind copying** - no trade quality filters
5. **Scaffolded constraints** - 4 constraint modules empty

## Philosophical Differences

**polymarket-copy-trading-bot**:

- No AI in decisions (blind copying)
- Single trader focus
- Manual selection

**Antevorta**:

- LLM agents as decision-makers
- Multiple strategies
- Adaptive and experimental

## Verdict

**MVP-ready** for personal use with manual monitoring. Not production-ready for
automated capital allocation or unattended operation.

**Rating**: ⭐⭐⭐ (3/5) **Maturity**: Beta (working but needs hardening) **Code
Quality**: Medium **Innovation**: Medium-High (dual-layer detection, database-centric)
**Relevance**: Medium (good architecture patterns, missing AI)
