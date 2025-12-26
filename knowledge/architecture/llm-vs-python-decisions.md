# LLM vs Python Decision Boundaries

Where AI judgment adds value vs where deterministic code is better.

## Core Principle

**LLM as analyst, Python as executor.**

LLMs run periodically to find/score opportunities and set parameters. Python handles the
fast loop of order management. This matches the strategy's natural rhythm—opportunities
develop over hours, but execution needs to be responsive.

## Pure Python (Deterministic, Fast, Testable)

These are math and API calls. No judgment needed.

### Filtering Mechanics

- Price range checks (1-4¢)
- Volume thresholds ($30K-$300K)
- Time to game start
- Position size limits
- Fee calculations

### Execution

- Order placement/cancellation
- Fill detection
- Position tracking
- P&L accounting
- Time-based exit triggers

## LLM Decision Candidates

### 1. Volatility Pattern Recognition ⭐

> "Detect price oscillation (has the price bounced between floor/ceiling?)"

Pure Python needs rigid rules (X bounces in Y hours). An LLM assesses: "Is this
tradeable oscillation or just noise?" Kenny's eye does this intuitively—pattern
recognition, not a formula.

### 2. Order Book Interpretation ⭐

> "Large pending orders that will move price", "thin liquidity", "large seller at
> target"

Reading order books for _meaning_ vs just reading numbers. An LLM says "this book looks
healthy for exit" vs "there's a wall at 2¢ that'll eat your sell."

### 3. Floor/Ceiling Estimation

> "Calculate floor price from recent history"

What's the _actual_ floor from noisy data? Is 1¢ an outlier or the real floor? Judgment.

### 4. Opportunity Ranking

When 40 markets qualify, which 5 are best? "Widest spread, most volatility" could be
rules, but Kenny has intuition about which _type_ of volatility is exploitable.

### 5. Edge Case Handling

> "What to do if can't exit (no liquidity)"

Unusual situations benefit from reasoning, not predetermined branches.

## Visual Technical Analysis

Kenny's manual process is inherently visual—scrolling through markets on his phone,
looking at patterns. We replicate his workflow by feeding charts to vision models.

### What to Visualize

**Price History Chart** - The core oscillation pattern:

```
Price
  4¢ ┤    ╭─╮      ╭─╮      ╭─╮
  3¢ ┤   ╱   ╲    ╱   ╲    ╱   ╲
  2¢ ┤  ╱     ╲  ╱     ╲  ╱     ╲
  1¢ ┤─╯       ╰╯       ╰╯       ╰─
     └─────────────────────────────▶ Time
```

LLM sees: "Clear oscillation, floor at 1¢, ceiling at 4¢, good entry at floor."

**Order Book Depth Chart** - Liquidity at each price level:

```
       BIDS          │         ASKS
◀━━━━━━━━━━━━━━━━━━━━┿━━━━━━━━━━━━━━━━━━━▶
████████████         │  1¢
██████               │  2¢  ████████████████  ← wall
████                 │  3¢  ██████
██                   │  4¢  ████
```

LLM sees: "Wall at 2¢ on ask side—exit might get stuck there."

**Volume/Activity Over Time** - When is the market active? Volume trends toward game?

### Kenny's Process → Visual LLM

| Kenny's Manual Process         | Visual LLM Equivalent             |
| ------------------------------ | --------------------------------- |
| Scrolls through markets on app | Batch-generates charts            |
| "Looks at" price patterns      | Feeds price chart to vision model |
| "Checks" order book            | Feeds depth chart                 |
| Makes gut call                 | Gets structured assessment        |

### Multi-Chart Batch Analysis

Composite multiple opportunities for ranking:

```
┌─────────────┬─────────────┬─────────────┐
│  NCAAB-A    │  NCAAB-B    │  CSGO-X     │
│  [chart]    │  [chart]    │  [chart]    │
│  ✓ Good     │  ⚠ Thin     │  ✓ Great    │
└─────────────┴─────────────┴─────────────┘
```

"Rank these opportunities. Which would you enter first and why?"

## Architecture

```
┌─────────────────────┐     ┌──────────────────────┐
│   LLM Analysis      │     │  Python Execution    │
│   (async, batched)  │────▶│  (fast, deterministic)│
└─────────────────────┘     └──────────────────────┘
         │                            │
    "This market                 Place order at 1¢
     looks good,                 Cancel if not filled
     floor is 1¢,                by T-30min
     target 2¢"
```

## Validation Approach

Have the LLM analyze a batch of markets Kenny already evaluated manually. Does it agree
with his picks? If it replicates his judgment, we've captured the edge. If not, the edge
is either simpler (pure rules) or more complex (needs refinement).

## Considerations

**For LLM decisions:**

- Latency acceptable (opportunity discovery, not execution)
- Cost per analysis (batch to reduce calls)
- Reasoning transparency (why did it pick this market?)

**Against LLM for some things:**

- Execution speed (1-10s latency matters for order management)
- Determinism (Python is predictable and testable)
- If Kenny can articulate exact rules, maybe it's not judgment

## Rendering Stack

Simple matplotlib—doesn't need to be pretty, just informative:

```python
plt.figure(figsize=(10, 4))
plt.plot(timestamps, prices)
plt.axhline(y=1, color='g', linestyle='--', label='Floor?')
plt.axhline(y=4, color='r', linestyle='--', label='Ceiling?')
plt.savefig(buffer, format='png')
```

## Open Questions

- What's the right cadence for LLM analysis? Every 5 min? On-demand?
- How to handle disagreement between LLM assessment and Kenny's intuition?
- Can we capture Kenny's reasoning in prompts, or does it need to be learned from
  examples?
