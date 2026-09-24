# `<execution>` (C++17)

### Notation
* `ExecutionPolicy`: one of the tag objects below, passed as the **first argument** to a parallel-enabled algorithm overload, e.g. `std::sort(std::execution::par, v.begin(), v.end())`.
* These tags are not user-constructible — you never write `execution::parallel_policy{}` yourself; you use the predefined global objects (`seq`, `par`, `par_unseq`, `unseq`) as-is. There is no Constructors section for this header for that reason.

---

### Execution Policies (global constexpr tag objects)

| Type | API | Usage | Complexity | Note |
|------|-----|-------|------------|------|
| `execution::sequenced_policy` | seq | `std::execution::seq` | N/A — forces no parallelism | Sequential execution — same as calling the algorithm without a policy at all. Baseline for correctness/perf comparison. |
| `execution::parallel_policy` | par | `std::execution::par` | N/A — spreads work across threads | Parallel execution across multiple threads. Use for CPU-bound work over large ranges. |
| `execution::parallel_unsequenced_policy` | par_unseq | `std::execution::par_unseq` | N/A — parallel + vectorized | Parallel + vectorized (SIMD). Requires the operation to be free of data races and safe to interleave/reorder — no locks, no exceptions inside. |
| `execution::unsequenced_policy` (C++20) | unseq | `std::execution::unseq` | N/A — vectorized only | Vectorized but single-threaded — allows SIMD reordering without spawning threads. |

---

### Type Traits

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| `bool` | is_execution_policy_v | `std::is_execution_policy_v<T>` | O(1) (compile-time) | Checks whether `T` is one of the standard execution policy types. Used in generic code that is itself templated on the policy type. |

> **Applying a policy to small ranges — avoid.** Thread-pool dispatch overhead can exceed the work itself for small `N` — measure before parallelizing.

> **Exceptions inside `par`/`par_unseq` — careful.** Throwing inside a parallel algorithm's callback, other than `bad_alloc` or a policy-defined exception, calls `std::terminate` — don't throw in the callback.

> **Where this is actually used:** most `<algorithm>` functions (`sort`, `for_each`, `find`, `transform`, `copy`, ...) and the C++17 numeric algorithms in `<numeric>` (`reduce`, `transform_reduce`, `inclusive_scan`, `exclusive_scan`, ...) gained an overload that takes an `ExecutionPolicy` as the first parameter — this header only defines the policy tags themselves, not the algorithms that consume them.
