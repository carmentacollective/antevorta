# Copy Trading Strategy

Mirror trades of top-performing Polymarket users in near-real-time to capture their edge
with acceptable slippage.

## The Core Insight

Polymarket makes all trades public. You can see exactly what every user buys and sells,
including the most profitable traders. Some traders have sophisticated algorithms,
insider knowledge, or exceptional intuition. By copying their trades with minimal delay,
you capture most of their edge.

The challenge is latency. If someone buys at 50¢ and the price immediately jumps to 55¢,
you need to decide if 55¢ is still a good entry. A 2% slippage tolerance accounts for
this while still capturing profitable trades.

## Strategy Mechanics

Target selection:

- Top 5 users by profit today
- Top 5 users by profit this week
- Overlap between lists indicates consistency
- Avoid one-hit wonders with single large win

Trade mirroring:

- When target user buys, bot buys 100 shares
- When target user sells (any percentage), bot sells same percentage
- When market closes, bot position resolves with market
- Track each position separately by user + market

Slippage handling:

- Add 2% to target's buy price as tolerance
- If current price > target price + 2%, skip the trade
- If current price > target price + 5%, definitely skip
- Minimum price threshold: avoid 1¢ shares (Kenny's edge)

## Leaderboard Mechanics

Polymarket leaderboard API:

- Not officially documented (discovered via browser inspection)
- Returns users sorted by profit (daily, weekly, monthly, all-time)
- Each user has public trade history
- Update frequency: near-real-time

Key metrics to track:

- Profit (absolute dollars)
- Win rate (percentage of winning trades)
- Volume (total dollars traded)
- Recency (when was last trade)

Filtering logic:

- High profit + high win rate = follow
- High profit + low win rate = might be whale, verify
- Low volume + high win rate = might be lucky, need more data

## Current Implementation (Torin's Bot)

Infrastructure:

- Raspberry Pi running 24/7
- Python script polling Polymarket API
- CSV logging for all simulated trades
- 4-second maximum latency (10 users × ~0.4s per API call)

Configuration:

- Following top 5 daily + top 5 weekly = 10 users max
- 100 shares per trade
- 2% slippage tolerance
- Minimum buy price: 5¢ (avoids 1-2¢ volatility plays)

Early results (December 2024):

- Simulated profit: $52 in first 24 hours
- 3 closed markets, 2 losses (slippage issue), 1 big win
- After minimum price fix, profitability improved

## Latency Optimization

Every second of latency costs money. Current bottleneck is API rate limits.

Current state:

- ~1 second per API call
- 10 users = ~10 seconds per full scan
- Reduced from 400 users (170 seconds) to 10 users (4 seconds)

Optimization opportunities:

- Parallel API calls (if rate limits allow)
- WebSocket connections (if available)
- Caching user trade history, only checking for new trades
- Prioritizing most active users

Target latency: < 4 seconds from target trade to bot trade

## Platform: Polymarket Only

Why Polymarket for this strategy:

- All trades are public (Kalshi trades are private)
- Leaderboard exists with top performers
- API access available
- Higher volume = easier to fill orders
- More diverse markets (politics, crypto, sports, events)

Polymarket limitations:

- Currently requires VPN + crypto for US users
- Expected to return to US legally in 2025
- Higher fees than Kalshi for takers

## Risk Factors

Following losers: The leaderboard shows recent winners, but some are lucky, not skilled.
A trader who made $50K today on a single bet might lose it tomorrow.

Mitigation: Require consistency across daily AND weekly leaderboard. High win rate, not
just high profit.

Front-running: If many bots copy the same traders, the first trade moves the price and
later copies get worse fills.

Mitigation: Accept some slippage as cost of doing business. 2% tolerance handles this.

Whale behavior: Large traders might have different risk tolerance. They can afford to
lose $10K on a trade; you might not want to follow that position sizing.

Mitigation: Fixed 100-share position size regardless of target's bet size.

## Gabagool Case Study

Notable Polymarket trader (username: Gabagool):

- 90% win rate on Bitcoin price prediction
- Exclusively trades 15-minute crypto markets
- Made $300K+ since October 2024
- Trades both YES and NO (hedging/arbitrage)
- Bot runs 24/7, never stops

Strategy appears to be:

- Arbitrage between YES and NO when combined price < 100¢
- Sophisticated algorithm predicting short-term crypto moves
- High-frequency trading with minimal slippage

Copying Gabagool specifically:

- His trades are in very short-term markets (15 min)
- Latency is critical - 4 seconds might be too slow
- His edge might require sub-second execution
- Better to understand his approach than blindly copy

## Bot Implementation Notes

Core loop:

1. Fetch top N users from daily and weekly leaderboard
2. For each user, check recent trades
3. If new buy detected, execute buy with slippage tolerance
4. Track all open positions per user + market
5. Monitor for sells and market closes
6. Log all activity for analysis

Data to capture:

- User ID, username, profit metrics
- Market ID, question, current price
- Target's entry price vs bot's entry price (slippage)
- Exit price, exit type (user sold vs market closed)
- P&L per trade

Scaling approach:

- Start with simulation (fake money, real trades)
- Validate profitability over 1+ weeks
- Gradually increase position size
- Add more users if profitable
