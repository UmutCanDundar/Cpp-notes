# HFT - Algo Trading Strategies & Concepts

---

## Strategy Types

### Market Making
- **What it is:** Continuously quoting both bid and ask. Earn the spread when both sides fill.
- **Edge:** Capturing bid-ask spread repeatedly, hundreds of thousands of times per day.
- **Risk:** Inventory risk (holding directional position), adverse selection (informed flow).
- **Key metric:** Fill rate, spread captured, adverse selection ratio.
- **HFT requirement:** Must update quotes faster than competitors. Co-location essential.

### Statistical Arbitrage (StatArb)
- **What it is:** Exploit temporary mispricings between related instruments using statistical models.
- **Example:** Pairs trading — two highly correlated stocks diverge → bet on mean reversion.
- **Key:** Mean-reversion signal + entry/exit z-score thresholds + position sizing.
- **Risk:** Regime change (correlation breaks permanently).

### Latency Arbitrage
- **What it is:** Exploit stale quotes on slower venues using faster market data.
- **Example:** See a trade on NYSE, hit the stale quote on BATS before it updates.
- **Key:** Co-location + kernel bypass + FPGA/ASIC.
- **Risk:** Arms race — only works if you're consistently faster than others.

### Momentum / Trend Following
- **What it is:** Buy assets going up, sell assets going down.
- **Timeframe in HFT:** Milliseconds to seconds (order flow imbalance as signal).
- **Signal:** Order book imbalance, trade initiation direction, recent price velocity.

### Mean Reversion
- **What it is:** Bet that prices will revert to a mean after a large move.
- **Timeframe in HFT:** Intraday, using tick data.
- **Signal:** Temporary price dislocation, low liquidity at extremes, inventory-driven moves.

---

## Order Types

| Order Type | Description | HFT Use |
|------------|-------------|---------|
| Market Order | Execute immediately at best available price | Never for making; sometimes for taking |
| Limit Order | Execute only at specified price or better | Core of market making |
| IOC (Immediate or Cancel) | Fill what you can now, cancel the rest | Latency arb, taking liquidity |
| FOK (Fill or Kill) | Fill entire order or cancel entirely | Large block execution |
| Stop Order | Becomes market order when price hits trigger | Risk management |
| Pegged Order | Price pegs to mid/bid/ask dynamically | Smart market making |
| Iceberg Order | Only shows part of the order size | Hiding intent |
| Post-Only | Only adds liquidity, never takes | Market making — avoid taker fees |

---

## Risk Management in HFT

**Q: What is a kill switch?**
A hard cutoff that immediately cancels all open orders and halts trading when a risk threshold is breached. Triggered by: max daily loss, max position size, max order rate, connectivity loss. Must be sub-millisecond in response.

**Q: What is fat finger risk?**
Sending an order with an incorrect price or quantity due to a software bug. E.g., selling 10,000 contracts instead of 10. Prevented by: max order size checks, price reasonability checks (price ± N × ATR), pre-trade risk validation.

**Q: What pre-trade risk checks must every HFT system have?**
1. Max order size
2. Max position size (long and short)
3. Max daily loss (P&L circuit breaker)
4. Price sanity check (not more than X% from last trade)
5. Max order rate (orders per second)
6. Duplicate order detection
7. Self-trade prevention

**Q: What is delta neutrality in market making?**
Keeping net position close to zero to avoid directional risk. Market makers hedge inventory continuously. If inventory grows too long, skew quotes to attract sellers.

**Q: What is quote skewing?**
Adjusting bid/ask prices based on current inventory. If long, lower the bid (less eager to buy more) and lower the ask (more eager to sell). Reduces adverse selection and inventory risk without canceling quotes.

---

## Exchange Connectivity

**Q: What is FIX protocol?**
Financial Information eXchange. Standard messaging protocol for order submission and execution reporting. Text-based (tag=value). Used by most exchanges and brokers. Verbose — HFT systems often use binary protocols (OUCH, ITCH, BOLT) directly.

