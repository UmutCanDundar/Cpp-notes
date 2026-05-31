# HFT — Mathematics & Quantitative Questions

---

## Probability & Statistics

**Q: What is the expected value of a fair die roll?**
E[X] = (1+2+3+4+5+6)/6 = 3.5

**Q: You flip a fair coin until you get heads. What is the expected number of flips?**
Geometric distribution with p=0.5. E[X] = 1/p = 2.

**Q: What is the expected number of flips to get two heads in a row?**
Let E = expected flips from start, A = state after one head.
- E = 1 + 0.5·A + 0.5·E → E = 2 + A
- A = 1 + 0.5·0 + 0.5·E → A = 1 + 0.5E
- Solving: E = 6

**Q: You have a biased coin with P(H) = p. What is the expected number of flips to get HH?**
E = (1 + p) / p²

**Q: What is variance and standard deviation? Why does it matter in trading?**
Variance = E[(X - μ)²]. Std dev = √Variance. In trading, std dev of returns = volatility. Higher volatility = wider bid-ask spread needed to be profitable as a market maker.

**Q: What is covariance and correlation?**
Cov(X,Y) = E[(X-μx)(Y-μy)]. Correlation ρ = Cov(X,Y) / (σx · σy) ∈ [-1, 1].
In pairs trading, you look for high positive correlation between two assets.

**Q: What is the law of large numbers?**
As n→∞, the sample mean converges to the true mean. This is why a strategy needs enough trades to have statistical significance — short-term P&L can look great by luck.

**Q: What is the central limit theorem?**
The sum of n independent random variables approaches a normal distribution as n→∞, regardless of the original distribution. Used to justify using Gaussian models for returns over short intervals.

**Q: What is a Poisson process? Where is it used in HFT?**
Events (e.g. order arrivals) occur at a constant average rate λ. P(k events in time t) = e^(-λt)(λt)^k / k!. Used to model order flow arrival rates, tick frequency, trade arrival.

**Q: What is the Kelly Criterion?**
Optimal fraction of capital to bet: f* = (bp - q) / b, where b = net odds, p = P(win), q = P(loss).
In trading: f* = (edge) / (variance). Maximizes long-run geometric growth. Over-betting Kelly leads to ruin.

**Q: A stock is at $100. It goes up 10% or down 10% with equal probability each day. After 2 days, what is the expected value?**
E = 0.25·121 + 0.5·99 + 0.25·81 = 100. Expected value is $100. But geometric mean = √(1.1·0.9) = √0.99 < 1. The stock drifts down in log space — this is volatility drag.

---

## Market Microstructure

**Q: What is the bid-ask spread and why does it exist?**
Bid = highest price a buyer will pay. Ask = lowest price a seller will accept. Spread compensates market makers for: (1) inventory risk, (2) adverse selection risk (trading against informed traders), (3) order processing costs.

**Q: What is adverse selection in market making?**
When a market maker quotes, informed traders (who know the true value) trade against them. The market maker buys when the price is about to fall, sells when it's about to rise. Adverse selection is the biggest risk in market making.

**Q: What is the mid-price?**
(Bid + Ask) / 2. The theoretical fair value. Market makers try to stay delta-neutral around the mid.

**Q: What is queue priority and why does it matter in HFT?**
Orders at the same price are filled in time priority (FIFO). Being first in the queue at a price level means you get filled before others. HFT systems compete on co-location and low-latency to get queue priority.

**Q: What is co-location?**
Placing your servers physically inside the exchange's data center. Reduces network round-trip time from ~1ms (internet) to ~1-10µs (cross-connect cable). Every microsecond of latency advantage translates to better queue position.

**Q: What is latency arbitrage?**
Exploiting the fact that market data from multiple venues arrives at different times. A fast trader sees a price update on venue A before venue B processes it, and trades on venue B before the stale quote is updated.

**Q: What is market impact?**
When you send a large order, you move the price against yourself. Buying pushes the ask price up. Market impact = expected price worsening as a function of order size. Almgren-Chriss model quantifies this.

**Q: What is the VWAP (Volume Weighted Average Price)?**
VWAP = Σ(price_i × volume_i) / Σ(volume_i). A common execution benchmark. Beating VWAP = executed at a better average price than the day's volume-weighted average.

**Q: What is TWAP?**
Time Weighted Average Price. Slice a large order into equal pieces executed at equal time intervals. Minimizes market impact but doesn't adapt to volume patterns.

**Q: What is the order book?**
A list of all outstanding limit orders, organized by price. The top of book (best bid, best ask) is level 1. Levels 2, 3, ... are deeper. HFT systems process the full order book in real-time to model supply/demand.

---

## Statistics for Trading

**Q: What is a p-value and how do you use it in backtesting?**
P-value = probability of observing results at least as extreme as seen, given the null hypothesis (no edge). p < 0.05 = "statistically significant at 5%". In trading, with many strategies tested, multiple testing inflates false positives (use Bonferroni correction or Sharpe ratio thresholds).

**Q: What is the Sharpe ratio?**
Sharpe = (E[R] - R_f) / σ(R). Risk-adjusted return. In HFT, annualized Sharpe > 2 is good, > 3 is excellent. Sharpe does not distinguish between skewness — a strategy can have high Sharpe but blow up on tail events.

**Q: What is the Sortino ratio?**
Like Sharpe but only penalizes downside volatility. Sharpe penalizes both up and down moves equally. Sortino = (E[R] - R_f) / σ_downside.

**Q: What is maximum drawdown?**
The largest peak-to-trough decline in cumulative P&L. A critical risk metric. HFT strategies typically have very low drawdown relative to annual return.

**Q: What is autocorrelation in returns?**
Correlation of a return series with its own lagged values. If autocorrelation > 0, trend-following may work. If < 0 (mean reversion), mean reversion strategies may work. Most liquid markets have near-zero short-lag autocorrelation — hard to exploit.

**Q: What is a z-score and how is it used in pairs trading?**
z = (spread - mean) / std_dev. In pairs trading, when z > 2, the spread is wide → short the expensive, buy the cheap. When z < -2, reverse. Exit when z approaches 0.

---

## Linear Algebra & Numerical Methods

**Q: What is matrix multiplication complexity?**
Naïve: O(n³). Strassen: O(n^2.807). Practically, BLAS libraries use blocked algorithms optimized for cache. Relevant for covariance matrix computation in portfolio optimization.

**Q: What is Cholesky decomposition and when is it used?**
Decomposes a positive-definite matrix A = LL^T. Used to: (1) solve linear systems efficiently, (2) sample correlated random variables (Monte Carlo). In risk, Cholesky of the covariance matrix generates correlated P&L scenarios.

**Q: What is PCA (Principal Component Analysis)?**
Finds the directions of maximum variance in a dataset. In fixed income, the first 3 PCs of the yield curve explain ~99% of variance (level, slope, curvature). Used for dimensionality reduction in factor models.

**Q: What is the difference between float and double precision?**
- float: 32-bit, ~7 decimal digits of precision
- double: 64-bit, ~15 decimal digits of precision
For financial calculations (prices, P&L), always use double or integer arithmetic (price in ticks). Never compare floats with ==.

**Q: What is integer overflow risk in financial systems?**
If price is stored as int32 and multiplied by quantity, overflow can silently corrupt data. Use int64 for any intermediate multiplication. In tick-based systems, price × quantity can easily exceed 2^31.
