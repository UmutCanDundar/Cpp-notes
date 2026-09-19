# `<bitset>`

### Notation
* `N`: compile-time constant — the number of bits, fixed at compile time (template parameter).
* `i`, `pos`: bit index, `0` to `N-1`.
* `b`, `b1`, `b2`: instances of `bitset<N>` (both operands of a binary op must have the same `N`).
* `pos`, `n`: (string constructors only) starting position in the source string, and number of characters to read from it.
* `zero`, `one`: (string constructors only) which characters represent bit `0` and bit `1` — default `'0'`/`'1'`, but any two distinct characters work.

---

| Return Type | Function Signature | Complexity | Description |
| :--- | :--- | :--- | :--- |
| *(class template)* | `bitset<N>` | N/A | Fixed-size sequence of bits, stored compactly (`N/8` bytes). Not resizable, unlike `vector<bool>`. |
| *(constructor)* | `bitset<N>()` | $O(N)$ | Default-constructs all bits to `0`. |
| *(constructor)* | `bitset<N>(unsigned long long val)` | $O(N)$ | Constructs from the low `N` bits of `val`. |
| *(constructor)* | `bitset<N>(const string& str, pos = 0, n = npos, zero = '0', one = '1')` | $O(N)$ | Constructs from a `std::string`, reading `n` characters starting at `pos`. `zero`/`one` let you use custom characters instead of literal `'0'`/`'1'` (e.g. `'A'`/`'B'`). |
| *(constructor)* | `bitset<N>(const CharT* str, n = npos, zero = '0', one = '1')` | $O(N)$ | Same as above, but constructs from a C-string (`const char*`) instead of `std::string`. |
| `bool` | `operator[](i)` | $O(1)$ | Unchecked read/write access to bit `i` (via a proxy reference). No bounds check. |
| `bool` | `test(i)` | $O(1)$ | Bounds-checked read of bit `i` — throws `out_of_range` if `i >= N`. |
| `bitset&` | `set()` | $O(N)$ | Sets all bits to `1`. |
| `bitset&` | `set(i, val = true)` | $O(1)$ | Sets (or, with `val = false`, clears) bit `i`. |
| `bitset&` | `reset()` | $O(N)$ | Clears all bits to `0`. |
| `bitset&` | `reset(i)` | $O(1)$ | Clears bit `i`. |
| `bitset&` | `flip()` | $O(N)$ | Toggles all bits. |
| `bitset&` | `flip(i)` | $O(1)$ | Toggles bit `i`. |
| `size_t` | `count()` | $O(N)$ | Number of set bits — similar to `std::popcount` but for the whole fixed-width set. |
| `size_t` | `size()` | $O(1)$ | Returns `N` (compile-time constant). |
| `bool` | `any()` | $O(N)$ | True if at least one bit is set. |
| `bool` | `all()` | $O(N)$ | True if every bit is set. |
| `bool` | `none()` | $O(N)$ | True if no bit is set. |
| `unsigned long` | `to_ulong()` | $O(N)$ | Converts to `unsigned long`. Throws `overflow_error` if the value doesn't fit. |
| `unsigned long long` | `to_ullong()` | $O(N)$ | Converts to `unsigned long long`. Throws `overflow_error` if the value doesn't fit. |
| `string` | `to_string()` | $O(N)$ | Converts to a string of `'0'`/`'1'` characters. |
| `bitset&` | `operator&=(b)` / `operator\|=(b)` / `operator^=(b)` | $O(N)$ | In-place bitwise AND / OR / XOR with another bitset of the same size. |
| `bitset` | `operator~()` | $O(N)$ | Returns a copy with every bit flipped. |
| `bitset&` | `operator<<=(pos)` / `operator>>=(pos)` | $O(N)$ | In-place bit shift by `pos` positions. |
| `bitset` | `operator<<(pos)` / `operator>>(pos)` | $O(N)$ | Returns a shifted copy, original unchanged. |
| `bool` | `operator==(b)` / `operator!=(b)` | $O(N)$ | Compares two bitsets of the same size. |
| `bitset` | `operator&(b1, b2)` / `operator\|(b1, b2)` / `operator^(b1, b2)` | $O(N)$ | Non-member bitwise set operations, return a new bitset. |
| `ostream&` | `operator<<(os, b)` | $O(N)$ | Stream insertion — writes the bitset as a string of `'0'`/`'1'`. |
| `istream&` | `operator>>(is, b)` | $O(N)$ | Stream extraction — reads a string of `'0'`/`'1'` into the bitset. |
| `size_t` | `hash<bitset<N>>{}(b)` | $O(N)$ | Hash specialization so `bitset` can be used as a key in unordered containers. |

> **`bitset` vs raw integer bit-flags:** for `N <= 64`, a raw `uint64_t` with manual bit ops is usually faster (no bounds checks, direct register ops). `bitset` mainly shines for readability, or when `N > 64` and you need more bits than a single machine word can hold.
