# Polymarket Trading Bot v3.2 - Automated Trading, Execution & Backtesting

**Polymarket trading bot for automated execution, arbitrage, dip trading, smart-money copy trading, risk management, monitoring, and backtesting.**

Built for traders and developers who want to automate Polymarket trading while keeping **execution safety, position sizing, exposure limits, loss protection, and operational controls** close to the trading engine.

Created by [@casatrick](https://t.me/casatrick)

[![English](https://img.shields.io/badge/Language-English-blue)](README.md) [![Arabic](https://img.shields.io/badge/Language-Arabic-green)](README_AR.md)

---

## What is this Polymarket trading bot?

This project is an open-source **automated Polymarket trading bot** designed to execute trading strategies while applying configurable risk and execution controls.

The bot includes:

* Automated Polymarket trading strategies
* Order execution with live price protection
* Fee- and gas-aware profit checks
* Arbitrage and dip-arbitrage execution
* Smart-money / trader copy filtering
* Dynamic position sizing
* Multi-layer risk management
* Exposure limits
* Loss-streak protection
* Wallet circuit breakers
* Real-time trading dashboard
* Emergency stop controls
* Polygon gas monitoring
* JSONL order-book backtesting

The goal is not only to generate trading signals, but to make the complete trading process more controlled:

```text
Market Data
     ↓
Strategy Signal
     ↓
Risk Checks
     ↓
Position Sizing
     ↓
Protected Order
     ↓
Execution
     ↓
Position / Exposure Tracking
     ↓
Monitoring & Recovery
```

---

## Why execution and risk management matter

A trading strategy can produce a valid signal and still fail during execution.

Real automated trading systems have to deal with:

* changing order-book prices
* transaction costs
* partial fills
* failed execution
* stale wallet activity
* excessive exposure
* repeated losses
* insufficient gas
* reconnects and operational failures

v3.2 adds multiple controls around these failure modes so the bot can make trading decisions with execution and account state in mind.

---

# What's New in v3.2

## Execution Safety

v3.2 resolves 13 known execution and safety issues.

### Fee-aware profit calculations

Arbitrage and DipArb profit calculations account for:

* taker fees
* gas costs
* minimum net-profit thresholds

This prevents signals from being triggered solely by gross price differences.

### Price-protected orders

Market orders use worst-price caps or floors derived from the live order book.

This adds an execution constraint instead of allowing an order to trade without a defined price boundary.

### Sequential arbitrage execution

YES and NO legs are executed sequentially rather than racing both orders simultaneously.

If a partial fill occurs, the system can unwind and reconcile the remaining exposure.

### Improved DipArb hedging

DipArb includes:

* shorter second-leg timeout
* stop-loss protection
* 1:1 hedge requirements based on actual fills

### Fresh copy-trading quotes

The bot skips stale whale trades and re-quotes entries using the current market book.

Entries are additionally checked against spread, premium, and liquidity conditions.

### Wallet filtering

Custom wallets go through the same filtering process as leaderboard candidates, including:

* win rate
* PnL
* trade history
* profit factor
* consistency
* whale-trade checks

### Exposure controls

The bot tracks total and per-market exposure and blocks new trades when exposure limits are exceeded.

### Configurable Polygon RPC

Set `POLYGON_RPC_URL` to use a dedicated RPC endpoint instead of relying on a public endpoint.

### Wallet circuit breaker

Copy trading can disable a wallet after three consecutive failures, followed by a cooldown period.

### Order-book backtesting

v3.2 includes a JSONL order-book replay harness with fee and gas modeling:

```bash
npm run backtest
```

### Polygon gas monitoring

The bot checks the configured Polygon gas balance periodically and can pause trading when the balance falls below the configured minimum.

### Position sizing and loss-streak protection

The bot skips orders below the configured USD minimum and pauses trading after six consecutive losses.

---

# Risk Management

The bot uses a six-layer protection system.

| Layer | Control                |  Default |
| ----- | ---------------------- | -------: |
| 1     | Daily loss limit       |       5% |
| 2     | Monthly loss limit     |      15% |
| 3     | Maximum drawdown       |      25% |
| 4     | Total-loss halt        |      40% |
| 5     | Consecutive-loss pause | 6 losses |
| 6     | Total exposure cap     |      30% |

### Daily loss limit

Trading pauses after the configured daily loss threshold is reached.

### Monthly loss limit

Trading is paused when the configured monthly loss threshold is exceeded.

### Drawdown limit

The bot monitors drawdown from peak capital and pauses trading when the configured threshold is breached.

### Total-loss halt

Trading stops when the configured total-loss threshold is reached and requires manual restart.

### Loss-streak pause

After six consecutive losing trades, the bot pauses trading for a cooldown period.

### Exposure cap

New positions are blocked when total open exposure exceeds the configured capital percentage.

Per-market exposure is also limited.

---

# Dynamic Position Sizing

Position size changes according to recent trading performance.

Default behavior:

* Base position size: **2% of capital**
* Consecutive losses reduce position size
* Consecutive wins increase position size
* Position growth is capped
* Orders below the configured minimum size are skipped

Example with `$250` configured capital:

```text
Base position:
$250 × 2% = $5

After consecutive losses:
position size decreases

After consecutive wins:
position size increases, subject to the configured cap
```

Position sizing is intended to work together with the bot's loss limits and exposure controls rather than operating independently.

---

# Trading Strategies

The bot currently supports four trading modes.

## 1. Arbitrage

The strategy searches for situations where the combined YES and NO prices satisfy an arbitrage condition.

Conceptually:

```text
YES + NO < $1.00
```

The system then attempts to execute both sides while considering:

* execution price
* fees
* gas
* minimum net profit
* available liquidity

The theoretical price relationship does not guarantee realized profit because execution costs, liquidity, timing, and market conditions affect the result.

---

## 2. DipArb

DipArb monitors short-duration crypto markets for large price movements.

Current configuration includes:

* BTC and ETH 15-minute markets
* rapid price-move detection
* first-leg entry
* opposite-side hedge
* stop-loss protection
* minimum trade size
* hedge validation using actual fills

The strategy is designed around fast market movement and therefore remains sensitive to execution conditions and liquidity.

---

## 3. Smart Money

The Smart Money strategy tracks selected Polymarket traders and applies filters before copying trades.

Current filters include:

* minimum 60% win rate
* minimum total PnL
* profit factor of at least 1.5x
* consistency score requirement
* protection against single-trade / whale-driven results
* validation for custom wallets

The strategy automatically copies qualifying trades subject to the bot's risk controls.

---

## 4. Direct Trading

Direct Trading provides manual execution tools through the dashboard.

Features include:

* FOK orders
* quick buy controls
* execution-oriented order controls
* stop-loss configuration
* take-profit configuration
* maximum holding period

---

# Backtesting

The repository includes an order-book replay backtesting harness.

Run:

```bash
npm run backtest
```

The backtesting environment can replay JSONL order-book data while modeling:

* order execution
* fees
* gas costs
* strategy behavior

This makes it possible to evaluate execution logic against recorded market conditions before using live capital.

---

# Dashboard

The web dashboard provides a central view of the trading system.

## Monitoring

The dashboard displays:

* live / dry-run mode
* USDC balance
* Polygon gas balance
* session PnL
* strategy status
* risk limits
* drawdown
* exposure
* win/loss streaks
* trading halt / pause state

## Controls

The dashboard provides:

* strategy toggles
* emergency stop
* panic sell
* live / dry-run mode controls

The emergency stop blocks trading until the bot is restarted.

---

# Installation

## Requirements

* Windows, macOS, or Linux
* Node.js 18+
* Git
* Polymarket account
* Trading funds on Polygon
* Polygon gas balance for transactions

Download:

* [Node.js](https://nodejs.org/)
* [Git](https://git-scm.com/)

---

## Clone the repository

```bash
git clone https://github.com/casatrickdev/polymarket-trading-bot
cd Polymarket-trading-bot
```

---

## Install dependencies

```bash
npm install
```

Build the dashboard:

```bash
cd dashboard
npm install
npm run build
cd ..
```

---

# Configuration

Create `.env` from `.env.example`:

```bash
cp .env.example .env
```

Then configure the required environment variables.

Example:

```env
# ==============================================
# WALLET CONFIGURATION
# ==============================================

POLYMARKET_PRIVATE_KEY=0xYourPrivateKeyHere

# ==============================================
# BOT SETTINGS
# ==============================================

CAPITAL_USD=250

# true = simulation mode
# false = live trading
DRY_RUN=true

# ==============================================
# RISK MANAGEMENT
# ==============================================

DAILY_MAX_LOSS_PCT=0.05
MONTHLY_MAX_LOSS_PCT=0.15
MAX_DRAWDOWN_PCT=0.25
TOTAL_MAX_LOSS_PCT=0.40

# ==============================================
# OPTIONAL RPC
# ==============================================

# POLYGON_RPC_URL=https://your-rpc-provider.example/...

# ==============================================
# OPTIONAL DASHBOARD SECURITY
# ==============================================

# DASHBOARD_HOST=127.0.0.1
# DASHBOARD_TOKEN=generate-a-long-random-string
```

### Private-key security

Never commit your private key to GitHub.

Do not place secrets directly into source files.

Use environment variables or an appropriate secrets-management solution for production deployments.

---

# Running the Bot

Start the trading bot with its dashboard:

```bash
npx tsx bot-with-dashboard.ts
```

The terminal will display:

1. Startup logs
2. Wallet connection status
3. Dashboard address
4. Strategy status
5. Risk status

The dashboard runs locally by default.

Typical URL:

```text
http://localhost:3001
```

When `DASHBOARD_TOKEN` is configured, use the authenticated URL printed by the application.

---

# Dry Run Mode

For initial testing:

```env
DRY_RUN=true
```

Dry-run mode allows you to inspect system behavior without intentionally placing live trades.

Do not enable live trading until you have verified:

* wallet configuration
* strategy configuration
* risk limits
* dashboard controls
* RPC connectivity
* execution behavior
* emergency-stop behavior

---

# Troubleshooting

## Command not found

Verify that Node.js is installed:

```bash
node --version
```

Node.js 18 or newer is required.

---

## Connection failed

Check:

* internet connectivity
* Polymarket configuration
* private-key configuration
* Polygon RPC configuration
* wallet access

---

## Insufficient funds

Verify that the wallet has the required trading balance and sufficient Polygon gas balance.

---

## Trade below minimum

The bot can reject positions below the configured minimum trade size.

Increase the configured capital or wait for a trade that satisfies the minimum-size conditions.

---

## Bot paused

Open the dashboard and inspect the Risk Status panel.

The bot can pause trading after:

* daily loss threshold
* monthly loss threshold
* drawdown threshold
* loss streak
* exposure limit
* insufficient gas
* other configured safety conditions

---

# Safety & Risk Disclosure

Automated trading involves financial risk.

This project does **not** guarantee profits.

Market conditions, liquidity, fees, gas costs, execution timing, technical failures, and strategy behavior can all affect results.

Before using live capital:

1. Test in dry-run mode.
2. Verify all risk limits.
3. Confirm emergency controls work.
4. Start with capital you can afford to lose.
5. Monitor the system regularly.
6. Keep private keys secure.

Never share your private key.

---

# Technical Focus

This repository is also intended as an engineering reference for building automated Polymarket trading systems.

Core engineering areas include:

```text
Polymarket Trading
├── Strategy execution
├── Order management
├── Order-book based pricing
├── Position sizing
├── Risk management
├── Exposure tracking
├── Execution safety
├── Backtesting
├── Monitoring
├── Recovery controls
└── Operational tooling
```

The project focuses on the part of automated trading that happens between a strategy signal and the final account state.

---

# Documentation

Additional documentation:

* [SDK Documentation](SDK_DOCUMENTATION.md)
* [Beginner Guide](BEGINNER_GUIDE.md)
* [Quick Start](QUICKSTART.md)
* [Arabic README](README_AR.md)

---

# Version History

### v3.2 - September 2026

* Fee-aware execution
* Price-protected orders
* Sequential arbitrage execution
* Improved DipArb hedging
* Fresh copy-trading quotes
* Wallet filtering
* Exposure caps
* Configurable Polygon RPC
* Wallet circuit breaker
* JSONL order-book backtesting
* Polygon gas monitoring
* Minimum-size protection
* Loss-streak protection

### v3.1 - January 2026

* Four-layer risk management
* Smart-money filtering
* Dynamic position sizing
* Enhanced monitoring

### v3.0 - December 2025

* Dashboard
* Multi-strategy support
* Auto-rotation

### v2.0 - November 2025

* Smart Money
* Arbitrage
* DipArb strategies

### v1.0 - October 2025

* Initial release

---

# About Casatrick

Built by [@casatrick](https://x.com/casatrick).

Casatrick focuses on **Polymarket trading systems, automated execution, trading infrastructure, and production-oriented tooling**.

For questions, open a GitHub issue or contact [@casatrick](https://x.com/casatrick).

---

## Keywords

Polymarket trading bot · Polymarket bot · automated Polymarket trading · Polymarket trading automation · Polymarket arbitrage bot · Polymarket trading system · Polymarket execution · Polymarket risk management · Polymarket backtesting · prediction market trading bot · automated trading bot · crypto trading bot

---

⚠️ **Disclaimer:** Trading involves risk. This software does not guarantee profits. Use it at your own risk and never trade more than you can afford to lose.
