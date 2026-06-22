# Trading & Quantitative Finance Glossary

A comprehensive reference for trading, algorithmic trading, HFT, statistics, and strategy terminology.

---

## 1. Market Fundamentals

**Asset**
Any tradable instrument — stock, bond, currency, commodity, derivative, cryptocurrency.

**Instrument**
A specific tradable security. "GARAN.IS is an instrument on BIST."

**Ticker / Symbol**
Short code identifying an instrument. `GARAN.IS` = Garanti Bankasi on Istanbul Stock Exchange. `.IS` suffix = BIST.

**Exchange**
The marketplace where instruments are traded. BIST (Borsa Istanbul), NASDAQ, NYSE, LSE.

**Venue**
A trading location — can be an exchange, dark pool, or ECN. Multiple venues can trade the same instrument.

**Market Hours / Trading Session**
The window during which a market accepts orders. Outside hours: pre-market, after-hours (limited liquidity).

**Open / Close**
Opening price = first traded price of the session. Closing price = last traded price. Used as reference points for daily returns and signals.

**OHLCV**
Open, High, Low, Close, Volume — the five standard data fields for a daily (or intraday) price bar.

**Tick**
The smallest unit of price movement an instrument can make. Also used to mean a single trade event.

**Tick Size**
The minimum price increment. BIST rules vary by price range (e.g. 0.01 TL for some ranges). Orders with invalid tick sizes are rejected by the exchange.

**Lot Size**
Minimum tradable quantity. On BIST, 1 lot = 1 share for most equities.

**Liquidity**
How easily you can buy or sell without moving the price. High liquidity = many orders in the book, tight spread. Low liquidity = few orders, wide spread, large orders move the market.

**Illiquidity**
The opposite of liquidity. Small orders can move prices significantly. Common in small-cap stocks and exotic instruments.

**Volume**
Number of shares (or contracts) traded in a given period. High volume often accompanies significant price moves.

**Market Cap**
Total market value of a company: `price × shares_outstanding`. Used to classify stocks (large-cap, mid-cap, small-cap).

**Index**
A basket of stocks representing a market or sector. BIST 100 = top 100 Turkish equities. S&P 500 = top 500 US equities.

---

## 2. Order Types

**Market Order**
"Buy/sell at the best available price right now." Guaranteed fill, no price guarantee. Causes slippage in illiquid markets.

**Limit Order**
"Buy/sell only at this price or better." Price guaranteed, fill not guaranteed. Adds to the order book if not immediately matchable.

**Stop Order**
Becomes a market order when price reaches a trigger level. Used for loss-limiting ("stop-loss") or trend-following entry.

**Stop-Limit Order**
Becomes a limit order (not market) when the stop triggers. Avoids slippage but risks not filling if price moves through quickly.

**IOC (Immediate or Cancel)**
Fill what you can right now, cancel the rest. No resting in the book. Taker behavior.

**FOK (Fill or Kill)**
Fill the entire order immediately or cancel it entirely. No partial fills, no resting.

**MOO (Market on Open)**
Executes at the opening auction price. Used by institutions for index rebalancing.

**MOC (Market on Close)**
Executes at the closing auction price. Benchmark for VWAP strategies.

**Iceberg Order**
Large order split into visible and hidden portions. Only the "tip" shows in the order book; rest is hidden to reduce market impact.

**Peg Order**
Automatically adjusts to track best bid or ask. Common in market-making to maintain queue position.

---

## 3. Order Book & Market Structure

**Order Book (Level 2)**
The full list of resting buy (bid) and sell (ask) orders at every price level, ranked by price then time.

**Best Bid**
The highest price any buyer is currently willing to pay. Sellers receive this price if they send a market order.

**Best Ask (Best Offer)**
The lowest price any seller is currently willing to accept. Buyers pay this price if they send a market order.

**Mid Price**
`(best_bid + best_ask) / 2`. Reference price used for mark-to-market and signal generation.

**Spread (Bid-Ask Spread)**
`best_ask - best_bid`. The market maker's gross profit per round-trip. Tight spread = liquid market. Wide spread = illiquid or uncertain market.

