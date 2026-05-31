# HFT — General Prep & Behavioral Questions

---

## What HFT Firms Actually Test

| Category | Weight | What They Test |
|----------|--------|----------------|
| C++ / Systems | High | Memory, concurrency, low-latency patterns, assembly awareness |
| Probability / Stats | High | Brain teasers, expected value, distributions |
| Math / Algorithms | High | Big O, data structures, numerical methods |
| Market Microstructure | Medium | Order book, spread, market making |
| Algo Trading | Medium | Strategy logic, risk management, order types |
| Behavioral | Low-Medium | Communication, debugging process, learning speed |

---

## Typical HFT Interview Process

1. **Phone screen** — 1-2 coding questions (LeetCode medium/hard), maybe a probability puzzle
2. **Technical round 1** — C++ deep dive: memory model, atomics, cache, virtual dispatch
3. **Technical round 2** — Algorithms + data structures + complexity analysis
4. **Quant round** — Probability puzzles, expected value, statistics
5. **System design** — Design a matching engine / market data feed handler / order management system
6. **Behavioral / Culture fit** — Why HFT? How do you debug under pressure?

---

## System Design Questions

**Design a lock-free SPSC ring buffer.**
- Fixed-size array, power-of-2 size (for bitmask wrap)
- `head` (producer writes), `tail` (consumer reads) — both `atomic<uint64_t>`
- Producer: load tail (relaxed), check not full, write data, store head (release)
- Consumer: load head (acquire), check not empty, read data, store tail (release)
- No locks, no syscalls, no false sharing — pad head and tail to separate cache lines

**Design a low-latency market data feed handler.**
- UDP multicast receive on dedicated NIC (kernel bypass preferred)
- Parse binary ITCH/OUCH packets in a tight loop
- Reconstruct order book in a price-level map (array-backed, not std::map)
- Sequence number gap detection → retransmit request
- Publish book updates via lock-free ring buffer to strategy thread
- All on a single pinned core, no allocations, no syscalls

**Design an order management system (OMS) for HFT.**
- Order lifecycle: New → Sent → Acked → Partially Filled → Filled / Cancelled
- State machine per order (enum + transition table, not virtual state objects)
- Object pool for order objects — no `new` on hot path
- Atomic counters for position tracking
- Pre-trade risk checks inline before send
- Kill switch: atomic flag checked on every order send

**Design a matching engine.**
- Price-time priority FIFO matching
- Order book: price levels sorted (std::map or sorted array)
- Each price level: FIFO queue of orders (linked list or ring buffer)
- Match: incoming aggressive order walks the book, fills resting orders
- Key: O(1) per match step — avoid scanning entire book

---

## Behavioral Questions & Good Answers

**Q: Why HFT specifically?**
"I want to work on systems where every microsecond matters — problems where software engineering, computer architecture, and mathematics all intersect. The performance constraints in HFT are more demanding than anywhere else in software."

**Q: How do you debug a latency spike?**
1. First: isolate — is it consistent or intermittent?
2. Profile with `perf` or `rdtsc` measurements at checkpoints
3. Check for: GC pauses (none in C++), memory allocation, lock contention, OS interrupts, NUMA effects, TLB misses
4. `perf c2c` for false sharing, `perf stat` for cache misses
5. Check if the spike correlates with market events (burst of messages)
6. Examine CPU frequency scaling (disable turbo boost for consistent latency)

**Q: Walk me through how you'd optimize a function that's identified as a hot path.**
1. Look at the generated assembly (Godbolt)
2. Check for unnecessary allocations, copies, virtual calls
3. Ensure data accessed is cache-friendly (sequential, no pointer chasing)
4. Check branch predictability
5. Consider SIMD if operating on arrays
6. Measure before and after with `rdtsc`, run 10,000+ iterations

**Q: What's the hardest bug you've debugged?**
Good answer structure: describe a concurrency bug (data race, false sharing, or memory ordering), explain how you systematically isolated it, what tool you used, and what you learned.

**Q: How do you stay current with C++ and HFT?**
- CppCon talks (especially: Herb Sutter, Timur Doumler, Chandler Carruth)
- cppreference.com for standard details
- "The Art of Writing Efficient Programs" (Pikus)
- Godbolt — understand what the compiler generates
- Following exchanges' technical documentation (ITCH, OUCH specs)

---

## Must-Know Papers & Books

| Resource | Why It Matters |
|----------|---------------|
| Almgren-Chriss (2000) | Optimal execution, market impact model |
| Avellaneda-Stoikov (2008) | Market making model with inventory and adverse selection |
| Glosten-Milgrom (1985) | Bid-ask spread decomposition (adverse selection component) |
| "Flash Boys" (Lewis) | Context — not technical but explains HFT perception |
| "Algorithmic Trading" (Narang) | Practical overview of stat arb strategies |
| "The Art of Writing Efficient Programs" (Pikus) | C++ performance, the best technical book for HFT C++ |
| "C++ Concurrency in Action" (Williams) | Atomics, memory model, lock-free |
| LMAX Disruptor whitepaper | Ring buffer + sequencer pattern, foundational HFT design |

---

## Quick-Reference: Things to Know Cold

- Cache line = 64 bytes
- L1 latency ≈ 1ns, RAM ≈ 60ns, syscall ≈ 300ns
- Kelly fraction = edge / variance = (p - q) for even-odds bets
- Sharpe > 2 is good in HFT
- `memory_order_acquire` on load, `release` on store for producer-consumer
- `alignas(64)` to prevent false sharing
- Ring buffer: power-of-2 size, `head & (size-1)` for wrap
- `rdtsc` for nanosecond measurement, not chrono
- `_mm_pause()` in spin loops
- Virtual function → replace with CRTP or variant+visit
- `std::function` → replace with template lambda or function pointer
- `shared_ptr` on hot path → replace with `unique_ptr` or raw pointer + pool
