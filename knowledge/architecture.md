# Antevorta Architecture

## Decided

### Backend: Python

**Decision**: Python is the backend language.

**Why**:

- Ecosystem dominance: 7-25x more prediction market code in Python than TypeScript
- Official support: Polymarket's AI agents framework (975 stars) is Python
- SDK health: py-clob-client (480 stars) vs TypeScript client (84 stars)
- Team fit: Strongest language on the team
- LLM integration: Better Claude/LangChain/ML ecosystem

### Job Scheduling: Celery

**Decision**: Celery for background jobs and scheduled tasks.

**Why**:

- Battle-tested for financial applications (Robinhood uses it)
- Team knows it well
- Mature workflow primitives (chains, chords, groups)
- Excellent monitoring with Flower

### Web Framework: Django

**Decision**: Django with Django REST Framework.

**Why**:

- 17+ years Django experience on team
- Built-in admin panel for immediate visibility
- Mature auth system for multi-user dashboard
- Battle-tested ORM and migrations
- Django + Celery is a well-documented pattern

### CLI: Click

**Decision**: Click for command-line interface.

**Why**:

- Pythonic, composable commands
- Team preference for CLI tooling
- CLI provides immediate utility before any UI exists

### Frontend: Next.js (when ready)

**Decision**: Next.js for the eventual web dashboard.

**Why**:

- Best charting libraries (TradingView, Recharts)
- Strong React ecosystem
- If adding JS complexity, go all the way - no half measures (Alpine, etc.)

**Timing**: Build after CLI and API are solid. Django Admin provides visibility in the
interim.

---

## Unresolved

### Platform Focus

**Options**:

1. **Kalshi first** - Underdog volatility strategy
   - Legal in US, no VPN needed
   - Lower volume = more volatility opportunities

2. **Polymarket first** - Copy trading strategy
   - Higher volume, more liquidity
   - Currently requires VPN for US users
   - Better leaderboard data for copy trading

3. **Both from start** - Arbitrage scanning
   - Cross-platform opportunities
   - More complexity upfront

**Decision needed by**: Before building platform integrations.

### User Interface Path

**Options**:

1. **Next.js Dashboard** - Full web application
   - Best for complex visualizations and charting
   - More development investment

2. **Telegram Bot** - Conversational interface
   - Push notifications for alerts and trades
   - Quick commands: `/status`, `/positions`, `/trade`
   - Mobile-first, always available
   - python-telegram-bot is mature and well-maintained
   - Could be the "good enough" UI that makes Next.js unnecessary

3. **Both** - Telegram for notifications/quick actions, Next.js for deep analysis

**Note**: Telegram could slot in between Phase 2 and Phase 3 as a lightweight mobile
interface that provides real value without the JS complexity.

---

## Phased Approach

The API is the product. Everything else is a client.

### Phase 1: CLI + API

Build the foundation. CLI provides immediate utility.

```
antevorta scan          # Find opportunities
antevorta trade         # Execute trades
antevorta status        # Check positions/P&L
antevorta config        # Manage settings
```

CLI calls the same service layer that API exposes. Both are thin wrappers around shared
business logic in `core/`.

### Phase 2: Visibility

- **Django Admin** - Customize for monitoring, position management, user config
- **Flower** - Celery task monitoring
- **Logging/Alerts** - Observability for trading activity

No custom UI yet. Admin is surprisingly powerful when customized.

### Phase 3: Next.js Dashboard

When the family needs a polished interface:

- Consumes Django REST API
- Real-time P&L via WebSocket
- User-specific views and permissions
- Mobile-responsive

CLI and Admin continue working. Nothing thrown away.

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                              Clients                                │
│  ┌──────────┐  ┌──────────────┐  ┌──────────┐  ┌────────────────┐  │
│  │   CLI    │  │ Django Admin │  │ Telegram │  │ Next.js        │  │
│  │  (Click) │  │  (Phase 2)   │  │   Bot    │  │ (if needed)    │  │
│  └────┬─────┘  └──────┬───────┘  └────┬─────┘  └───────┬────────┘  │
└───────┼───────────────┼───────────────┼────────────────┼────────────┘
        │               │               │                │
        ▼               ▼               ▼                ▼
┌─────────────────────────────────────────────────────────────────────┐
│                         Django Backend                              │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                       REST API (DRF)                        │    │
│  └─────────────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                         core/                               │    │
│  │  ┌─────────────┐  ┌──────────────┐  ┌─────────────┐         │    │
│  │  │ strategies/ │  │  platforms/  │  │  models/    │         │    │
│  │  │ (trading)   │  │ (API clients)│  │  (data)     │         │    │
│  │  └─────────────┘  └──────────────┘  └─────────────┘         │    │
│  └─────────────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │               LLM Agents (Claude, Gemini)                   │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────┬───────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        Celery + Redis                               │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────┐  │
│  │ Scheduled scans │  │ Trade execution │  │       Alerts        │  │
│  └─────────────────┘  └─────────────────┘  └─────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────┐
│                         External APIs                               │
│               Polymarket  │  Kalshi  │  Price Feeds                 │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Project Structure (Target)

```
antevorta/
├── cli/                    # Click commands
│   ├── __init__.py
│   └── commands.py
├── core/                   # Business logic (shared)
│   ├── strategies/         # Trading strategies
│   ├── platforms/          # Polymarket, Kalshi clients
│   └── models.py           # Domain models
├── api/                    # Django REST Framework
│   ├── views.py
│   ├── serializers.py
│   └── urls.py
├── tasks/                  # Celery tasks
│   └── trading.py
├── manage.py
└── config/                 # Django settings
    └── settings.py
```

---

## Research References

- [Library analyses](./libraries/) - SDK evaluations
- [Competitor analyses](./competitors/) - What others built