**Depth**
How many orders sit at each price level. "Deep book" means many orders at many levels — large trades have less price impact.

**NBBO (National Best Bid and Offer)**
In the US: the best bid and ask consolidated across all venues. Regulation requires brokers to route orders to the NBBO venue.

**Price Impact**
How much a trade moves the market price. Large orders consume multiple price levels in the book, pushing price against you.

**Market Impact**
Broader than price impact — includes the effect of your trading on other market participants who observe your order flow.

**Queue Position**
Your position in the FIFO queue at a given price level. Earlier orders fill first. Being first in queue is critical for market makers.

**FIFO (First In First Out)**
Standard order matching rule: at the same price, oldest order fills first.

**Pro-Rata Matching**
Alternative to FIFO: at the same price, fills are allocated proportionally to order size. Common in some futures markets.

**Dark Pool**
A private trading venue where orders are not visible to the public order book. Used by institutions to execute large orders without market impact.

**ECN (Electronic Communication Network)**
An automated system that matches buy and sell orders electronically, often providing direct market access.

---

## 4. Trade Execution

**Fill**
A completed trade. "My order got filled at 100.05."

**Partial Fill**
Only part of the order was executed. The rest remains open or is cancelled (depending on order type).

**Execution Report**
The message from the exchange confirming fill details: price, quantity, timestamp, order ID.

**Slippage**
The difference between the expected price and the actual fill price. Caused by market movement between order submission and execution, or by consuming multiple price levels.

**Market Impact Cost**
The price movement caused by your own order. A large buy order pushes prices up, making subsequent fills more expensive.

**Latency**
Time delay between an event (e.g. market data arrival) and your response (e.g. order sent). In HFT, measured in microseconds or nanoseconds.

**Round-Trip Latency**
Time from sending an order to receiving the execution report. The key performance metric for order execution systems.

**Co-location (Colo)**
Placing your trading servers physically inside or adjacent to the exchange's data center. Minimizes round-trip latency due to cable length.

**Cross-Connect**
A direct cable between your server and the exchange matching engine within the data center. Faster than going through the internet.

**Microwave / Wireless Links**
Trading firms use point-to-point microwave towers to transmit data faster than fiber optic cable (electromagnetic waves travel at ~99% speed of light in air vs ~70% in fiber).

**Fill Rate**
Percentage of orders that result in fills. `fills / orders_sent`.

**Rejection**
An order refused by the exchange or broker. Causes: invalid price, exceeded limits, bad parameters.

**Adverse Fill**
A fill received at an unfavorable time — for example, a market maker fills a large order just before the price moves sharply against them.

---

## 5. Positions & PnL

**Long**
You own the asset. You profit if price rises. Maximum loss = what you paid (if price goes to zero).

**Short**
You borrowed and sold an asset you don't own. You profit if price falls. You must buy it back later ("covering the short"). Theoretically unlimited loss if price keeps rising.

**Flat**
No position. Neither long nor short.

**Position Size**
How many units you hold. Also called "notional exposure" when expressed in currency terms.

**Mark-to-Market (MtM)**
Valuing your position at current market price. "My position is marked at 105 TL."

**Unrealized PnL**
Profit or loss on an open position that has not yet been closed. "Paper profit."

**Realized PnL**
Profit or loss locked in when a position is closed. Actual cash change.

**PnL (Profit and Loss)**
Total performance: `realized_PnL + unrealized_PnL`. The bottom line of any trading strategy.

**Inventory**
The net position a market maker holds as a result of providing liquidity. Market makers try to keep inventory near zero to limit directional risk.

**Inventory Risk**
The risk a market maker takes by holding non-zero inventory. If you are long 10,000 shares and price falls, you lose.

**Mark-to-Market Loss**
Loss on open positions due to price moving against you, not yet realized.

**Drawdown**
Decline from a peak equity value to a subsequent trough. `(peak - trough) / peak`. Measures how much you lost before recovering.

**Max Drawdown**
The largest drawdown over a historical period. Key risk metric — tells you the worst case loss you would have experienced.

---

## 6. Risk Management

**Risk Management**
The system of rules and limits that prevents a single bad day, strategy failure, or bug from causing catastrophic loss.

