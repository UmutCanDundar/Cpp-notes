# C++ Alternatives — When to Use Which

| C Function | C++ Alternative | When to Use the C Version |
|------------|-----------------|---------------------------|
| memcpy | `std::copy`, `std::copy_n` | For arrays of trivially copyable structs, memcpy is still preferred — compiler converts to SIMD. |
| memmove | `std::copy` (overlapping range) | memmove is more explicit for overlapping C arrays. |
| memset | `std::fill`, `std::fill_n` | `memset(p,0,n)` is still common for zeroing. Use fill for non-zero fills. |
| memcmp | `std::equal`, `operator==` | memcmp is fast for POD struct comparison, but compares padding bytes — be careful. |
| strcpy/strcat | `std::string` assignment/`+=` | Almost never. Only when writing to a C API. |
| strcmp | `std::string::compare`, `operator==` | strcmp is still used for const char* comparisons. |
| sprintf/snprintf | `std::format` (C++20), `ostringstream` | snprintf is still preferred in HFT — no allocation, fast. |
| strtol/strtod | `std::from_chars` (C++17) | from_chars is faster and has no exceptions. Prefer in new code. |

## HFT Special: Fastest Copy Selection

| Situation | Preference |
|-----------|------------|
| Small struct (≤64 bytes) | `=` operator — compiler loads into registers |
| Large trivially copyable array | `memcpy` — SIMD vectorization |
| Non-trivial objects | `std::copy` — constructor/destructor called |
| Overlapping regions | `memmove` |
| Zeroing (POD array) | `memset(p, 0, sizeof(arr))` |
