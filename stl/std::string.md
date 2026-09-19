# `std::string` Methods

### Constructors

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| *(constructor)* | default | `string s;` | O(1) | Constructs an empty string. |
| *(constructor)* | fill | `string s(count, ch)` | O(count) | Constructs a string with `count` copies of character `ch`. |
| *(constructor)* | substring | `string s(other, pos, count = npos)` | O(count) | Constructs from a substring of `other`, starting at `pos`, `count` characters (or to the end if omitted). |
| *(constructor)* | from sized C-string | `string s(cstr, count)` | O(count) | Constructs from the first `count` characters of `cstr`; `cstr` need not be null-terminated. |
| *(constructor)* | from C-string | `string s(cstr)` | O(n) | Constructs from a null-terminated C-string; `n = strlen(cstr)`. |
| *(constructor)* | iterator range | `string s(first, last)` | O(distance(first, last)) | Constructs from a range of characters given by a pair of iterators. |
| *(constructor)* | copy | `string s(other)` | O(n) | Copy constructor — deep-copies `other`'s contents. |
| *(constructor)* | move | `string s(std::move(other))` | O(1) | Move constructor — steals `other`'s buffer; `other` is left valid but unspecified (usually empty). |
| *(constructor)* | initializer list | `string s({'a','b','c'})` | O(n) | Constructs from a brace-enclosed list of `char`. |
| *(constructor)* | from string_view-like (C++17) | `string s(t, pos, count)` | O(count) | Constructs from a substring of any type convertible to `string_view`. |
| *(constructor)* | from string_view-like, explicit (C++17) | `string s(t)` | O(n) | Explicit construction from any type convertible to `string_view` (e.g. `std::string_view`). |

> `n` here = the resulting string's length. Constructors that copy `count`/`n` characters are linear in that count because each character must be copied into the new buffer.

### Methods

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| `size_t` | size / length | `s.size()` | O(1) | Character count, both are the same. |
| `bool` | empty | `s.empty()` | O(1) | Is it empty? |
| `size_t` | max_size | `s.max_size()` | O(1) | Theoretical max characters the string could hold. |
| `size_t` | capacity | `s.capacity()` | O(1) | Current allocated storage size (≥ size()). |
| `char&` / `const char&` | at | `s.at(i)` | O(1) | Bounds-checked access, throws `out_of_range`. |
| `char&` / `const char&` | operator[] | `s[i]` | O(1) | No bounds check, fast. |
| `char&` | front / back | `s.front()` / `s.back()` | O(1) | Reference to first / last character. Undefined if empty. |
| `string` | substr | `s.substr(pos, len)` | O(len) | Copies `len` characters from `pos`. If `len` omitted, goes to end. |
| `size_t` | find | `s.find("abc")` / `s.find('x', pos)` | O(n*m) worst case | Index of first match. Returns `string::npos` if not found. |
| `size_t` | rfind | `s.rfind("abc")` | O(n*m) worst case | Searches from the end, returns index of last match. |
| `size_t` | find_first_of | `s.find_first_of("aeiou")` | O(n*k) | Finds first index of any one of the given characters. |
| `size_t` | find_last_of | `s.find_last_of("aeiou")` | O(n*k) | From the end, index of any of the given characters. |
| `size_t` | find_first_not_of | `s.find_first_not_of(" \t")` | O(n*k) | First index of a character NOT in the set — useful for trimming. |
| `size_t` | find_last_not_of | `s.find_last_not_of(" \t")` | O(n*k) | Last index of a character NOT in the set — useful for trimming trailing whitespace. |
| `bool` | contains (C++20) | `s.contains("abc")` | O(n*m) | Shorter than `find != npos`. |
| `bool` | starts_with (C++20) | `s.starts_with("http")` | O(k), k = prefix length | Checks prefix. |
| `bool` | ends_with (C++20) | `s.ends_with(".txt")` | O(k), k = suffix length | Checks suffix. |
| `string&` | append | `s.append("xyz")` / `s.append(3, 'x')` | O(k) amortized, k = length appended | Same as `+=` but has a repeat-count overload. |
| `string&` | operator+= | `s += "xyz"` / `s += 'x'` | O(k) amortized | Same as `append`, more common in practice. |
| `string&` | insert | `s.insert(pos, "abc")` | O(n) | Insert at position; shifts everything after `pos`. |
| `string&` | erase | `s.erase(pos, len)` | O(n) | Delete `len` characters from `pos`; shifts remaining characters left. |
| `string&` | replace | `s.replace(pos, len, "new")` | O(n) | Replace `len` characters at `pos` with `"new"`. |
| `void` | clear | `s.clear()` | O(1) (chars have no destructor) | Empty the string; capacity is typically retained. |
| `void` | resize | `s.resize(n)` / `s.resize(n, 'x')` | O(n) | Adjust size. If growing, fills new slots with `'\0'` or the given char. |
| `void` | reserve | `s.reserve(n)` | O(n) if reallocation happens, else O(1) | Pre-allocate capacity to avoid future reallocations. |
| `void` | shrink_to_fit | `s.shrink_to_fit()` | O(n) | Non-binding request to release unused capacity. |
| `const char*` | c_str | `s.c_str()` | O(1) | Null-terminated C string. Use when passing to C APIs. |
| `const char*` (C++11) / `char*` (C++17+) | data | `s.data()` | O(1) | Raw character buffer; null-terminated since C++11. |
| `int` | compare | `s.compare(other)` | O(min(n, m)) | `0` = equal, `<0` = less, `>0` = greater. Also supports substring comparison overloads. |
| `size_t` (chars copied) | copy | `s.copy(buf, len, pos)` | O(len) | Copies into a raw `char*` buffer. **Does not** null-terminate. |
| `void` | push_back | `s.push_back('x')` | O(1) amortized | Append a single character to the end. |
| `void` | pop_back | `s.pop_back()` | O(1) | Remove the last character. Undefined if empty. |
| `string&` | assign | `s.assign("new")` / `s.assign(5, 'x')` | O(n) | Replace the entire contents of the string. |
| `void` | swap | `s.swap(other)` | O(1) | Swaps contents with another string (pointer/size swap, no copy). |
| `iterator` | begin / end | `s.begin()` / `s.end()` | O(1) | Iterators for range-based loops / `<algorithm>` functions. |
| `reverse_iterator` | rbegin / rend | `s.rbegin()` / `s.rend()` | O(1) | Reverse iterators — handy for reading/processing a string backwards. |

> `string::npos` = max value of `size_t` (~18 quintillion). After `find`/`rfind`, always check with `if (pos != string::npos)`.

> For `find`-family functions, `n` = length of the string being searched, `m` = length of the search pattern, `k` = size of the character set argument (e.g. `"aeiou"` → k = 5). Complexity is worst-case; typical real-world implementations are close to linear.
