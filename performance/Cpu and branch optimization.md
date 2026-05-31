# CPU and Branch Optimizations

| Technique | Example | What It Gains |
|-----------|---------|---------------|
| Branchless arithmetic | `int min = a < b ? a : b` → `int min = b + ((a-b) & ((a-b)>>31))` | No branch predictor miss. But measure first — compiler may already generate cmov. |
| `[[likely]]` / `[[unlikely]]` | `if (likely(x > 0))` | Hint to compiler for branch prediction. Puts the frequent path on hot path. |
| Switch → lookup table | switch with 5+ cases | Array index gives O(1) dispatch. No branch. |
| Loop unrolling | `#pragma GCC unroll 4` | Reduces loop overhead. Compiler usually does this automatically. |
| Function inlining | `__forceinline` / `inline` | No call overhead. Critical in small hot functions. |
| Prefetch | `__builtin_prefetch(ptr, 0, 1)` | Load next data into cache early. Effective for pointer chase. |
| Avoid division | `x / 4` → `x >> 2` | Division is expensive (~20-80 cycles). Use shift for powers of 2 — compiler already does this. |
| Strength reduction | `i * stride` in loop → `+= stride` each step | Addition instead of multiplication. Compiler usually does this automatically. |

> Branchless code is not always faster. A small, predictable branch = predictor is 99% correct = 0 cost. Don't change it without measuring.
