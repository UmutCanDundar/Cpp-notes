# `<limits>` `<bit>` `<cmath>` `<cstdint>`

| API | Priority | Description |
|-----|----------|-------------|
| `std::numeric_limits<T>::max()` | memorize | Max value of T. Use instead of INT_MAX. |
| `std::numeric_limits<T>::min()` | memorize | Min value of T. For floats, this is the smallest positive value! |
| `std::numeric_limits<T>::lowest()` | memorize | True minimum for floats (negative). Do not confuse with min(). |
| `std::numeric_limits<T>::infinity()` | know | +∞ for float/double. |
| `std::numeric_limits<T>::is_integer` | know | Compile-time bool. |
| `std::popcount(x)` | memorize | C++20. Count of set bits. → `popcnt` intrinsic. For bitmask operations. |
| `std::countl_zero(x)` | know | C++20. Count of leading zeros. → `lzcnt`. For computing log2. |
| `std::countr_zero(x)` | know | C++20. Count of trailing zeros. Position of lowest set bit. |
| `std::has_single_bit(x)` | know | C++20. Is it a power of 2? Instead of `x && !(x & (x-1))`. |
| `std::bit_ceil(x)` | know | C++20. Smallest power of 2 >= x. For buffer size calculation. |
| `std::byteswap(x)` | memorize | C++23. Swap byte order. Network → host byte order. Instead of htonl/ntohl. |
| `uint8_t/uint16_t/uint32_t/uint64_t` | memorize | `<cstdint>`. Fixed-size integers. Required in network protocols. |
| `int64_t` / `int32_t` | memorize | Signed fixed-size. For price, PnL calculation. |
| `std::abs` / `std::fabs` | know | `<cmath>`. Absolute value. Use fabs for float. |
| `std::floor` / `ceil` / `round` / `trunc` | know | `<cmath>`. Float rounding. |