**Position Limit**
Maximum allowed exposure in a single instrument. Pre-trade check: if the new order would breach the limit, reject it.

**Loss Limit (Daily Loss Limit)**
Maximum allowable loss per day. Once hit, trading is automatically halted.

**Kill Switch**
An emergency mechanism to immediately cancel all open orders and halt all trading. Required by regulators in most jurisdictions.

**Pre-Trade Risk Check**
Risk validation performed before an order is sent to the exchange. Checks: position limits, loss limits, price sanity, order size limits.

**Post-Trade Risk Update**
Updating internal position and PnL state after receiving a fill confirmation.

**VaR (Value at Risk)**
The maximum expected loss over a time period at a given confidence level. "95% VaR of 100K TL over 1 day" means: on 95 out of 100 days, loss will not exceed 100K TL.

**CVaR (Conditional VaR / Expected Shortfall)**
The expected loss given that you are already beyond the VaR threshold. Captures tail risk that VaR misses.

**Tail Risk**
The risk of rare but extreme events — the "fat tail" of the return distribution. Normal distribution underestimates this.

**Fat Tails (Leptokurtosis)**
Financial return distributions have heavier tails than a normal distribution predicts. Extreme moves happen more often than the model says.

**Leverage**
Using borrowed capital to amplify returns (and losses). 2x leverage means a 1% move causes a 2% PnL change.

**Margin**
Collateral required to hold a leveraged position. If losses erode margin below a threshold, a margin call or forced liquidation occurs.

**Margin Call**
A demand to deposit more collateral or reduce position size because losses have eroded margin below the required level.

**Liquidation**
Forced closing of a position, usually by the broker, because margin requirements are no longer met. Can cascade (one liquidation drops prices, triggering more liquidations).

**Short Squeeze**
When a heavily shorted stock rises sharply, forcing short sellers to buy back (cover) at higher prices, driving the price even higher. A positive feedback loop.

**Circuit Breaker**
A market-wide or strategy-level halt triggered when price moves or loss thresholds are exceeded. Prevents runaway losses.

**Greeks (Option Risk Sensitivities)**
Measures of how an option's price changes with respect to various inputs.

**Delta (Δ)**
Change in option price per 1 unit change in the underlying price. Call delta: 0 to +1. Put delta: -1 to 0.

**Gamma (Γ)**
Rate of change of delta. High gamma = delta changes rapidly as price moves = frequent rehedging required.

**Vega (ν)**
Change in option price per 1% change in implied volatility. Always positive — higher vol = more valuable option.

**Theta (Θ)**
Daily time decay of an option's value (assuming all else equal). Negative for option buyers, positive for option sellers.

**Delta Hedging**
Neutralizing directional (delta) exposure by taking an offsetting position in the underlying asset. Must be rebalanced as delta changes (due to gamma).

**Hedge**
Any position taken to reduce or offset the risk of another position. Imposes a cost (reduced upside) in exchange for reduced downside.

---

## 7. Market Participants

**Market Maker**
A participant that continuously quotes both bid and ask prices, providing liquidity. Profits from the spread. Bears inventory risk.

**Market Taker**
A participant that sends market orders or aggressive limit orders, consuming existing liquidity. Pays the spread.

**Maker Fee / Taker Fee**
Exchanges often charge takers and rebate makers to incentivize liquidity provision. "Maker-taker model."

**Informed Trader**
A trader with an informational edge — they know something the market maker doesn't, and trade to exploit it. The market maker's adversary.

**Uninformed Trader (Noise Trader)**
Trades for reasons unrelated to price information (liquidity needs, portfolio rebalancing). The market maker's preferred counterparty.

**Institutional Investor**
Large entities (pension funds, asset managers) executing large orders over time to minimize market impact.

**Retail Trader**
Individual trader with small order sizes. Generally uninformed, low market impact.

**HFT (High Frequency Trader)**
Firms that trade at extremely high speed (microsecond to millisecond) using automated strategies. Often market makers or statistical arbitrageurs.

**Prop Trader (Proprietary Trader)**
A firm trading its own capital (not client money) for direct profit.

**Quant (Quantitative Analyst)**
A specialist in mathematical and statistical modeling of financial markets. Develops signals, models, and strategies.

