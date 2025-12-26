# Plan

## Review

Walk through what exists before building:

- [ ] [Strategy: Underdog Volatility](./strategies/underdog-volatility.md) - Kenny's
      approach
- [ ] [Platform: Kalshi](./platforms/kalshi.md) - API, auth, rate limits
- [ ] [Architecture](./architecture.md) - Decisions made (Python, Django, Celery)
- [ ] [Competitors](./competitors/) - What others have built
- [ ] Repo tooling - Linting, formatting, pre-commit

## Build: Underdog Volatility Bot

### Account Setup

- [ ] Kalshi account created
- [ ] KYC verified
- [ ] Account funded
- [ ] API keys generated and private key downloaded

### Kalshi API

- [ ] Authenticate successfully
- [ ] Get list of events (sports)
- [ ] Get markets for an event
- [ ] Get orderbook for a market (bid/ask depth at each price)
- [ ] Get account balance
- [ ] Get current positions
- [ ] Place a limit order
- [ ] Cancel an order
- [ ] Check if order filled

### Strategy: Find Opportunities

- [ ] Filter to heavy underdogs (1-4% odds / 1-4 cents)
- [ ] Filter to low volume markets ($30K-$300K)
- [ ] Filter to pregame only
- [ ] Detect price oscillation (has the price bounced between floor/ceiling?)
- [ ] Kenny validates: does this find the same opportunities he finds manually?

### Strategy: Entry

- [ ] Calculate floor price from recent history
- [ ] Place limit buy at floor
- [ ] Kenny validates: are these the entries he would make?

### Strategy: Exit

- [ ] When buy fills, immediately place limit sell at target (50-100% gain)
- [ ] If game starting soon and position hasn't moved, exit at cost
- [ ] Kenny validates: does this match his exit approach?

### Strategy: Risk

- [ ] Never hold through game start
- [ ] Position size limits defined
- [ ] What to do if API fails mid-trade
- [ ] What to do if can't exit (no liquidity)

### CLI (Click)

`antevorta scan` - Find opportunities

- List markets matching underdog criteria
- Show: ticker, current price, floor/ceiling range, volume, game start time
- Highlight best opportunities (widest spread, most volatility)

`antevorta status` - Check positions and P&L

- Open positions with entry price, current price, unrealized P&L
- Pending orders (unfilled buys/sells)
- Account balance

`antevorta buy <ticker> --price <cents> --shares <n>` - Place limit buy

- Confirm before placing
- Show order ID and status

`antevorta sell <ticker> --price <cents> --shares <n>` - Place limit sell

- Confirm before placing
- Show order ID and status

`antevorta cancel <order-id>` - Cancel pending order

`antevorta watch` - Live monitoring (stretch)

- Stream position updates
- Alert when orders fill
- Warn when games approaching start time

## Roadmap

- Copy trading strategy (Polymarket)
- Arbitrage scanner (cross-platform)
- Production infrastructure
- Multi-account support
