# `<numeric>` — Numeric Algorithms

| API | Priority | Description |
|-----|----------|-------------|
| `std::accumulate(b, e, init)` | memorize | Sum. init=0. For custom operation, 4th argument: `accumulate(b,e,1,multiplies<>{})`. |
| `std::reduce(b, e, init)` | know | Like accumulate but has a parallel version. No ordering guarantee. |
| `std::inner_product(b,e,b2,init)` | know | Dot product. For multiply-sum operations like PnL calculation. |
| `std::partial_sum(b, e, out)` | know | Prefix sum. For cumulative calculations. |
| `std::iota(b, e, val)` | memorize | Fill with val, val+1, val+2... For test data, index arrays. |
| `std::gcd(a, b)` | know | Greatest common divisor. C++17. |
| `std::lcm(a, b)` | know | Least common multiple. C++17. |
| `std::midpoint(a, b)` | know | Overflow-safe midpoint. C++20. Useful for bid-ask midpoint. |
