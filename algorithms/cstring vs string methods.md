# str* — Null-Terminated C String Functions `<cstring>` vs std::string — C++ String Functions `<string>`

| C string | std::string equivalent | Signature | Perf & Notes |
|----------|----------------------|-----------|-------|
| `strlen(s)` | `.size()` / `.length()` | `s.size()` | ✅ O(1) — std::string caches size. strlen is O(n) scan. Always prefer `.size()` in hot path. |
| `strcpy(dst, src)` | `= operator` | `s = other` | ✅ Deep copy, safe. Use `s = std::move(other)` if src no longer needed — avoids allocation. |
| `strncpy(dst, src, n)` | `.substr()` / `.assign()` | `s.substr(0, n)` | ⚠️ `substr` allocates new string. For in-place use `.assign(other, 0, n)` to avoid extra allocation. |
| `strcat(dst, src)` | `+=` / `.append()` | `s += other` | ✅ No O(n²) trap. std::string manages capacity with doubling strategy. `+=` preferred over `append` for readability. |
| `strncat(dst, src, n)` | `.append(s, pos, n)` | `s.append(other, 0, n)` | ✅ Appends first n chars. Safe, no buffer overflow. |
| `strcmp(a, b)` | `==` / `<` / `<=>` | `a == b` | ✅ `==` short-circuits on size mismatch — fast. For sorting use `<` or `<=>` (C++20 spaceship). |
| `strncmp(a, b, n)` | `.compare(0, n, other)` | `a.compare(0, n, b)` | ⚠️ Returns int like strcmp. Less readable than `==` but needed for partial comparison. |
| `strchr(s, c)` | `.find(c)` | `s.find('c')` | ⚠️ Returns `size_t` index, not pointer. Check against `std::string::npos`. Linear scan. |
| `strrchr(s, c)` | `.rfind(c)` | `s.rfind('c')` | ⚠️ Scans full string from end. More expensive than `find`. Returns `npos` if not found. |
| `strstr(haystack, needle)` | `.find(str)` | `s.find("needle")` | ⚠️ Naive O(n*m). No guaranteed Boyer-Moore. Avoid on large strings in hot path. |
| `strtok(s, delim)` | `std::stringstream` / `ranges::split` | `getline(ss, token, delim)` | ❌ strtok has static state, thread-unsafe. stringstream is cleaner but allocates. ranges::split (C++20) is lazy, no allocation. |
| `strtol(s, &end, base)` | `std::stoi` / `std::stol` / `std::from_chars` | `std::stoi(s)` | ⚠️ stoi throws on error. strtol gives end pointer. `std::from_chars` (C++17) is fastest — no allocation, no exception, no locale. |
| `strtod(s, &end)` | `std::stod` / `std::from_chars` | `std::stod(s)` | ⚠️ Same as above. `from_chars` for float is C++17 but some compilers were late to implement. |
| `sprintf(buf, fmt)` | `std::format` (C++20) | `std::format("{}", x)` | ✅ Type-safe, no buffer overflow. Slightly slower than snprintf in some benchmarks but negligible. |
| `snprintf(buf, n, fmt)` | `std::format` / `std::to_string` | `std::format("{}", x)` | ✅ snprintf still valid in perf-critical C-interop code. format is cleaner for general use. |
| — | `std::to_string(x)` | `std::to_string(42)` | ❌ Allocates. Locale-dependent in some implementations. Avoid in hot path. Use `std::format` or `from_chars` instead. |
| — | `std::from_chars` | `from_chars(p, end, val)` | ✅✅ Fastest string→number. No allocation, no exception, no locale. Hot path safe. C++17. |
| — | `std::to_chars` | `to_chars(p, end, val)` | ✅✅ Fastest number→string. Writes to existing buffer, no allocation. Hot path safe. C++17. |
| — | `.reserve(n)` | `s.reserve(100)` | ✅ Pre-allocate capacity. Use when final size is known — eliminates reallocation in append loops. |
| — | `.empty()` | `s.empty()` | ✅ O(1). Never use `s.size() == 0` — empty() is more explicit and just as fast. |
| — | `.clear()` | `s.clear()` | ✅ Sets size to 0, keeps capacity. Good for reusing buffer without reallocating. |
| — | `.c_str()` | `s.c_str()` | ⚠️ Returns null-terminated const char*. Valid until string is modified. Use only for C API interop. |
| — | `.data()` | `s.data()` | ⚠️ Same as c_str() since C++11 for std::string. For std::string_view, not null-terminated — careful with C APIs. |
| — | `std::string_view` | `std::string_view sv = s` | ✅✅ No allocation, no copy. Just a pointer+size. Use for read-only string params instead of const std::string&. Hot path friendly. |

---

**Key takeaways:**

**Hot path safe:**
```cpp
from_chars / to_chars   // number conversion, zero allocation
string_view             // read-only access, zero allocation
.size()                 // O(1) vs strlen O(n)
.reserve()              // eliminate reallocation in loops
.clear()                // reuse buffer
```

**Avoid in hot path:**
```cpp
std::to_string()        // allocates, locale-dependent
.substr()               // always allocates new string
.find() on large string // naive O(n*m) for substring
strcat in loop          // O(n²) trap
```

**C interop:**
```cpp
s.c_str()    // null-terminated, read-only, invalidated on modification
s.data()     // same as c_str() for std::string, NOT null-terminated for string_view
```
