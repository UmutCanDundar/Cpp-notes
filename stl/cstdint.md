# `<cstdint>`

| API | Priority | Description |
|-----|----------|-------------|
| `uint8_t` / `uint16_t` / `uint32_t` / `uint64_t` | memorize | Fixed-size unsigned integers. Required in network protocols and binary layouts. |
| `int8_t` / `int16_t` / `int32_t` / `int64_t` | memorize | Fixed-size signed integers. For price, PnL calculation. |
| `int_fast32_t` / `uint_fast64_t` | know | "At least N bits, whatever is fastest on this platform" — for loop counters where exact width doesn't matter. |
| `int_least32_t` | know | "At least N bits, smallest such type" — for portability when exact size matters less than a guaranteed minimum. |
| `intptr_t` / `uintptr_t` | know | Integer types large enough to hold a pointer. For pointer arithmetic/hashing as integers. |
| `INT32_MAX` / `UINT64_MAX` etc. | know | Macro constants, C-style equivalent of `numeric_limits<T>::max()`. |
| `SIZE_MAX` | know | Maximum value of `size_t`. |