**Q: What is ITCH?**
Market data protocol used by NASDAQ. Binary, UDP multicast. Each message is ~20 bytes. HFT systems parse ITCH feeds to reconstruct the full order book in real-time.

**Q: What is OUCH?**
Order entry protocol by NASDAQ. Binary TCP. For sending orders. Faster than FIX.

**Q: What is multicast and why is it used for market data?**
UDP multicast sends one packet to many subscribers simultaneously. The exchange sends a single market data stream that all co-located participants receive. Efficient — no per-subscriber bandwidth cost.

**Q: What is sequence number gap handling?**
Multicast UDP packets can be lost. Each packet has a sequence number. If a gap is detected (seq N+2 received, N+1 missing), the system must either request a retransmit or fall back to a recovery feed. Critical — a missed order can corrupt your order book.

---

## Performance Benchmarks (Know These Numbers)

| Operation | Approximate Latency |
|-----------|---------------------|
| L1 cache hit | ~4 cycles (~1ns) |
| L2 cache hit | ~12 cycles (~4ns) |
| L3 cache hit | ~40 cycles (~12ns) |
| RAM access | ~100-200 cycles (~60ns) |
| `rdtsc` instruction | ~1 cycle |
| Atomic CAS (uncontested) | ~5-10 cycles |
| `mfence` | ~20-50 cycles |
| Syscall overhead | ~1,000 cycles (~300ns) |
| Context switch | ~10,000 cycles (~3µs) |
| OS scheduler wakeup | ~50,000 cycles (~15µs) |
| Loopback network (software) | ~5-10µs |
| Co-location cross-connect | ~1µs |
| Kernel bypass NIC (DPDK/Solarflare) | ~100-200ns |
| FPGA-based order entry | ~1-10ns |

---

## FPGA vs CPU vs GPU in HFT

| Aspect | CPU | FPGA | GPU |
|--------|-----|------|-----|
| Latency | ~1µs | ~1-100ns | ~100µs |
| Flexibility | High | Low (requires RTL recompile) | Medium |
| Development cost | Low | Very high | Medium |
| Parallelism | Limited (cores) | Massive (pipeline) | Massive (CUDA cores) |
| Use in HFT | Order management, risk, strategy | Market data parsing, order entry | Research/backtesting, ML inference |
| Power | Low | Medium | High |

---

## Interview Brain Teasers (Common in HFT)

**Q: You have a 3-sided die (faces 1,2,3) and a 4-sided die (faces 1,2,3,4). You roll both. What is the probability that the 3-sided die shows a strictly higher value?**
P(3d > 4d): (2,1),(3,1),(3,2) = 3 outcomes out of 12. P = 3/12 = 1/4.

**Q: A stock has 50% chance of going up $1 and 50% chance of going down $1 each step. What is the expected time to reach $10 starting from $0, with absorbing barriers at -$10 and $10?**
Gambler's ruin with symmetric walk. Expected hitting time = N² - x² where N=10, x=0. E[T] = 100.

**Q: If you can bet on a biased coin (P(H)=0.6), what fraction of your bankroll should you bet each time to maximize long-run growth?**
Kelly: f* = p - q = 0.6 - 0.4 = 0.2 = 20%.

**Q: You observe 100 trades and win 55. Is your strategy profitable or just lucky?**
Binomial test: under null (p=0.5), std dev of wins = √(100 × 0.5 × 0.5) = 5. z = (55-50)/5 = 1.0. p-value ≈ 0.31. Not statistically significant. Need ~270 trades with 55% win rate for p < 0.05.

**Q: What is the difference between P&L and realized P&L?**
Realized P&L: from closed positions. Unrealized P&L: mark-to-market on open positions. In HFT risk systems, both are tracked in real-time. Daily P&L = realized + unrealized.

**Q: Why does a market maker make money even with a 50% win rate?**
They make the spread on every trade. If spread = 2 ticks and they fill 1,000 times/day = 2,000 ticks/day of gross P&L before adverse selection and market impact.
