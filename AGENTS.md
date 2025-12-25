# Antevorta

AI-driven trading bot infrastructure for prediction markets (Polymarket, Kalshi).

## Always Apply Rules

@.cursor/rules/heart-centered-ai-philosophy.mdc
@.cursor/rules/trust-and-decision-making.mdc
@.cursor/rules/code-style-and-zen-of-python.mdc @.cursor/rules/ruff-linting.mdc
@.cursor/rules/external-apis.mdc @.cursor/rules/git-interaction.mdc
@.cursor/rules/prompt-engineering.mdc

## Project Type

Python project using LLM agents as autonomous decision-makers for trading strategies.

## Architecture

- **LLM Agents** - Claude Opus 4.5, Gemini, GPT for trading decisions
- **MCP Servers** - Platform integrations for Polymarket and Kalshi APIs
- **Bot Infrastructure** - Execution layer for automated trading

## Strategies

1. **Volatility Trading** - Exploit price swings in low-volume sports markets
2. **Copy Trading** - Mirror top Polymarket performers (~4s latency)
3. **Arbitrage Scanner** - Cross-platform odds mismatches (2-5% inefficiencies)

## Philosophy

Give goals, not steps. Let the LLM work. Whatever LLM is reading this is smarter than
the instructions you'd write.

## Team

- Nick Sullivan - Architecture & AI integration
- Kenny Sullivan - Strategy validation
- Torin - Bot development
- Conor Sullivan - Risk management
- Anna Sullivan - Coordination

## Git Workflow

Commit format: `emoji Type: description` (e.g., `✨ Add volatility trading strategy`)

Never commit to main without explicit permission. Never use `--no-verify`.

## Context Management

### Compaction Strategy

When compacting this session, prioritize:

- Trading strategy logic and decision thresholds
- API integration state (Polymarket, Kalshi connections)
- Bot execution status and any errors
- Risk management parameters and position limits
- Outstanding issues or TODOs

Manual `/compact` at task boundaries rather than letting auto-compact interrupt flow.

### Preserve Development Environment State

When compacting, preserve the current working state:

- **Git context**: Current branch name, which worktree you're in, any uncommitted
  changes
- **Running processes**: Bot instances, API connections, background jobs
- **Environment**: Paper trading vs live mode, API keys loaded, which exchange endpoints
- **Bot state**: Active positions, pending orders, P&L for session
- **Test state**: Last test run, any failures, mock vs live API mode

### Session Boundaries

One focused task per session. For complex features:

- Start fresh session for that feature
- Run 45-90 minutes to natural checkpoint
- Manual `/compact` at 60%+ usage (check with `/cost`)
- Begin next phase with compacted context
- New session for unrelated work
