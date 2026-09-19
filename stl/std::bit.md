# `<bit>` (C++20)

### Notation
* `x`, `s`: unsigned integer type (`unsigned int`, `uint32_t`, `uint64_t`, etc.) — `s` is the rotate/shift amount, `x` is the value being inspected/manipulated.
* `T`: the deduced type of `x` — the return type matches the input type for the rotate/width/ceil/floor/swap functions.
* `From`, `To`: distinct types used only by `bit_cast`, which reinterprets the bit pattern of a `From` value as a `To` value.

---

| Return Type | Function Signature | Complexity | Description |
| :--- | :--- | :--- | :--- |
| `int` | `popcount(x)` | $O(1)$ | Counts the number of set (`1`) bits. Maps to the `popcnt` intrinsic. Used for bitmask operations. |
| `int` | `countl_zero(x)` | $O(1)$ | Counts consecutive `0` bits starting from the most significant bit. Maps to `lzcnt`. Used for computing `log2`. |
| `int` | `countr_zero(x)` | $O(1)$ | Counts consecutive `0` bits starting from the least significant bit. Gives the position of the lowest set bit. |
| `int` | `countl_one(x)` | $O(1)$ | Counts consecutive set (`1`) bits starting from the most significant bit. |
| `int` | `countr_one(x)` | $O(1)$ | Counts consecutive set (`1`) bits starting from the least significant bit. |
| `bool` | `has_single_bit(x)` | $O(1)$ | Checks if `x` is an exact power of 2. Replaces the `x && !(x & (x-1))` idiom. |
| `T` | `bit_ceil(x)` | $O(1)$ | Smallest power of 2 `>= x`. Used for buffer size calculations. |
| `T` | `bit_floor(x)` | $O(1)$ | Largest power of 2 `<= x`. Returns `0` if `x == 0`. |
| `T` | `bit_width(x)` | $O(1)$ | Number of bits needed to represent `x` (`0` if `x == 0`). |
| `T` | `rotl(x, s)` | $O(1)$ | Bitwise rotate left by `s` positions. Common in hashing code. |
| `T` | `rotr(x, s)` | $O(1)$ | Bitwise rotate right by `s` positions. Common in hashing code. |
| `To` | `bit_cast<To>(from)` | $O(1)$ | Type-punning without UB. Replaces `memcpy` reinterpret tricks and `reinterpret_cast` aliasing violations. Requires `From` and `To` to be the same size and both trivially copyable. |
| `T` | `byteswap(x)` (C++23) | $O(1)$ | Reverses the byte order of an integer (e.g. for endianness conversion). |
| `enum class` | `endian` | N/A (compile-time) | Enum to detect native byte order: `endian::little`, `endian::big`, `endian::native`. Usage: `endian::native == endian::little`. |
