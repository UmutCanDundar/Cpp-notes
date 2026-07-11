# `<execution>` (C++17)

| API | Priority | Description |
|-----|----------|-------------|
| `std::execution::seq` | know | Sequential execution policy — same as calling the algorithm without a policy. Baseline for comparison. |
| `std::execution::par` | memorize | Parallel execution across multiple threads. Use for CPU-bound work over large ranges, e.g. `std::sort(std::execution::par, v.begin(), v.end())`. |
| `std::execution::par_unseq` | careful | Parallel + vectorized (SIMD). Requires the operation to be free of data races and safe to interleave/reorder — no locks, no exceptions inside. |
| `std::execution::unseq` | know | C++20. Vectorized but single-threaded — allows SIMD reordering without spawning threads. |
| Applying a policy to small ranges | avoid | Thread-pool dispatch overhead can exceed the work itself for small `N` — measure before parallelizing. |
| Exceptions inside `par`/`par_unseq` | careful | Throwing inside a parallel algorithm body other than `bad_alloc`/policy-defined exceptions calls `std::terminate` — don't throw in the callback. |
