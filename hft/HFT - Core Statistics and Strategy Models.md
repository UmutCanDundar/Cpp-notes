# HFT: Core Statistics and Strategy Models

---

## Table of Contents

1. [Strategy Types](#1-strategy-types)
2. [Market Making in Detail](#2-market-making-in-detail)
3. [Backtesting and Overfitting](#3-backtesting-and-overfitting)
4. [Common Interview Questions](#4-common-interview-questions)

---

## 1. Strategy Types

### 1.1 Momentum

Assumes a price moving in one direction will keep moving that way.

```
signal positive (price rising)  → go long
signal negative (price falling) → go short
```

**Order Flow Imbalance (OFI)** — one of the most important momentum signals:

```
OFI = bid_volume_delta - ask_volume_delta
```

If volume is building up on the bid side and draining on the ask side, the price tends to move up.

**When it breaks down:**

| Condition | Why it hurts |
|---|---|
| Mean-reverting market | If the market trades in a range, "price is rising" can actually mark the local top — the trend doesn't continue, price snaps back |
| High slippage | If the gap between the intended price and the executed price exceeds the expected profit, every trade loses money |
| Heavy noise | Random price jitter with no real trend causes the model to keep firing false-positive signals |

---

### 1.2 Mean Reversion

Assumes price reverts to its long-run average (the opposite assumption of momentum).

```
z-score = (price - rolling_mean) / rolling_std

z > 2   → sell (price looks expensive, expect it to fall back)
z < -2  → buy  (price looks cheap, expect it to rise back)
```

**Where it shows up in HFT:**
- Price movement within the bid-ask spread
- Correlated instruments (e.g. ES futures vs SPY ETF)
- The theoretical foundation of market making

**Pairs trading:** Trade the spread between two cointegrated instruments. Open a position when the spread widens abnormally, close it when it reverts to the mean.

---

### 1.3 Arbitrage

Exploiting the same (or economically equivalent) asset being priced differently in different places.

| Type | Description |
|---|---|
| Latency arbitrage | Trading on one exchange the instant a price move occurs, before a slower venue has caught up |
| Statistical arbitrage | Trading temporary divergences between highly correlated instruments |
| ETF arbitrage | The gap between an ETF's price and the value of its underlying basket |
| Cross-venue arbitrage | Price differences for the same security across exchanges (e.g. NYSE vs NASDAQ) |

**Key point:** Arbitrage opportunities are not permanent — competition (other firms spotting and trading the same opportunity) closes the gap quickly.

---

## 2. Market Making in Detail

Quoting on both sides (bid and ask) to earn the spread.

```
Bid: 100.00   (offer to buy)
Ask: 100.02   (offer to sell)
Spread = 0.02 (target profit)
```

Ideally, buy and sell flow arrives in balance, inventory (the position held) stays close to zero, and only the spread is captured as profit.

### 2.1 Inventory Risk

If the market moves in one direction (e.g. everyone wants to sell), the market maker keeps receiving orders on the same side and **inventory** (the position held) builds up:

```
+500 lots → +1000 lots → +2000 lots ...
```

At that point the market maker is no longer just earning the spread — they're carrying a directional position and exposed to adverse price moves. This is an unwanted risk.

### 2.2 Adverse Selection

The risk that the counterparty knows more than you do. Someone may send what looks like a "normal" order, but they actually have better information (or a better prediction) about where the price is heading than you do. Without realizing it, you've been "adversely selected."

### 2.3 Asymmetric Quoting

As inventory builds up, a market maker shifts their quotes **away from being symmetric around the mid price**:

```
Inventory = 0      →  Bid: 99.99   Ask: 100.01   (symmetric)
Inventory = +1000  →  Bid: 99.95   Ask: 99.98    (both shifted down)
                       (bought too much, wants to sell)
```

- Lowering the ask makes selling more attractive → encourages trades that reduce inventory
- Lowering the bid too → discourages buying more, since the position is already too large

### 2.4 Avellaneda-Stoikov Model

A classic market making model that prices inventory risk mathematically.

```
reservation_price = mid - q * γ * σ² * T

spread = γ * σ² * T + (2/γ) * ln(1 + γ/k)
```

| Symbol | Meaning |
|---|---|
| `q` | current inventory |
| `γ` | risk aversion coefficient |
| `σ` | volatility |
| `T` | time remaining (e.g. until end of day) |
| `k` | order book liquidity parameter |

As `q` (inventory) grows, `reservation_price` moves away from the mid price — this is the mathematical source of asymmetric quoting.

---

## 3. Backtesting and Overfitting

### 3.1 What Overfitting Is

The model memorizes historical data instead of learning a real pattern, so it fails on new/live data. Analogy: a student who memorizes exam answers instead of understanding the material fails as soon as the questions change.

**Example:**

```
Strategy: RSI < 27.3 → buy, RSI > 71.8 → sell, lookback = 14, stop-loss = 2.7%
```

If these specific numbers were chosen by testing dozens of combinations and picking whichever gave the highest profit on 2020–2023 data, the numbers aren't grounded in market logic — they just happen to fit that specific dataset. Tested on 2024 data, the strategy usually collapses.

**Why it happens — the statistical intuition:**

Test enough parameter combinations and you can find one that looks great purely by chance, even on data with no real edge at all.

```
100 different parameter combinations tested
Each one has roughly a 50% chance of looking "profitable" on random data
Out of 100 trials, at least one will look 95%+ profitable purely by luck
→ If that one gets picked and called "the strategy," there is no real edge
```

### 3.2 Deflated Sharpe Ratio (DSR)

A metric proposed by Bailey & López de Prado that "deflates" the Sharpe ratio based on how many strategies/parameter sets were tried. The more trials, the more skeptical you should be of the best-looking result.

### 3.3 Walk-Forward Validation

The standard method for detecting/preventing overfitting:

```
[train: 2020] → [test: 2021]
        [train: 2021] → [test: 2022]
                [train: 2022] → [test: 2023]
```

At each stage, the model is tested on data it has never seen before. Consistent performance across periods suggests a real edge. Strong in one period but weak in others is a red flag for overfitting.

### 3.4 Lookahead Bias

The error of using information at decision time that wasn't actually known yet.

**Example 1:** A backtest decides "buy/sell at 9am" based on that same day's closing price — but the closing price isn't known at 9am. Future information has leaked into a past decision.

**Example 2:** A company's quarterly earnings are released on April 15th, but the backtest data marks this information as available starting April 1st (a mislabeled timestamp).

**Result:** The backtest shows outstanding performance because the strategy can effectively "see the future" — an advantage that's impossible in live trading.

### 3.5 Survivorship Bias

The error of testing only on assets/companies that **still exist today** (the survivors).

**Example:** A "S&P 500 stocks 2000–2024" dataset built from today's S&P 500 membership list won't include companies like Lehman Brothers, which went bankrupt in 2008 and is no longer in the index.

**Result:** A strategy like "buy falling stocks, they recover" looks great in backtests because the dataset only contains companies that **were able to recover** (the survivors). In reality, some companies that fell kept falling into bankruptcy — but since they're excluded from the data, results look artificially good.

### 3.6 Other Common Backtesting Mistakes

| Mistake | Description |
|---|---|
| Ignoring transaction costs | Skipping commissions, spread, and slippage makes profitability look higher than it really is |
| Lookahead bias | See 3.4 |
| Survivorship bias | See 3.5 |
| Overfitting | See 3.1 |
| Insufficient sample / single-regime testing | Testing only in one market regime (e.g. only a bull market) |

---

## 4. Common Interview Questions

- **When does a momentum strategy stop working?**
  It breaks down in mean-reverting markets, under high slippage, and during periods of heavy noise.

- **Why do arbitrage opportunities close?**
  Other firms spot and trade the same opportunity, and competition pushes the gap to zero. Arbitrage is never permanent.

- **What are the most common backtesting mistakes?**
  Lookahead bias, survivorship bias, ignoring transaction costs, overfitting.

- **What is lookahead bias?**
  Using information at decision time that wasn't actually known yet (e.g. using a day's closing price as if it were known at the start of that day).

- **What is survivorship bias?**
  Including only assets that still exist today in a dataset, while excluding the ones that failed or were delisted.

- **Why does a market maker avoid inventory risk?**
  Because the goal is to earn the spread, not to take directional bets. A growing inventory exposes the firm to price risk and adverse selection.

- **Why does the Deflated Sharpe Ratio matter?**
  When many strategies/parameter sets are tested, it helps distinguish a genuine edge from a result that only looks good by chance.

---

