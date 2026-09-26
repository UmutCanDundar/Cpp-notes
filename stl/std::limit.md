# `<limits>`

### Notation
* `T`: any arithmetic type (`int`, `unsigned`, `float`, `double`, etc.) — `numeric_limits<T>` is specialized for each.
* All members below are `static constexpr` — called as `std::numeric_limits<T>::member` or `std::numeric_limits<T>::member()`, no object construction needed, so there is no Constructors section.

---

### Methods

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| `T` | max() | `std::numeric_limits<T>::max()` | O(1) (compile-time constant) | Largest finite value of `T`. Use instead of hardcoding `INT_MAX`. |
| `T` | min() | `std::numeric_limits<T>::min()` | O(1) | Smallest value of `T` — for integers this is the most negative value, but **for floats this is the smallest *positive* normal value**, not the most negative! |
| `T` | lowest() (C++11) | `std::numeric_limits<T>::lowest()` | O(1) | The true minimum (most negative) finite value — use this for floats, not `min()`. For integer types, identical to `min()`. |
| `T` | epsilon() | `std::numeric_limits<T>::epsilon()` | O(1) | Smallest representable difference between `1.0` and the next representable float/double value. Used for tolerance-based float comparison. `0` for integer types. |
| `T` | infinity() | `std::numeric_limits<T>::infinity()` | O(1) | `+∞` for float/double (only meaningful if `has_infinity` is true). `0` for integer types. |
| `T` | quiet_NaN() | `std::numeric_limits<T>::quiet_NaN()` | O(1) | A non-signaling NaN value. Useful as a "missing value" sentinel for prices/measurements. |
| `T` | signaling_NaN() | `std::numeric_limits<T>::signaling_NaN()` | O(1) | A NaN that raises a floating-point exception on most operations — used to catch accidental use of an uninitialized/invalid value. |
| `T` | denorm_min() | `std::numeric_limits<T>::denorm_min()` | O(1) | The smallest positive **subnormal** value (smaller than `min()`, with reduced precision). |
| `bool` | is_integer | `std::numeric_limits<T>::is_integer` | O(1) (compile-time bool) | Whether `T` is an integer type. |
| `bool` | is_signed | `std::numeric_limits<T>::is_signed` | O(1) | Whether `T` can hold negative values. |
| `bool` | is_exact | `std::numeric_limits<T>::is_exact` | O(1) | Whether all arithmetic on `T` is exact (`true` for integers, `false` for floating point due to rounding). |
| `bool` | is_iec559 | `std::numeric_limits<T>::is_iec559` | O(1) | Whether `T` conforms to IEEE 754 — determines whether `infinity()`/NaN behave as expected. |
| `bool` | traps | `std::numeric_limits<T>::traps` | O(1) | Whether operations on `T` can trigger a hardware trap (e.g. integer division by zero on some platforms). |
| `int` | digits | `std::numeric_limits<T>::digits` | O(1) | Number of usable bits (integers) or mantissa bits (floats) for representing the value, excluding the sign bit. |
| `int` | digits10 | `std::numeric_limits<T>::digits10` | O(1) | Number of decimal digits guaranteed to round-trip without loss — the practical "how many digits can I trust" figure. |
| `int` | max_digits10 | `std::numeric_limits<T>::max_digits10` | O(1) | Number of decimal digits needed so that parsing the printed text back always reproduces the exact original float value. |
| `int` | radix | `std::numeric_limits<T>::radix` | O(1) | The base used to interpret `digits` — `2` for binary floats/integers. |
| `int` | min_exponent / max_exponent | `std::numeric_limits<T>::min_exponent`<br>`std::numeric_limits<T>::max_exponent` | O(1) | Smallest/largest exponent (base `radix`) such that `radix^(exponent-1)` is a representable normalized float. |
