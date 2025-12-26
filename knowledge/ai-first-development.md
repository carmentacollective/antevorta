# AI-First Development

How trading systems get built in the age of AI. The methodology Antevorta follows.

## The Core Insight

Trading strategies are conversations, not artifacts.

A strategy is an ongoing dialogue between human insight and market reality, continuously
shaped by signals, never finished. The specification is a living model of intent that
evolves as understanding deepens.

Code is derived. The strategy specification is the source of truth. When understanding
changes, implementation follows.

## The Self-Improving Trading Loop

```
Strategy Vision → Specification → Implementation → Execution → Market Signals
                        ↑                                            ↓
                        ←←←←←←←← AI Trading Intelligence ←←←←←←←←←←←←
```

Traditional trading systems iterate over weeks. Backtest, analyze, adjust parameters,
deploy, monitor, repeat. The loop takes days or weeks per adjustment.

AI-first trading compresses this loop to hours:

1. AI agents execute trades, generating performance signals
2. AI analyzes signals and proposes strategy refinements
3. Human approves significant changes; minor tuning proceeds autonomously
4. Loop repeats

The strategies improve while you sleep. Market feedback flows directly into improvement.
Competitor movements are automatically analyzed. Edge compounds.

## What the Specification Contains

The specification captures complete strategy understanding. An AI with no other context
could trade the strategy from this alone.

Why this strategy exists:

- The market inefficiency being exploited
- What conditions make this work
- What success looks like (Sharpe, win rate, drawdown limits)
- What this strategy explicitly avoids

How it behaves:

- Entry and exit conditions
- Position sizing logic
- Risk limits per trade and overall
- Edge cases and unusual market states

What constraints apply:

- Maximum position sizes
- Drawdown limits that trigger pauses
- Platform-specific requirements (Polymarket vs Kalshi)
- Latency and execution requirements

What has been learned:

- Trades that worked and why
- Approaches tried and abandoned
- Market conditions where the strategy underperforms
- Parameter values that evolved through testing

## Strategy Maintenance Through Dialogue

Specification maintenance happens through dialogue. The AI:

Understands context. Reads and internalizes the full strategy specification before
responding. Knows what has been tried, what constraints apply, what edge is being
exploited.

Clarifies before acting. When intent is ambiguous, asks. When a proposed change
conflicts with risk limits, surfaces the conflict.

Proposes changes explicitly. Shows exactly what would change before making changes.
"Proposing to widen entry threshold from 2% to 3% based on 14-day backtest showing
improved Sharpe."

Seeks appropriate approval. Some changes require explicit approval. Others proceed with
notification. The boundary shifts based on observed patterns and risk impact.

Executes completely. Once approved, updates the specification and triggers strategy
reload.

## Signal Processing

All external signals flow through the same dialogue interface:

| Signal               | Potential Specification Update         |
| -------------------- | -------------------------------------- |
| Trade outcome        | Entry/exit logic refinement            |
| Market regime change | Strategy pause or parameter adjustment |
| Platform update      | Integration contract changes           |
| Competitor intel     | New opportunities or edge erosion      |
| Risk event           | Position limit adjustments             |
| Performance decay    | Strategy review or deprecation         |

The AI processes signals by proposing specification updates, following the same approval
flow as human-initiated changes.

## Approval Boundaries

The system learns when to ask and when to proceed.

Always ask:

- Changes to core strategy thesis (what inefficiency we exploit)
- Risk limit modifications
- Adding new trading strategies
- Connecting to new platforms or capital
- Any change that could increase maximum loss

Ask once, then proceed on pattern:

- Parameter tuning within established ranges
- Adding new market categories within existing strategy
- Minor execution improvements

Proceed, notify after:

- Bug fixes where implementation did not match existing spec
- Latency improvements that don't change behavior
- Logging and observability additions

Proceed silently:

- Regeneration from unchanged specification
- Formatting and structural improvements to spec

These boundaries shift based on observed approval patterns. The system learns your risk
tolerance over time.

## What Remains Human

Risk judgment. Knowing how much to bet. The difference between a strategy that works in
backtest and one that survives real markets. AI calculates expected value. Someone
decides acceptable loss.

Market intuition. Understanding when conditions have fundamentally changed. When a
strategy that worked is now arbitraged away. AI detects performance decay. Someone
recognizes regime change.

Capital allocation. How much of your bankroll goes to this system. AI optimizes within
its scope. Someone owns the portfolio decision.

Relationships and access. Platform relationships, regulatory compliance, access to
capital. These are durably human.

## The Self-Improving Layer

Two components enable self-improvement:

Trading Intelligence: The AI analyst. Processes trade outcomes, analyzes market
conditions, synthesizes insights into specification updates. Maintains the knowledge
base. Proposes improvements continuously.

Backtesting Agents: Synthetic markets that stress test strategies. Generate performance
signals across historical conditions. Find edge cases humans miss. Run continuously, not
just before deployment.

Together they create the flywheel: agents test, AI analyzes, strategies refine, agents
test again. The cycle that compresses weeks into hours.

## Evolving Strategies

Static strategies freeze at version one. Production trading needs strategies, memories,
and parameters that update through execution feedback.

This happens in the specification layer, not the model weights.

How Strategies Evolve

Small structured increments that sharpen edge instead of overwriting it:

- After a trading session, AI proposes refinements based on outcomes
- Learnings stored as structured updates (not free-form notes)
- Parameters sharpen based on what worked and what didn't
- Strategies learn from doing, not from human tinkering

What Gets Updated

Strategies: Which approaches work for which market conditions. Pattern-matched solutions
accumulate.

Heuristics: Rules of thumb that improve over time. "When volume drops below X, widen
spreads."

Market knowledge: Accumulated understanding of platforms, common patterns, edge case
behaviors.

Error patterns: What failed before and how to avoid it. Self-debugging improves as the
system encounters and resolves issues.

Constraints on Self-Modification

Agents can update their:

- Approach within defined strategy scope
- Market-specific knowledge
- Tactical decision-making
- Execution patterns

Agents cannot update:

- Core identity or values
- Risk limits
- Strategy boundaries or thesis
- Platform integration contracts
- Approval boundaries

The system learns your judgment over time - what to ask about, what to proceed with, how
you prefer things done. This learning is scoped appropriately so agents don't drift into
risk decisions requiring human judgment.

## Implementation in Antevorta

Antevorta is built this way. The `/knowledge` directory is the specification. Code is
generated from it. The specification is the IP.

Current strategies documented:

- [Initial Context](./initial-context.md) - Project overview and team
- [Competitors](./competitors/) - Analysis of other prediction market bots

Strategies to spec:

- Volatility Trading - Kenny's approach to low-volume underdog markets
- Copy Trading - Torin's leaderboard mirroring bot
- Arbitrage Scanner - Cross-platform odds exploitation

The strategies specified in `/knowledge` define what the trading system does. The Python
code in the codebase implements those specifications. When specifications change,
implementation follows.

## The Vision

Strategies become organisms, not artifacts.

The team is:

- Human(s) with market insight and risk judgment
- AI analyst processing all trading signals
- AI implementing strategy refinements
- AI testing continuously against historical and simulated markets

The specification is not a markdown file. It is a living model of trading intent,
maintained by the system, viewable as documents but not limited to them.

Human creative capacity to imagine what edge exists becomes the constraint. The
translation to running trading code is nearly instantaneous. Strategies converge toward
profitability automatically.

The gap between idea and execution is asymptotically reaching zero.
