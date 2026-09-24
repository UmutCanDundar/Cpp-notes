# `<cmath>`

### Notation
* `x`, `y`, `z`: floating-point arguments (`float`, `double`, or `long double` — all functions below are overloaded for the three).
* `n`: integer argument (exponent, count, etc.).
* Return type is written as `double` throughout for brevity; the actual return type matches the widest floating-point argument type (e.g. calling with `float` args returns `float`).

---

### Basic & Rounding

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| `double` / `int` | abs / fabs | `std::abs(x)`<br>`std::fabs(x)` | O(1) | `abs` has an integer overload (also in `<cstdlib>`); `fabs` is float/double-only. Use `fabs` when you specifically want the floating-point version. |
| `double` | floor / ceil | `std::floor(x)`<br>`std::ceil(x)` | O(1) | Rounds down to / up to the nearest integer value (still returned as a float type). |
| `double` | round | `std::round(x)` | O(1) | Rounds to the nearest integer; halfway cases round away from zero. |
| `double` | trunc | `std::trunc(x)` | O(1) | Rounds toward zero (drops the fractional part). |
| `double` | nearbyint / rint | `std::nearbyint(x)`<br>`std::rint(x)` | O(1) | Round using the current rounding mode (usually "to nearest, ties to even" — matches IEEE 754 default). `rint` may raise the inexact FP exception; `nearbyint` never does. |

---

### Power, Root & Exponential

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| `double` | sqrt / cbrt | `std::sqrt(x)`<br>`std::cbrt(x)` | O(1) | Square root / cube root. `cbrt` also works correctly for negative `x` (unlike `pow(x, 1.0/3)`). |
| `double` | pow | `std::pow(base, exp)` | O(1) (but slow) | General power function. Slow compared to manual multiplication for small integer exponents (`x*x*x` beats `pow(x,3)`). |
| `double` | hypot | `std::hypot(x, y)`<br>`std::hypot(x, y, z)` (C++17) | O(1) | `sqrt(x²+y²)` (or `x²+y²+z²`) computed without intermediate overflow/underflow. |
| `double` | exp / exp2 / expm1 | `std::exp(x)`<br>`std::exp2(x)`<br>`std::expm1(x)` | O(1) | `e^x` · `2^x` · `e^x - 1` computed accurately for small `x` (avoids cancellation error that `exp(x)-1` would have). |
| `double` | log / log2 / log10 / log1p | `std::log(x)`<br>`std::log2(x)`<br>`std::log10(x)`<br>`std::log1p(x)` | O(1) | Natural log · base-2 log · base-10 log · `log(1+x)` computed accurately for small `x`. |

---

### Trigonometric & Hyperbolic

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| `double` | sin / cos / tan | `std::sin(x)`<br>`std::cos(x)`<br>`std::tan(x)` | O(1) | Standard trig functions, `x` in radians. |
| `double` | asin / acos / atan | `std::asin(x)`<br>`std::acos(x)`<br>`std::atan(x)` | O(1) | Inverse trig functions, result in radians. |
| `double` | atan2 | `std::atan2(y, x)` | O(1) | Angle of the point `(x, y)` from the origin, correctly handling all four quadrants and `x == 0` (unlike plain `atan(y/x)`). |
| `double` | sinh / cosh / tanh | `std::sinh(x)`<br>`std::cosh(x)`<br>`std::tanh(x)` | O(1) | Hyperbolic sine / cosine / tangent. |
| `double` | asinh / acosh / atanh | `std::asinh(x)`<br>`std::acosh(x)`<br>`std::atanh(x)` | O(1) | Inverse hyperbolic functions. |

---

### Floating-Point Manipulation & Classification

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| `double` | fmod | `std::fmod(x, y)` | O(1) | Floating-point remainder of `x/y`, unlike `%` which is integer-only. Result has the same sign as `x`. |
| `double` | remainder | `std::remainder(x, y)` | O(1) | IEEE remainder of `x/y` (rounds the quotient to nearest, unlike `fmod`'s truncation) — result can be negative even if `x` is positive. |
| `double` | copysign | `std::copysign(x, y)` | O(1) | Returns `x` with the sign of `y`. Avoids branchy sign-fixing code. |
| `double` | fma | `std::fma(a, b, c)` | O(1) | Fused multiply-add, `a*b+c` with a single rounding — more accurate and often a single hardware instruction. |
| `double` | fmin / fmax / fdim | `std::fmin(x, y)`<br>`std::fmax(x, y)`<br>`std::fdim(x, y)` | O(1) | Min / max ignoring NaN (returns the non-NaN operand if one is NaN, unlike `<` comparisons) · positive difference (`max(x-y, 0)`). |
| `double` | frexp / ldexp | `std::frexp(x, &exp)`<br>`std::ldexp(x, n)` | O(1) | Decomposes `x` into mantissa × 2^exp · reconstructs `x * 2^n`. Useful for manual exponent manipulation. |
| `double` | modf | `std::modf(x, &intpart)` | O(1) | Splits `x` into integer and fractional parts (both returned as `double`, written via the out-param and the return value respectively). |
| `double` | nextafter | `std::nextafter(x, y)` | O(1) | The next representable floating-point value after `x` in the direction of `y`. Useful for ULP-level testing. |
| `double` | nan | `std::nan("")` | O(1) | Constructs a quiet NaN, optionally tagged with a string payload. |
| `bool` | isnan / isinf | `std::isnan(x)`<br>`std::isinf(x)` | O(1) | Check for NaN / infinity. Essential when validating price/quantity feeds. |
| `bool` | isfinite | `std::isfinite(x)` | O(1) | True if not NaN and not infinite. |
| `bool` | isnormal | `std::isnormal(x)` | O(1) | True if `x` is a "normal" float — not zero, subnormal, infinite, or NaN. |
| `bool` | signbit | `std::signbit(x)` | O(1) | True if the sign bit is set (works correctly on `-0.0`, unlike `x < 0`). |

---

### Special Functions

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| `double` | tgamma / lgamma | `std::tgamma(x)`<br>`std::lgamma(x)` | O(1) | The gamma function (generalizes factorial: `tgamma(n+1) == n!`) · its natural log (avoids overflow for large `x`). |
| `double` | erf / erfc | `std::erf(x)`<br>`std::erfc(x)` | O(1) | The error function (used in probability/statistics, e.g. the normal CDF) · its complement (`1 - erf(x)`, more accurate for large `x`). |
