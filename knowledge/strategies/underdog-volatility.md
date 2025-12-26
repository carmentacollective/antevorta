# Underdog Volatility Strategy

Exploit price swings in low-volume prediction markets by buying heavy underdogs at
volatility lows and selling at highs, all before game resolution.

## The Core Insight

Prediction market odds for heavy underdogs fluctuate wildly in low-volume markets, even
though the "true" probability (sportsbook odds) stays constant. This volatility is
caused by:

1. Low liquidity amplifying order impact
2. Kalshi's "buy in dollars" feature filling at bad prices
3. Retail traders not using limit orders
4. Information asymmetry between platforms

A 1% underdog might oscillate between 1% and 4% all day pregame. Buying at 1% and
selling at 2% doubles your money regardless of game outcome.

## Strategy Mechanics

Entry criteria:

- Heavy underdog (1-4% sportsbook odds)
- Low market volume ($30K-$300K range)
- Visible volatility pattern over past 24-48 hours
- Clear oscillation between floor and ceiling (e.g., 1% ↔ 4%)
- Pregame only (never enter during live game)

Entry execution:

- Use limit orders exclusively (maker = zero fees)
- Place buy orders at the floor of observed range
- "Hopeful" orders at 1¢ often fill when retail traders dump
- Never use "buy in dollars" - always specify share count and price

Exit criteria:

- Target: 50-100% gain (1¢ → 2¢ or 2¢ → 4¢)
- Immediately place limit sell order after buy fills
- Exit before game starts if position hasn't moved
- Break-even exit if odds flatten and won't recover

Risk management:

- Floor is 1¢ - can always sell at 1¢ pregame
- Never hold through game resolution
- If odds collapse near game time, exit at cost basis
- Maximum loss = cost of shares (known upfront)

## Informational Edge

The strategy has additional alpha from information asymmetry:

Kalshi score lag: Kalshi's live scores often lag 10-30 minutes behind source feeds (NCAA
website, ESPN). If you see a team scoring well on the source feed while Kalshi still
shows old score, odds will rise when Kalshi catches up.

Esports text feeds: For esports events, text feeds (HLTV for Counter-Strike) are often
2+ minutes ahead of video streams. Bettors watching video don't know what just happened.

Low-volume obscurity: Chinese Basketball Association, women's college basketball, small
esports tournaments - fewer eyes means more inefficiency.

## Ideal Markets

College basketball:

- 25+ games daily with massive underdogs
- Women's college basketball especially inefficient
- Games start throughout day, constant opportunities
- Volume typically $100K-$300K per game

Esports:

- Counter-Strike, Valorant, League of Legends
- Smaller tournaments have lowest volume
- Text feeds provide information edge
- Rich enthusiasts (Faze Clan fans) create price spikes

## Platform: Kalshi

Why Kalshi over Polymarket for this strategy:

- Legal in US (no VPN/crypto needed)
- Sports markets available
- Trades are private (competitors can't see your orders)
- Simple limit order interface

Kalshi limitations:

- Only whole-number percentages (no 1.5¢)
- Score feed often inaccurate
- Lower volume than Polymarket

## Order Book Reading

The order book is more valuable than price charts for this strategy.

What to look for:

- Depth at each price level (how many shares available)
- Gaps between bid and ask
- Large pending orders that will move price when filled
- Thin order books = bigger swings possible

Warning signs:

- Very thin liquidity (can't exit position)
- Large seller sitting at your target exit price
- Order book suddenly filling up near game time

## Validated Performance

Kenny's results (December 2024):

- Starting capital: $5
- Current balance: $2,600+
- Win rate: 58 wins, 3 break-evens, 1 loss
- Strategy: Manual execution on Kalshi mobile app
- Time horizon: ~3 weeks

Clara's results:

- Starting capital: $100
- Current balance: $350+
- Strategy: Following Kenny's approach
- Time horizon: ~3 days

## Scaling Considerations

The strategy works at current scale but has liquidity limits:

Current comfort zone: $100-$500 per position

Theoretical limit: ~$10,000 per game before moving the market yourself

Scaling approach:

- Spread across multiple games simultaneously
- 40+ qualifying games per day = $400K+ daily capacity
- Never concentrate in single market

## Bot Implementation Notes

For automation, the bot needs to:

1. Scan all markets for underdog + low volume + volatility pattern
2. Place limit buy orders at floor price
3. Immediately place limit sell orders when buys fill
4. Monitor game start times and exit positions before tip-off
5. Cancel unfilled orders when games start
6. Track P&L per market for strategy refinement

API considerations:

- Kalshi API requires authentication
- Rate limits apply (plan for ~1 second per call)
- Order book data available via API
- Game start times available for scheduling
