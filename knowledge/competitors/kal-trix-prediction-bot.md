# Kal-trix-prediction-bot - Production-Grade Multi-Model Sentiment Trading

**Repository**: [LoQiseaking69/Kal-trix-prediction-bot](https://github.com/LoQiseaking69/Kal-trix-prediction-bot)
**Status**: Active (production-ready)
**Markets**: Kalshi
**Strategy**: Multi-model sentiment analysis + statistical arbitrage
**Local Clone**: `../reference/Kal-trix-prediction-bot`

## Overview

Comprehensive production-ready trading bot featuring advanced AI-powered sentiment analysis using multiple transformer models (BERT, RoBERTa, FinBERT), statistical arbitrage strategies, and sophisticated risk management. Includes modern React admin panel, Telegram integration, and full Docker containerization.

**Key Stats**:

- **Codebase**: 2,676 lines of core strategy/ML code (excluding UI)
- **Sentiment Models**: 6 models (FinBERT, RoBERTa, BERT, VADER, TextBlob, custom RF)
- **Architecture**: Flask backend + React frontend + Telegram web app
- **Deployment**: Complete Docker Compose stack with Nginx reverse proxy

## Strategy: Advanced Sentiment Analysis with Multi-Model Ensemble

### Multi-Model Sentiment Architecture

**Six-Model Ensemble**:

1. **FinBERT** (ProsusAI/finbert) - Financial domain-specific BERT
2. **RoBERTa** (cardiffnlp/twitter-roberta-base-sentiment) - Social media optimized
3. **BERT** (bert-base-uncased) - General language understanding
4. **VADER** - Rule-based lexicon for social media
5. **TextBlob** - Simple polarity scoring
6. **Custom RandomForest** - Domain-specific classifier

**Weighted Ensemble Scoring**:

```python
# Dynamic weight adjustment based on recent performance
weights = {
    'finbert': 0.3,    # Highest weight for financial markets
    'roberta': 0.25,   # Social media sentiment
    'bert': 0.2,       # General context
    'vader': 0.15,     # Quick polarity
    'textblob': 0.05,  # Baseline
    'custom': 0.05     # Domain-specific
}

ensemble_score = sum(weights[model] * scores[model] for model in models)
confidence = np.std([scores[model] for model in models])  # Lower std = higher confidence
```

### Sentiment-Driven Signal Generation

**Advanced Sentiment Strategy** (backend/strategies/advanced_sentiment_strategy.py):

```python
class AdvancedSentimentStrategy:
    def __init__(self):
        self.min_sentiment_threshold = 0.6
        self.min_confidence_threshold = 0.7
        self.sentiment_momentum_window = 6  # hours
        self.volume_threshold_multiplier = 1.5
        self.max_position_correlation = 0.8
```

**Trading Logic**:

1. **Keyword Extraction**: Extract market-relevant keywords from title/subtitle
2. **Multi-Source Data**: Gather news articles, social media, market commentary
3. **Model Inference**: Run text through all 6 sentiment models
4. **Ensemble Aggregation**: Weighted average with confidence scoring
5. **Momentum Analysis**: Track sentiment trend over 6-hour window
6. **Volume Confirmation**: Require 1.5x average volume for signal
7. **Correlation Check**: Avoid redundant positions (max 0.8 correlation)
8. **Signal Filtering**: Only trade if sentiment > 0.6 AND confidence > 0.7

**Sentiment Momentum**:

```python
def calculate_sentiment_momentum(self, market_id: str) -> float:
    """Calculate sentiment trend strength over time window"""
    recent_sentiments = self.get_recent_sentiment_data(market_id, hours=6)

    if len(recent_sentiments) < 3:
        return 0.0

    # Linear regression on sentiment scores over time
    timestamps = [s['timestamp'] for s in recent_sentiments]
    scores = [s['sentiment'] for s in recent_sentiments]

    momentum = np.polyfit(timestamps, scores, 1)[0]  # Slope = momentum
    return momentum
```

### Statistical Arbitrage Strategy

**Correlation-Based Mean Reversion**:

```python
class StatisticalArbitrageStrategy:
    def __init__(self):
        self.min_correlation = 0.7
        self.zscore_threshold = 2.0
        self.lookback_days = 30
        self.min_data_points = 20
```

**Pair Identification**:

1. Calculate price correlation between all market pairs
2. Filter for correlation > 0.7
3. Calculate historical price spread
4. Compute Z-score of current spread vs historical
5. Trade when |Z-score| > 2.0 (price relationship has diverged)

## Architecture

### Tech Stack

**Backend (Flask + SQLAlchemy)**:

- Python 3.9+
- Flask REST API
- SQLite database
- Real-time WebSocket service
- Background job scheduler

**Frontend (React + Vite)**:

- Modern React 18
- Vite for fast builds
- Recharts for visualizations
- Socket.IO client for real-time updates

**Deployment**:

- Docker + Docker Compose
- Nginx reverse proxy
- Redis caching (planned)
- Health check monitoring

**Project Structure**:

```
backend/
├── api/                      # Kalshi API client with HMAC auth
├── core/                     # Trading engine, config, database
├── ml_models/               # Sentiment analyzer (6 models)
├── strategies/              # Sentiment + arbitrage strategies
├── risk_management/         # Portfolio risk manager
└── services/                # WebSocket, real-time data

desktop_admin_panel/
├── src/
│   ├── components/          # Dashboard, Markets, Portfolio, Orders, Bot Control
│   └── App.jsx             # Main React app
└── vite.config.js          # Vite bundler config

telegram_webapp/
└── app.py                   # Telegram mini-app (Flask)
```

### Kalshi API Integration

**HMAC-SHA256 Authentication**:

```python
def _sign_request(self, method: str, path: str, timestamp: str) -> str:
    """Sign request with HMAC-SHA256 for Kalshi API"""
    payload = f"{timestamp}{method}{path}".encode('utf-8')

    signature = hmac.new(
        self.api_secret.encode('utf-8'),
        payload,
        hashlib.sha256
    ).hexdigest()

    return signature
```

**Enhanced Client** (backend/api/enhanced_kalshi_client.py):

- Proper authentication with signature generation
- Fallback to demo data when API unavailable
- Rate limiting awareness
- Error handling with retries
- Comprehensive logging

## Risk Management: Portfolio-Level Controls

### Sophisticated Risk System (backend/risk_management/portfolio_manager.py)

**RiskMetrics Dataclass**:

```python
@dataclass
class RiskMetrics:
    total_exposure: float
    max_position_size: float
    portfolio_correlation: float
    var_95: float  # Value at Risk 95%
    expected_shortfall: float
    sharpe_ratio: float
    max_drawdown: float
    daily_pnl_volatility: float
```

**Risk Parameters**:

```python
class PortfolioRiskManager:
    def __init__(self):
        self.max_portfolio_exposure = 0.8  # Max 80% bankroll at risk
        self.max_single_position = 0.15    # Max 15% per position
        self.max_correlation = 0.8         # Limit correlated positions
        self.var_confidence = 0.95         # 95% VaR
        self.lookback_days = 30            # Historical window
```

**Advanced Features**:

1. **Value at Risk (VaR) Calculation**:
   - Monte Carlo simulation for tail risk
   - 95% confidence interval
   - Expected Shortfall (CVaR) for extreme scenarios

2. **Correlation Monitoring**:
   - Real-time correlation matrix between positions
   - Dynamic correlation limits
   - Concentration risk alerts

3. **Position Sizing**:
   - Kelly Criterion optimization (planned)
   - Volatility-based sizing
   - Correlation-adjusted sizing

4. **Stop-Loss Mechanisms**:
   - Individual position stop-losses (15% default)
   - Portfolio-level drawdown limits (20% max)
   - Time-based exits for stale positions

5. **Risk Levels**:
   - LOW, MEDIUM, HIGH, CRITICAL risk states
   - Dynamic trading restrictions based on risk level
   - Automatic de-risking when CRITICAL

## Code Quality

### Strengths

- ✅ Clean modular architecture with single responsibility
- ✅ Comprehensive type hints throughout
- ✅ Detailed docstrings explaining complex logic
- ✅ Structured logging at every layer
- ✅ Production-ready Docker deployment
- ✅ Professional React UI with real-time updates
- ✅ Extensive configuration management

### Weaknesses

- ❌ **Limited test coverage**: No visible test files
- ❌ **Mock data fallback**: Uses demo data when API fails (good for testing, but needs clear separation)
- ❌ **Hardcoded thresholds**: Many magic numbers (0.6, 0.7, 1.5, etc.) should be configurable
- ❌ **No backtesting framework**: No historical simulation for strategy validation
- ❌ **Missing CI/CD**: No automated testing or deployment pipeline

### Documentation

**Excellent** - 5 comprehensive markdown files:

- README.md (1,152 lines)
- ARCHITECTURE.md
- API_DOCUMENTATION.md
- DEPLOYMENT_GUIDE.md
- ENHANCED_DEPLOYMENT_GUIDE.md

## AI Integration

### State-of-the-Art NLP

**Transformer Models**:

```python
# FinBERT - Financial sentiment
self.finbert_tokenizer = AutoTokenizer.from_pretrained("ProsusAI/finbert")
self.finbert_model = AutoModelForSequenceClassification.from_pretrained("ProsusAI/finbert")

# RoBERTa - Social media sentiment
self.roberta_tokenizer = AutoTokenizer.from_pretrained("cardiffnlp/twitter-roberta-base-sentiment")
self.roberta_model = AutoModelForSequenceClassification.from_pretrained("cardiffnlp/twitter-roberta-base-sentiment")

# BERT - General language understanding
self.bert_tokenizer = BertTokenizer.from_pretrained('bert-base-uncased')
self.bert_model = BertForSequenceClassification.from_pretrained('bert-base-uncased')
```

**GPU Acceleration**:

```python
device = 0 if torch.cuda.is_available() else -1
pipeline("sentiment-analysis", model=model, tokenizer=tokenizer, device=device)
```

### Custom Financial Keyword Lexicons

**Domain-Specific Sentiment**:

```python
positive_keywords = {
    'financial': ['profit', 'gain', 'growth', 'increase', 'rise', 'bull'],
    'political': ['victory', 'win', 'lead', 'ahead', 'support', 'popular'],
    'general': ['optimistic', 'confident', 'promising', 'bright']
}

negative_keywords = {
    'financial': ['loss', 'decline', 'fall', 'drop', 'bear', 'weak'],
    'political': ['defeat', 'lose', 'behind', 'scandal', 'controversy'],
    'general': ['pessimistic', 'worried', 'concerning', 'dark']
}
```

## Unique Innovations

### 1. Six-Model Sentiment Ensemble 🔥

**Novel aspect**: Combines 3 transformer models + 3 traditional models with dynamic weighting.

Most bots use a single model or simple averaging. This bot:

- Uses domain-specific models (FinBERT for finance)
- Weights models by historical accuracy
- Calculates confidence from ensemble disagreement (low std = high confidence)
- Adapts weights based on recent performance

### 2. Sentiment Momentum Analysis 🔥

**Novel aspect**: Tracks sentiment trend over 6-hour window using linear regression.

```python
momentum = np.polyfit(timestamps, scores, 1)[0]  # Slope of sentiment over time

if momentum > 0 and latest_sentiment > 0.6:
    signal_strength = sentiment * (1 + momentum)  # Amplify with momentum
```

A rising sentiment trend is more valuable than static high sentiment.

### 3. Volume-Confirmed Sentiment 🔥

**Novel aspect**: Requires 1.5x average volume to validate sentiment signals.

Prevents false signals from low-volume noise:

```python
if current_volume < avg_volume * 1.5:
    return None  # Reject signal despite good sentiment
```

### 4. Correlation-Aware Position Sizing 🔥

**Novel aspect**: Reduces position size for highly correlated markets.

```python
portfolio_correlation = calculate_correlation_with_existing_positions(market)
max_size = base_size * (1 - portfolio_correlation)  # Scale down if correlated
```

Prevents concentration risk from multiple correlated bets.

### 5. Real-Time Web Interface with WebSockets 🔥

**Novel aspect**: Professional React dashboard with live data updates.

Most prediction market bots are CLI-only. This has:

- Real-time P&L charts
- Live market data
- Bot control interface
- Risk metrics dashboard
- Mobile-responsive design

### 6. Production-Ready Docker Stack 🔥

**Novel aspect**: Complete containerized deployment with one-command setup.

```bash
./install.sh  # Interactive setup wizard
docker-compose up -d  # Full stack running
```

Includes:

- Nginx reverse proxy
- Health check endpoints
- Database migrations
- Secure secrets management

## What to Learn

**Steal These Patterns**:

1. **Multi-model ensemble** - Don't rely on single sentiment model
2. **Momentum analysis** - Sentiment trend matters more than point-in-time
3. **Volume confirmation** - Validate signals with market activity
4. **Correlation-aware sizing** - Adjust position size based on portfolio correlation
5. **Production architecture** - Docker, health checks, monitoring from day one
6. **Real-time UI** - Professional dashboard makes bot accessible
7. **Comprehensive docs** - 5 detailed markdown files covering all aspects

**Avoid**:

1. **Hardcoded thresholds** - Make magic numbers configurable
2. **No backtesting** - Add historical simulation framework
3. **Limited tests** - Need unit/integration/e2e test coverage
4. **Mock data mixing** - Separate demo mode from production clearly
5. **Manual deployment** - Add CI/CD pipeline

**Architecture Lessons**:

- **Modular design scales** - Clean separation enables easy extension
- **Type hints matter** - Makes codebase maintainable
- **Multiple UIs** - Desktop + mobile + Telegram = broad accessibility
- **Risk management is not optional** - Portfolio-level controls from day one
- **Monitoring built-in** - Health checks, logging, metrics from start

## Philosophical Differences

**Kal-trix-prediction-bot**:

- Multi-model AI ensemble as primary strategy
- Sentiment + arbitrage hybrid approach
- Production-first mindset (Docker, docs, UI)
- Comprehensive risk management
- Kalshi-only focus

**Antevorta**:

- LLM agents as decision-makers
- Multiple market support (Polymarket + Kalshi)
- Experimental and adaptive
- Simpler initial architecture

## Verdict

**Production-grade sentiment trading system** with sophisticated multi-model AI, professional UI, and comprehensive risk management. This is how you build a trading bot for actual deployment.

The code quality is high, architecture is sound, and documentation is excellent. Missing elements are testing and CI/CD, not core functionality.

**Rating**: ⭐⭐⭐⭐ (4/5)
**Maturity**: Production (needs tests and CI/CD)
**Code Quality**: High
**Innovation**: Very High (6-model ensemble, momentum analysis, correlation-aware sizing)
**Relevance**: Very High (advanced AI strategy, production patterns to adopt)
