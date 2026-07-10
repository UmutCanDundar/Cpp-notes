# `<bit>` (C++20)

| API | Priority | Description |
|-----|----------|-------------|
| `std::popcount(x)` | memorize | Count of set bits. → `popcnt` intrinsic. For bitmask operations. |
| `std::countl_zero(x)` | know | Count of leading zeros. → `lzcnt`. For computing log2. |
| `std::countr_zero(x)` | know | Count of trailing zeros. Position of lowest set bit. |
| `std::countl_one(x)` / `std::countr_one(x)` | know | Count leading/trailing set (1) bits. |
| `std::has_single_bit(x)` | know | Is it a power of 2? Instead of `x && !(x & (x-1))`. |
| `std::bit_ceil(x)` | know | Smallest power of 2 >= x. For buffer size calculation. |
| `std::bit_floor(x)` | know | Largest power of 2 <= x. |
| `std::bit_width(x)` | know | Number of bits needed to represent x. |
| `std::rotl(x, s)` / `std::rotr(x, s)` | know | Bitwise rotate left/right. Common in hashing code. |
| `std::bit_cast<To>(from)` | memorize | Type-punning without UB. Replaces `memcpy` reinterpret tricks and `reinterpret_cast` aliasing violations. |
| `std::endian` | know | Enum to detect native byte order (`endian::native == endian::little`). |