**Algo Trader**
Uses automated algorithms to execute trading strategies. May or may not be high frequency.

---

## 8. Strategies

**Market Making**
Continuously posting both bid and ask orders to earn the spread. Requires fast execution and careful inventory management.

**Statistical Arbitrage (StatArb)**
Exploiting statistical relationships between instruments. Pairs trading is a simple example.

**Pairs Trading**
Trading two historically correlated instruments: when they diverge, short the outperformer and long the underperformer, betting on convergence.

**Latency Arbitrage**
Exploiting price differences between venues by being faster than other participants. If the same stock shows different prices on two exchanges, buy on the cheap one and sell on the expensive one before the gap closes.

**Mean Reversion**
The hypothesis that prices tend to return to their historical average after deviating. Buy when price is "too low," sell when "too high." Works in range-bound markets, fails in trending markets.

**Momentum (Trend Following)**
The hypothesis that prices that have been rising will continue to rise (and vice versa). Buy breakouts, sell breakdowns. Works in trending markets, fails in range-bound markets.

**Moving Average Crossover**
A momentum signal: when a short-term moving average crosses above a long-term moving average, go long; when it crosses below, go short.

**Bollinger Bands**
Price bands set at `mean ± N * std_dev`. Price at the upper band → overbought, potential short. Price at lower band → oversold, potential long. A mean-reversion signal with visual representation.

**RSI (Relative Strength Index)**
A momentum oscillator measuring the speed and magnitude of recent price changes. RSI > 70 = overbought (potential reversal). RSI < 30 = oversold (potential reversal).

**MACD (Moving Average Convergence Divergence)**
The difference between two exponential moving averages. Signal line crossovers used as entry/exit signals.

**Trend Following**
Systematic strategy of buying assets in uptrends and shorting assets in downtrends. Profits from large, sustained moves.

**Mean Reversion vs Momentum**
These are opposite philosophies. The same market can be mean-reverting at short time scales and trending at longer time scales. Regime detection is required to know which to apply.

**Market Regime**
The current "state" of the market — trending, range-bound, high volatility, low volatility. Different strategies work in different regimes.

**TWAP (Time-Weighted Average Price)**
An execution algorithm that splits a large order into equal-sized pieces over equal time intervals. Minimizes timing risk.

**VWAP (Volume-Weighted Average Price)**
An execution algorithm that splits a large order proportionally to market volume. Widely used as a performance benchmark.

**Execution Algorithm**
An algorithm responsible for how (not whether) to trade — minimizing market impact, timing, and cost of executing a given target position.

**Signal**
A quantitative indicator suggesting a trading action. A signal alone is not a strategy — it must be combined with position sizing and risk management.

**Alpha**
Return in excess of a benchmark (e.g., buy-and-hold). "Generating alpha" means your strategy beats passive investing.

**Beta**
Sensitivity of a portfolio's returns to market returns. Beta = 1 means your portfolio moves with the market.

**Market Neutral**
A strategy with zero net market exposure (equal long and short). PnL comes from relative performance, not market direction. Pairs trading is market neutral.

**Long/Short Equity**
A strategy that holds long positions in stocks expected to rise and short positions in stocks expected to fall.

**High Frequency Trading (HFT)**
Trading strategies that operate at very high speeds (microseconds to milliseconds). Includes market making, latency arbitrage, and statistical arbitrage.

---

## 9. Backtesting & Research

**Backtesting**
Simulating a strategy on historical data to estimate its past performance. Does not guarantee future performance.

**In-Sample (Train Set)**
The historical period used to develop and calibrate a strategy. Parameters are optimized on this data.

**Out-of-Sample (Test Set)**
Historical data not used during development. Used to validate that the strategy generalizes beyond what it was built on.

**Walk-Forward Analysis**
A more rigorous backtesting framework: repeatedly train on a rolling window, test on the next period, roll forward. Reduces overfitting risk.

**Overfitting (Curve Fitting)**
Calibrating a strategy so precisely to historical data that it performs well in-sample but poorly out-of-sample. The strategy learned the noise, not the signal.

