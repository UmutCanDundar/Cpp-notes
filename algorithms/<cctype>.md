# `<cctype>` — Character Classification & Conversion

All functions take an `int` (the char, possibly cast to `unsigned char`), return `int`.
Locale-dependent unless noted. For pure ASCII, behavior is fixed.

## Classification — return non-zero (true) or 0 (false)

| Function | Checks | Example |
|----------|--------|---------|
| `isalpha(c)` | a-z, A-Z | `isalpha('a')` → true |
| `isdigit(c)` | 0-9 | `isdigit('5')` → true |
| `isalnum(c)` | alpha or digit | `isalnum('_')` → false |
| `isupper(c)` | A-Z | `isupper('A')` → true |
| `islower(c)` | a-z | `islower('a')` → true |
| `isspace(c)` | space, \t, \n, \v, \f, \r | `isspace(' ')` → true |
| `ispunct(c)` | punctuation (not alnum, not space, printable) | `ispunct('!')` → true |
| `iscntrl(c)` | control characters (0-31, 127) | `iscntrl('\n')` → true |
| `isprint(c)` | printable, including space | `isprint(' ')` → true |
| `isgraph(c)` | printable, excluding space | `isgraph(' ')` → false |
| `isxdigit(c)` | 0-9, a-f, A-F | `isxdigit('F')` → true |
| `isblank(c)` | space or \t (C++11) | `isblank('\t')` → true |

## Conversion

| Function | Does | Example |
|----------|------|---------|
| `toupper(c)` | lowercase → uppercase, else unchanged | `toupper('a')` → `'A'` |
| `tolower(c)` | uppercase → lowercase, else unchanged | `tolower('A')` → `'a'` |

---

## Usage notes

```cpp
#include <cctype>

char c = 'A';

if (std::isalpha(c)) { ... }
char lower = std::tolower(c);   // 'a'
```

### ⚠️ Signed char trap

```cpp
char c = -1;  // some platforms: char is signed, e.g. extended ASCII byte 0xFF

std::isalpha(c);  // UB — negative value, not EOF, not unsigned char

// fix:
std::isalpha(static_cast<unsigned char>(c));
```

`int` parameter must be either `EOF` or representable as `unsigned char`. Passing a negative `signed char` (other than `EOF` = -1) is undefined behavior — common bug with non-ASCII bytes.

---

## C++ equivalents — `std::isalpha` etc. in `<locale>`

| C | C++ (`<locale>`) | Notes |
|---|------------------|-------|
| `std::isalpha(c)` | `std::isalpha(c, loc)` | Locale-aware version, takes `std::locale`. |
| `std::toupper(c)` | `std::toupper(c, loc)` | Same — locale parameter. |

```cpp
#include <locale>

std::locale loc;
bool b = std::isalpha('a', loc);  // locale-aware
```

For ASCII-only, fast code, `<cctype>` versions are simpler and avoid locale overhead.

---

## Quick reference table — ASCII ranges

```
'0'-'9'  → 48-57   → isdigit
'A'-'Z'  → 65-90   → isupper, isalpha
'a'-'z'  → 97-122  → islower, isalpha
' '      → 32      → isspace, isprint (not isgraph)
'\t'     → 9       → isspace, iscntrl
'\n'     → 10      → isspace, iscntrl
```

---

## Performance note

```cpp
// branch-heavy
if (c >= 'a' && c <= 'z') ...

// table lookup — branchless, same idea as the exec_type lookup table
constexpr auto is_alpha_table = [] {
    std::array<bool, 256> t{};
    for (int c = 'a'; c <= 'z'; c++) t[c] = true;
    for (int c = 'A'; c <= 'Z'; c++) t[c] = true;
    return t;
}();

if (is_alpha_table[static_cast<unsigned char>(c)]) ...
```

`std::isalpha` etc. are typically already implemented as a 256-entry lookup table internally — usually no need to roll your own unless avoiding locale/library call overhead in a hot loop.
