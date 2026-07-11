# `<bitset>`

| API | Priority | Description |
|-----|----------|-------------|
| `std::bitset<N>` | know | Fixed-size (compile-time N) sequence of bits, stored compactly (N/8 bytes). Not resizable, unlike `vector<bool>`. |
| `.set(i)` / `.reset(i)` / `.flip(i)` | memorize | Set, clear, or toggle a single bit. Without an index, applies to all bits. |
| `.test(i)` | memorize | Bounds-checked read of bit i (throws if out of range) — `operator[]` is the unchecked, faster version. |
| `.count()` | know | Number of set bits — similar to `std::popcount` but works on the whole fixed-width set. |
| `.any()` / `.all()` / `.none()` | know | Quick predicates on the whole bitset. |
| `.to_ulong()` / `.to_ullong()` | know | Converts to an integer, throws `overflow_error` if it doesn't fit. |
| `operator&` / `operator\|` / `operator^` | know | Bitwise set operations between two bitsets of the same size. |
| `std::bitset` vs raw integer bit-flags | careful | For N ≤ 64, a raw `uint64_t` with manual bit ops is usually faster (no bounds checks, direct register ops) — bitset shines mainly for readability or N > 64. |