**Look-Ahead Bias**
Using information in the backtest that would not have been available at the time of the trade. Makes backtest results artificially good. Example: using today's closing price to generate and execute a signal on the same day.

**Survivorship Bias**
Backtesting only on companies that still exist today, ignoring ones that went bankrupt. Produces optimistic results.

**Transaction Costs**
Commissions, exchange fees, and slippage. Must be included in backtests or results are unrealistic.

**Slippage Model**
An assumption about how much worse than the mid price you will actually execute. Realistic models are critical for high-frequency strategies.

**Paper Trading**
Running a strategy in a simulated environment with real market data but no actual money. Used to validate live performance before deployment.

**Forward Test (Live Test)**
Running a strategy live with real money (or in paper trading) after backtesting. The true test of whether a strategy works.

**P-Hacking (Multiple Testing Problem)**
If you test 100 strategy variations and pick the best one, you will likely find one that looks good by pure chance. Any p-value below 0.05 loses meaning when many tests are run.

**Monte Carlo Simulation**
Generating thousands of random scenarios (e.g. return sequences) to estimate the distribution of outcomes. Used for stress testing and risk estimation.

---

## 10. Performance Metrics

**Sharpe Ratio**
`(strategy_return - risk_free_rate) / std_dev(returns)`, annualized. Measures return per unit of risk. Higher is better. Rule of thumb: > 1 acceptable, > 2 good, > 3 excellent.

**Sortino Ratio**
Like Sharpe, but only penalizes downside volatility (negative returns). More appropriate if upside volatility is acceptable.

**Calmar Ratio**
`annualized_return / max_drawdown`. Measures return relative to worst-case loss.

**Max Drawdown**
Largest peak-to-trough decline in equity. Key measure of risk tolerance — "can you survive this drawdown emotionally and financially?"

**Win Rate**
Percentage of trades that are profitable. A strategy can have a low win rate and still be profitable if winners are much larger than losers.

**Profit Factor**
`gross_profit / gross_loss`. > 1 means the strategy makes money overall. > 2 is considered good.

**Average Win / Average Loss**
"Risk-reward ratio." Even a 40% win rate is profitable if average win is 3x the average loss.

**Annualized Return**
The compound annual growth rate of the strategy. `(final_equity / initial_equity)^(1/years) - 1`.

