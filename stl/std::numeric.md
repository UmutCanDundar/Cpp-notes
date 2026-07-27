# C++ Numeric Utilities & Algorithms (`<numeric>`)

### Notation
* `b`, `e`: Begin / End iterators
* `out`: Destination iterator
* `init`: Initial value
* `op`, `binary_op`: Custom binary operation (e.g., `std::multiplies<>{}`)
* `N`: Distance between iterators (`std::distance(b, e)`)

---

## 1. Numeric Algorithms (`<numeric>`)

| Return Type | API / Signature | Big O Complexity | Description |
| :--- | :--- | :--- | :--- |
| `T` | `accumulate(b, e, init)` | $O(N)$ | Computes the sum of `init` and all elements in the range. |
| `T` | `reduce(b, e, init)` | $O(N)$ | Out-of-order parallelizable reduction (sum). C++17. |
| `T` | `inner_product(b1, e1, b2, init)` | $O(N)$ | Computes the sum of element-wise products (Dot Product). |
| `T` | `transform_reduce(b, e, init, red_op, trans_op)` | $O(N)$ | Applies a transformation then reduces the range. C++17. |
| `OutIt` | `partial_sum(b, e, out)` | $O(N)$ | Computes cumulative prefix sums into `out`. |
| `OutIt` | `inclusive_scan(b, e, out)` | $O(N)$ | Parallelizable prefix sum including the current element. C++17. |
| `OutIt` | `exclusive_scan(b, e, out, init)` | $O(N)$ | Parallelizable prefix sum excluding the current element. C++17. |
| `OutIt` | `transform_inclusive_scan(b, e, out, op, trans_op)` | $O(N)$ | Transforms elements then performs an inclusive scan. C++17. |
| `OutIt` | `transform_exclusive_scan(b, e, out, init, op, trans_op)` | $O(N)$ | Transforms elements then performs an exclusive scan. C++17. |
| `OutIt` | `adjacent_difference(b, e, out)` | $O(N)$ | Computes differences between adjacent elements. |
| `void` | `iota(b, e, val)` | $O(N)$ | Fills range with sequentially increasing values starting from `val`. |
| `T` | `gcd(a, b)` | $O(\log(\min(a,b)))$ | Greatest Common Divisor of `a` and `b`. C++17. |
| `T` | `lcm(a, b)` | $O(\log(\min(a,b)))$ | Least Common Multiple of `a` and `b`. C++17. |
| `T` | `midpoint(a, b)` | $O(1)$ | Computes overflow-safe midpoint between `a` and `b`. C++20. |
| `T` | `lerp(a, b, t)` | $O(1)$ | Computes linear interpolation $a + t(b - a)$. C++20. |

