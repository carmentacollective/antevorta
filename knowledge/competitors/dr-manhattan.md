# dr-manhattan - CCXT for Prediction Markets

**Repository**: [guzus/dr-manhattan](https://github.com/guzus/dr-manhattan) **Status**:
Active **Platforms**: Polymarket, Kalshi (planned), Opinion, Limitless **Type**: Unified
API abstraction layer **Local Clone**: `../reference/dr-manhattan`

## Overview

CCXT-style unified API for prediction markets. Provides exchange-agnostic interface to
multiple prediction market platforms, enabling portable trading strategies and
cross-platform bots.

**Core Thesis**: Just as CCXT unified cryptocurrency exchanges, dr-manhattan unifies
prediction markets with a common interface.

**Key Features**:

- Single API for multiple exchanges
- Strategy base class for portable trading logic
- WebSocket support for real-time data
- Order tracking and event logging
- Exchange factory pattern
- Full type hints

## Architecture: Exchange Abstraction

### Base Exchange Interface

```python
class Exchange(ABC):
    """Abstract base class all exchanges must implement"""

    @abstractmethod
    def fetch_markets(self) -> List[Market]:
        """Get all available markets"""
        pass

    @abstractmethod
    def fetch_orderbook(self, market_id: str) -> OrderBook:
        """Get current orderbook for market"""
        pass

    @abstractmethod
    def create_order(self, market_id: str, outcome: str, side: OrderSide, price: float, size: float, **params) -> Order:
        """Place an order"""
        pass

    @abstractmethod
    def cancel_order(self, order_id: str, **params) -> bool:
        """Cancel an order"""
        pass

    @abstractmethod
    def fetch_balance(self) -> Dict[str, float]:
        """Get account balance"""
        pass

    @abstractmethod
    def fetch_positions(self) -> List[Position]:
        """Get current positions"""
        pass
```

**Every exchange implements these methods** → Write once, run anywhere.

### Supported Exchanges

**Currently Supported**:

1. **Polymarket** - Largest crypto prediction market
2. **Opinion** - BNB Chain prediction market
3. **Limitless** - New prediction market protocol

**Planned**:

- **Kalshi** - Regulated US prediction market

### Usage: Unified Interface

**Same Code, Different Exchanges**:

```python
import dr_manhattan

# Initialize any exchange with identical interface
polymarket = dr_manhattan.Polymarket({'timeout': 30})
opinion = dr_manhattan.Opinion({'timeout': 30})
limitless = dr_manhattan.Limitless({'timeout': 30})

# Fetch markets - same method for all
poly_markets = polymarket.fetch_markets()
opinion_markets = opinion.fetch_markets()
limitless_markets = limitless.fetch_markets()

# Place order - same method signature
order1 = polymarket.create_order(
    market_id="market_123",
    outcome="Yes",
    side=dr_manhattan.OrderSide.BUY,
    price=0.65,
    size=100,
    params={'token_id': 'token_id'}
)

order2 = limitless.create_order(
    market_id="market_456",
    outcome="Yes",
    side=dr_manhattan.OrderSide.BUY,
    price=0.65,
    size=100
)
```

**Exchange-Agnostic Bot**:

```python
def run_arbitrage(exchange1: Exchange, exchange2: Exchange, market_id1: str, market_id2: str):
    """Works with ANY two exchanges"""
    book1 = exchange1.fetch_orderbook(market_id1)
    book2 = exchange2.fetch_orderbook(market_id2)

    if book1.best_ask < book2.best_bid:
        # Buy on exchange1, sell on exchange2
        exchange1.create_order(market_id1, "Yes", OrderSide.BUY, book1.best_ask, 100)
        exchange2.create_order(market_id2, "Yes", OrderSide.SELL, book2.best_bid, 100)

# Run with any exchange pair
run_arbitrage(polymarket, limitless, "pm_market", "lm_market")
run_arbitrage(opinion, polymarket, "op_market", "pm_market")
```

## Strategy Base Class

**Portable Trading Strategies**:

```python
from dr_manhattan import Strategy

class MakingMarketStrategy(Strategy):
    """Market making strategy that works on any exchange"""

    def __init__(self, exchange: Exchange, market_id: str, spread: float = 0.02):
        super().__init__(exchange, market_id)
        self.spread = spread

    def on_tick(self):
        """Called periodically"""
        book = self.exchange.fetch_orderbook(self.market_id)
        mid = (book.best_bid + book.best_ask) / 2

        # Cancel existing orders
        for order in self.open_orders:
            self.exchange.cancel_order(order.id)

        # Place new orders around mid
        self.exchange.create_order(
            self.market_id, "Yes", OrderSide.BUY, mid - self.spread, 100
        )
        self.exchange.create_order(
            self.market_id, "Yes", OrderSide.SELL, mid + self.spread, 100
        )

# Run on any exchange
strategy = MakingMarketStrategy(polymarket, "market_123", spread=0.01)
strategy.run()

# Same strategy, different exchange
strategy2 = MakingMarketStrategy(limitless, "market_456", spread=0.015)
strategy2.run()
```

## Data Models

**Standardized Models**:

```python
from dataclasses import dataclass

@dataclass
class Market:
    id: str
    question: str
    outcomes: List[str]
    prices: Dict[str, float]  # outcome -> price
    volume: float
    liquidity: float
    end_date: datetime

@dataclass
class Order:
    id: str
    market_id: str
    outcome: str
    side: OrderSide  # BUY or SELL
    price: float
    size: float
    filled: float
    status: OrderStatus  # OPEN, FILLED, CANCELED

@dataclass
class OrderBook:
    market_id: str
    bids: List[Tuple[float, float]]  # (price, size)
    asks: List[Tuple[float, float]]
    best_bid: float
    best_ask: float
    spread: float

@dataclass
class Position:
    market_id: str
    outcome: str
    size: float
    avg_price: float
    current_price: float
    unrealized_pnl: float
```

**Type Safety** - Models use full type hints for autocomplete and validation.

## WebSocket Support

**Real-Time Market Data**:

```python
# Polymarket WebSocket
from dr_manhattan.exchanges.polymarket_ws import PolymarketWS

async with PolymarketWS() as ws:
    async for update in ws.subscribe_orderbook("market_123"):
        print(f"Best bid: {update.best_bid}, Best ask: {update.best_ask}")

# Limitless WebSocket
from dr_manhattan.exchanges.limitless_ws import LimitlessWS

async with LimitlessWS() as ws:
    async for update in ws.subscribe_trades("market_456"):
        print(f"Trade: {update.price} @ {update.size}")
```

**WebSocket Base Class**:

```python
class WebSocket(ABC):
    @abstractmethod
    async def subscribe_orderbook(self, market_id: str):
        """Subscribe to orderbook updates"""
        pass

    @abstractmethod
    async def subscribe_trades(self, market_id: str):
        """Subscribe to trade updates"""
        pass
```

## Exchange Factory

**Dynamic Exchange Creation**:

```python
from dr_manhattan import create_exchange, list_exchanges

# List available exchanges
exchanges = list_exchanges()
print(exchanges)  # ['polymarket', 'limitless', 'opinion']

# Create exchange by name (config-driven)
config = {
    'private_key': os.getenv('PRIVATE_KEY'),
    'timeout': 30
}

exchange = create_exchange('polymarket', config)
markets = exchange.fetch_markets()

# Loop through multiple exchanges
for exchange_name in ['polymarket', 'limitless']:
    ex = create_exchange(exchange_name, config)
    print(f"{exchange_name}: {len(ex.fetch_markets())} markets")
```

## Order Tracking

**Event Logging**:

```python
from dr_manhattan.base.order_tracker import OrderTracker

tracker = OrderTracker()

# Track order lifecycle
order = exchange.create_order(market_id, "Yes", OrderSide.BUY, 0.6, 100)
tracker.track_order(order)

# Log fills
tracker.on_fill(order.id, fill_price=0.605, fill_size=50)

# Get order history
history = tracker.get_order_history(market_id)
```

## Code Quality

### Strengths

- ✅ **Clean abstractions** - Well-designed base classes
- ✅ **Full type hints** - Complete type safety
- ✅ **Minimal dependencies** - Lightweight
- ✅ **CCXT-inspired** - Proven architectural pattern
- ✅ **Extensible** - Easy to add new exchanges
- ✅ **Strategy base class** - Portable trading logic

### Weaknesses

- ❌ **Limited exchange coverage** - Only 3 exchanges (Kalshi planned)
- ❌ **No tests visible** - Missing test suite
- ❌ **Sparse documentation** - README only, no API docs
- ❌ **No error handling** - Basic exception propagation
- ❌ **Third-party** - Not official (unofficial integration risk)

## Unique Innovations

### 1. CCXT Pattern for Prediction Markets 🔥

**Novel aspect**: First CCXT-style unified interface for prediction markets.

CCXT transformed crypto trading by providing one API for 100+ exchanges. dr-manhattan
does the same for prediction markets.

**Before dr-manhattan**:

```python
# Different code for each exchange
polymarket_client = PolymarketClient()
poly_markets = polymarket_client.getMarkets()

kalshi_client = KalshiAPI()
kalshi_markets = kalshi_client.list_markets()
```

**With dr-manhattan**:

```python
# Same code, any exchange
for exchange in [polymarket, kalshi]:
    markets = exchange.fetch_markets()
```

### 2. Strategy Portability 🔥

**Novel aspect**: Write strategy once, run on any exchange.

```python
class MyStrategy(Strategy):
    def on_tick(self):
        self.place_bbo_orders()

# Same strategy, different exchanges
strategy1 = MyStrategy(polymarket, "market_1")
strategy2 = MyStrategy(limitless, "market_2")
```

No exchange-specific code in strategy logic.

### 3. Exchange Factory Pattern 🔥

**Novel aspect**: Config-driven exchange creation.

```python
config = load_config("exchanges.yaml")

for exchange_name, exchange_config in config.items():
    exchange = create_exchange(exchange_name, exchange_config)
    strategies[exchange_name] = MyStrategy(exchange, exchange_config['market_id'])
```

Enable/disable exchanges via configuration without code changes.

## What to Learn

**Steal These Patterns**:

1. **Abstract base classes** - Define interface, let exchanges implement
2. **Factory pattern** - Create exchanges by name string
3. **Strategy base class** - Separate strategy logic from exchange details
4. **Standardized models** - Common data structures across exchanges
5. **Type safety** - Full type hints for developer experience

**Avoid**:

1. **Missing tests** - Need comprehensive test suite
2. **Basic error handling** - Add retries, rate limiting, validation
3. **No docs site** - README is not enough for complex library
4. **Unofficial status** - Coordinate with exchange teams

**Architecture Lessons**:

- **Abstraction enables scale** - Add new exchanges without rewriting strategies
- **CCXT proved the pattern** - Unified APIs work for trading
- **Type hints are critical** - Make abstractions usable
- **Factory pattern is powerful** - Config-driven exchange selection

## Comparison

**dr-manhattan**:

- Multi-exchange support
- Strategy portability
- Abstract interface
- Third-party

**py-clob-client**:

- Polymarket-specific
- Official SDK
- No abstraction
- More comprehensive per exchange

**predmarket**:

- Multi-exchange (Kalshi + Polymarket)
- Async-native
- Less mature
- Different API design

## Verdict

**The right architecture for multi-exchange trading**. If you're building bots that
trade across multiple prediction markets, this is the cleanest approach.

Missing tests, documentation, and some exchanges. But the core abstraction is sound and
mirrors CCXT's proven success.

**Rating**: ⭐⭐⭐⭐ (4/5) **Maturity**: Beta (works but needs polish) **Code Quality**:
High (clean architecture) **Innovation**: Very High (CCXT for prediction markets)
**Relevance**: Very High (essential for multi-exchange strategies)