**Alpha (Jensen's Alpha)**
Return attributable to skill (not market exposure). Strategy return minus `beta × market_return`.

**Information Ratio**
`(strategy_return - benchmark_return) / tracking_error`. How consistently the strategy beats its benchmark.

**Turnover**
How frequently the portfolio changes. High turnover = high transaction costs.

---

## 11. Statistics

**Mean (μ)**
Average value of a dataset. `sum(values) / count`.

**Variance (σ²)**
Average squared deviation from the mean. Measures spread.

**Standard Deviation (σ)**
Square root of variance. Same units as the data. In finance = volatility.

**Volatility**
Standard deviation of returns, usually annualized. Low volatility = calm market. High volatility = large daily moves.

**Implied Volatility (IV)**
The market's expectation of future volatility, extracted from option prices using a pricing model (e.g. Black-Scholes). High IV = expensive options = market expects large moves.

**Normal Distribution (Gaussian)**
Bell-shaped distribution. 68% of data within ±1σ, 95% within ±2σ, 99.7% within ±3σ. Financial returns approximately follow this, but with fatter tails.

**Fat Tails**
More extreme events than normal distribution predicts. Real market returns have negative skew and excess kurtosis — crashes happen more often than the model says.

**Skewness**
Asymmetry of a distribution. Negative skew = long left tail (more extreme losses than gains).

**Kurtosis**
"Peakedness" of a distribution. Excess kurtosis (leptokurtosis) = fat tails.

**Z-Score**
`(value - mean) / std_dev`. How many standard deviations a value is from the mean. Dimensionless. Used in mean-reversion signals.

**P-Value**
The probability that the observed result occurred by chance under the null hypothesis. p < 0.05 is conventionally "statistically significant."

**Null Hypothesis (H0)**
The baseline assumption to be tested. For a trading strategy: "this strategy has no edge" (expected return = 0).

**Correlation**
Linear relationship between two variables. Range: -1 (perfect inverse) to +1 (perfect positive). 0 = no linear relationship.

**Covariance**
Unnormalized version of correlation. `cov(A,B) = correlation(A,B) × σ_A × σ_B`.

**Rolling Window**
A fixed-size window that slides through time. Used to compute rolling mean, rolling std, rolling correlation. Captures "recent" behavior instead of all-history averages.

**Exponential Moving Average (EMA)**
A moving average that gives more weight to recent observations. Reacts faster to price changes than a simple moving average.

**Simple Moving Average (SMA)**
Equal weight to all observations in the window. Smoother but slower than EMA.

**Log Return**
`ln(P_t / P_{t-1})`. Additive across time: sum of daily log returns = total log return. Approximately equal to simple return for small changes.

**Simple Return (Arithmetic Return)**
`(P_t - P_{t-1}) / P_{t-1}`. The everyday percentage change. Not additive across time (compounding).

**Compound Return**
`(1 + r1) × (1 + r2) × ... × (1 + rn) - 1`. The actual cumulative return accounting for compounding.

**Annualization**
Scaling a return or volatility to a yearly basis. Returns: raise to the power of `250/n`. Volatility: multiply by `sqrt(250)` (250 trading days per year).

**Autocorrelation**
Correlation of a series with a lagged version of itself. Positive autocorrelation = momentum. Negative autocorrelation = mean reversion.

**Stationarity**
A time series is stationary if its statistical properties (mean, variance) don't change over time. Required assumption for many statistical models.

**Cointegration**
Two non-stationary series whose linear combination is stationary. The basis for pairs trading — even if two stock prices drift, their spread may be stationary.

---

## 12. Financial Instruments

**Equity (Stock / Share)**
Ownership stake in a company. Entitles holder to dividends and voting rights.

**Bond**
Debt instrument. The issuer borrows money and pays interest (coupon) over time, returning principal at maturity.

**Futures**
A standardized contract to buy/sell an asset at a specified price on a specified future date. Exchange-traded. Used for hedging and speculation.

**Options**
The right (not obligation) to buy (call) or sell (put) an asset at a specified price (strike) before or on a specified date (expiry).

**Call Option**
The right to buy the underlying at the strike price. Profitable if price rises above strike.

**Put Option**
The right to sell the underlying at the strike price. Profitable if price falls below strike.

**Strike Price (Exercise Price)**
The price at which an option can be exercised.

**Expiry (Expiration Date)**
The date after which an option can no longer be exercised.

**Premium**
The price paid to buy an option. The option seller receives this upfront.

**In The Money (ITM)**
An option with intrinsic value. Call: current price > strike. Put: current price < strike.

**Out of The Money (OTM)**
An option with no intrinsic value (only time value). Call: current price < strike.

**At The Money (ATM)**
Current price ≈ strike price. Delta ≈ 0.5 for calls, ≈ -0.5 for puts.

**Forward Contract**
Agreement to buy/sell an asset at a future date for a price agreed today. OTC (not exchange-traded). Pricing: `F = S × (1 + r)^T`.

**Swap**
An OTC contract to exchange cash flows. Interest rate swaps, currency swaps, equity swaps.

**ETF (Exchange-Traded Fund)**
A fund that holds a basket of assets and trades on an exchange like a stock. Tracks an index or sector.

**ADR (American Depositary Receipt)**
A US-listed security representing shares in a foreign company. Allows US investors to trade foreign stocks on US exchanges.

---

## 13. Market Microstructure

**Market Microstructure**
The study of how trading mechanisms, order flow, and information affect price formation.

**Price Discovery**
The process by which a market determines the "fair" price of an asset through the interaction of supply and demand.

**Information Asymmetry**
One market participant knows more than another. Informed traders exploit this; market makers try to detect and manage it.

**Adverse Selection**
The risk that the counterparty in a trade is better informed. A market maker who fills an order from an informed trader will systematically lose.

**Order Flow Toxicity**
A measure of how much of your order flow comes from informed traders (bad) vs uninformed traders (good). High toxicity = market maker is being adversely selected.

**VPIN (Volume-synchronized Probability of Informed Trading)**
A real-time measure of order flow toxicity. Market makers widen spreads when VPIN is high.

**Quote Stuffing**
Flooding the market with large numbers of orders that are quickly cancelled, slowing down competitors' systems. A form of market manipulation.

**Spoofing**
Placing large orders with the intent to cancel them before execution, to manipulate perceived supply/demand. Illegal in most jurisdictions.

**Layering**
A form of spoofing where multiple fake orders are placed at different price levels to create a false impression of market depth.

**Wash Trading**
Trading with yourself (both buyer and seller) to create artificial volume. Illegal.

**Frontrunning**
Trading ahead of a known large client order using information about that order. Illegal for brokers; similar activity by HFT (detecting order flow patterns) is legal but controversial.

**Payment for Order Flow (PFOF)**
A practice where brokers sell their client order flow to market makers. The market maker pays for the right to be on the other side of retail orders (which are uninformed and thus profitable to trade against).

**Co-location**
Placing servers in the same data center as the exchange matching engine to minimize physical latency.

**Latency Arbitrage**
Exploiting price differences between venues by being faster than other participants at detecting and responding to the discrepancy.

---

## 14. Protocols & Technology

**FIX (Financial Information eXchange)**
The dominant industry protocol for order submission and execution reporting. Tag-value format (e.g. `35=D` = new order). Used by most exchanges and brokers.

**ITCH**
NASDAQ's market data protocol. Binary, unidirectional (exchange → client). Delivers the full order book feed.

**OUCH**
NASDAQ's order entry protocol. Binary. Used to submit and manage orders on NASDAQ.

**SoupBinTCP**
NASDAQ's session layer protocol, used under ITCH and OUCH. Handles connection, sequencing, and heartbeats.

**MoldUDP64**
NASDAQ's multicast market data protocol. UDP-based for lower latency than TCP.

**Binary Protocol**
Messages encoded in binary (not human-readable). Smaller messages, faster parsing. ITCH and OUCH are binary.

**TCP (Transmission Control Protocol)**
Reliable, ordered delivery of data. Guarantees messages arrive and in order. Used for order entry (OUCH over SoupBinTCP).

**UDP (User Datagram Protocol)**
No delivery guarantees, no ordering. Lower overhead and latency. Used for market data multicast (MoldUDP64). Gaps must be handled by the application.

**Socket Programming**
Low-level network programming using OS sockets. Required for custom exchange connectivity.

**Kernel Bypass**
Routing network packets directly from the NIC to user space, bypassing the OS kernel. Reduces latency by eliminating kernel networking stack overhead (~1-5 microseconds saved). Techniques: DPDK, Solarflare OpenOnload.

**DPDK (Data Plane Development Kit)**
An open-source framework for kernel bypass networking. Allows user-space packet processing at line rate.

**RDMA (Remote Direct Memory Access)**
Allows data to be transferred directly between the memory of two machines without involving either CPU. Used for ultra-low latency inter-system communication.

**FPGA (Field-Programmable Gate Array)**
Reconfigurable hardware. In HFT: used to parse market data and generate/send orders entirely in hardware, achieving sub-100ns latency. Programmed in Verilog or VHDL.

**Lock-Free Data Structure**
A concurrent data structure that does not use mutexes. Uses atomic operations (CAS, load/store with memory ordering) instead. Avoids scheduling jitter from lock contention.

**SPSC Queue (Single Producer Single Consumer)**
A lock-free queue optimized for one producer thread and one consumer thread. Maximum throughput with minimal overhead.

**MPMC Queue (Multiple Producer Multiple Consumer)**
A concurrent queue supporting multiple producers and consumers. Requires more sophisticated synchronization (CAS loops).

**Memory Ordering**
The rules governing the visibility of memory operations across threads. `acquire/release` is the standard pair for producer-consumer. `seq_cst` is the strongest (and most expensive) ordering.

**False Sharing**
When two threads access different variables that happen to reside on the same CPU cache line. Causes unnecessary cache invalidation and performance degradation. Fixed with `alignas(64)`.

**Cache Line**
The unit of data transfer between CPU cache and RAM. Typically 64 bytes. Critical for performance: hot data should fit in as few cache lines as possible.

**SIMD (Single Instruction Multiple Data)**
CPU instructions that operate on multiple data elements simultaneously. Used in HFT for fast market data parsing (e.g. SIMD FIX parser).

**Core Pinning (Thread Affinity)**
Binding a thread to a specific CPU core so the OS scheduler never migrates it. Eliminates scheduling jitter.

**SCHED_FIFO**
A Linux real-time scheduling policy. The thread runs until it voluntarily yields or is preempted by a higher-priority real-time thread. Used for trading threads to minimize latency jitter.

**Busy-Wait (Spin Loop)**
Continuously polling a variable instead of sleeping and waiting for a notification. Eliminates the wakeup latency of a sleep/notify mechanism. Used on hot paths where even microseconds matter.

---

## 15. Position Sizing & Portfolio Theory

**Position Sizing**
Determining how much capital to allocate to each trade or instrument.

**Kelly Criterion**
A formula for optimal position sizing to maximize long-term capital growth: `f = (bp - q) / b` where `b` = odds, `p` = win probability, `q = 1-p`. Often used at a fraction (half-Kelly) to reduce variance.

**Risk-Reward Ratio**
`potential_gain / potential_loss` for a trade. A 3:1 ratio means you risk 1 to potentially gain 3.

**Diversification**
Holding multiple uncorrelated assets to reduce portfolio risk. "Don't put all your eggs in one basket."

**Correlation Matrix**
A table showing pairwise correlations between all assets in a portfolio. Essential for understanding diversification benefits.

**Portfolio Optimization (Mean-Variance Optimization)**
Markowitz's framework for finding the portfolio weights that maximize expected return for a given level of risk.

**Efficient Frontier**
The set of portfolios that offer the highest expected return for each level of risk. Developed by Markowitz.

**Beta**
A measure of a security's systematic risk relative to the market. Beta = 1 moves with the market. Beta > 1 is more volatile. Beta < 0 moves inversely.

**Hedging Ratio**
The proportion of a position offset by a hedge. A 1:1 hedging ratio means the hedge fully offsets the position.

---

## 16. Arbitrage

**Arbitrage**
Simultaneously buying and selling equivalent assets in different markets to profit from a price difference. Theoretically risk-free. In practice, speed and execution risk exist.

**No-Arbitrage Condition**
The principle that identical assets must trade at the same price (after adjusting for costs). Arbitrageurs enforce this by trading whenever it's violated.

**Cash-and-Carry Arbitrage**
Buy the spot asset now, sell the futures contract, deliver the asset at expiry. Profits if the futures price is above its fair value `S × (1+r)^T`.

**Index Arbitrage**
Exploiting price differences between an index ETF and the underlying basket of stocks.

**Triangular Arbitrage**
In FX: exploiting inconsistencies between three currency pairs. e.g. USD → EUR → GBP → USD returns more than 1 USD.

**Regulatory Arbitrage**
Exploiting differences in rules between jurisdictions. Not market arbitrage; more of a legal/business strategy.

---

## 17. Costs & Fees

**Commission**
Fee charged by a broker per trade, typically a flat fee or per-share rate.

**Exchange Fee**
Fee charged by the exchange per trade. Separate from broker commission.

**Spread Cost**
The implicit cost of crossing the bid-ask spread. Buying at the ask and selling at the bid = you paid the full spread.

**Basis Points (bps)**
1 bps = 0.01%. Used to express fees, spreads, and returns. "5 bps commission" = 0.05%.

**Rebate**
Payment made by the exchange to liquidity providers (market makers). Negative fee — you get paid for posting limit orders that get filled.

**Maker Rebate / Taker Fee**
In maker-taker pricing: posting a resting limit order earns a rebate; sending a market order or aggressive limit order incurs a fee.

**Short Interest**
The percentage of a stock's float that is currently sold short. High short interest → potential short squeeze.

**Borrow Cost**
The fee charged to borrow shares for short selling. High borrow cost for heavily shorted or illiquid stocks.

---

*This glossary covers the core terminology used in trading, HFT, quantitative finance, and trading system development. Terms are grouped by domain but concepts are deeply interconnected — understanding one category deeply illuminates the others.*
