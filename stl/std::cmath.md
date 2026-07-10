# `<cmath>`

| API | Priority | Description |
|-----|----------|-------------|
| `std::abs` / `std::fabs` | know | Absolute value. Use `fabs` for float/double, `abs` for int (or the `<cstdlib>` overload). |
| `std::floor` / `ceil` / `round` / `trunc` | know | Float rounding. `trunc` rounds toward zero, `round` to nearest. |
| `std::sqrt` / `std::cbrt` | know | Square root / cube root. |
| `std::pow(base, exp)` | careful | General power function. Slow compared to manual multiplication for small integer exponents. |
| `std::exp` / `std::log` / `std::log2` / `std::log10` | know | Exponential and logarithm functions. |
| `std::fmod(x, y)` | know | Floating point remainder, unlike `%` which is integer-only. |
| `std::isnan(x)` / `std::isinf(x)` | memorize | Check for NaN / infinity. Essential when validating price/quantity feeds. |
| `std::isfinite(x)` | know | True if not NaN and not infinite. |
| `std::copysign(x, y)` | know | Returns x with the sign of y. Avoids branchy sign-fixing code. |
| `std::fma(a, b, c)` | know | Fused multiply-add, `a*b+c` with a single rounding — more accurate and often a single instruction. |
| `std::hypot(x, y)` | know | sqrt(x²+y²) without intermediate overflow. |
