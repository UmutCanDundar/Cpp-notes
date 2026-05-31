# Optimization Order — Big to Small First

| # | Step |
|---|------|
| 1 | **Measure first — profile** Don't guess, measure. Optimizing the wrong place is the most common mistake. Find the hot path with `perf`, `gprof`, `valgrind --tool=callgrind`, `rdtsc`. 80% of time is spent in 20% of the code. |
| 2 | **Algorithm complexity — O(n²) → O(n log n) → O(n)** The biggest gains are here. Is there a loop inside a loop? Can a hash map be used? Lookup table instead of binary search? No micro-optimization can beat an O(n²) → O(n) change. |
| 3 | **Memory allocation — remove from hot path** If you see malloc/new/delete on the hot path — red flag. Replace with object pool, arena allocator, stack allocation. Is vector's push_back causing a realloc? Add `reserve()`. |
| 4 | **Cache access pattern** Is data accessed sequentially (cache-friendly) or randomly (cache miss)? Vector instead of linked list. SoA instead of AoS. Hot and cold data separated? Pointer chase chains present? These affect throughput by 10x. |
| 5 | **Branches and branching** Is there heavy if/else on the hot path? Branch predictor miss = ~15 cycles. Lookup table, branchless arithmetic, `[[likely]]`/`[[unlikely]]` hints. Function pointer array instead of switch. |
| 6 | **Help the compiler** `inline`, `[[nodiscard]]`, `__builtin_expect`, `__attribute__((hot))`, `restrict`. Const correctness — compiler optimizes better. Can loop-invariant computation be hoisted outside the loop? |
| 7 | **SIMD / vectorization** Did the loop auto-vectorize? Check on Godbolt with `-O2 -march=native`. Is data aligned? `alignas(32)`. If compiler can't do it, write intrinsics (SSE/AVX) manually — but measure first. |
| 8 | **Micro-optimization — last** Register spills, instruction pairing, pipeline stalls — don't look at these until the profiler shows them. Early micro-optimization is usually a waste of time and hurts readability. |

> In an interview, when asked to "optimize," your first question should be: "Did you profile? Where is the bottleneck?" — Asking this makes you look senior.
