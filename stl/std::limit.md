# `<limits>`

| API | Priority | Description |
|-----|----------|-------------|
| `std::numeric_limits<T>::max()` | memorize | Max value of T. Use instead of INT_MAX. |
| `std::numeric_limits<T>::min()` | memorize | Min value of T. For floats, this is the smallest positive value! |
| `std::numeric_limits<T>::lowest()` | memorize | True minimum for floats (negative). Do not confuse with min(). |
| `std::numeric_limits<T>::epsilon()` | know | Smallest representable difference for float/double. For safe float comparison. |
| `std::numeric_limits<T>::infinity()` | know | +∞ for float/double. |
| `std::numeric_limits<T>::quiet_NaN()` | know | Non-signaling NaN value. Useful as a "missing value" sentinel for prices. |
| `std::numeric_limits<T>::is_integer` | know | Compile-time bool. |
| `std::numeric_limits<T>::is_signed` | know | Compile-time bool, whether T can hold negative values. |
| `std::numeric_limits<T>::digits` | know | Number of bits (or decimal digits for floats) usable for the value. |
