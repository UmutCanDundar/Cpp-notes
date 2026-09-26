# `<string_view>` (C++17)

### Notation
* `sv`, `sv1`, `sv2`: instances of `string_view`. `i`: a character index. `pos`, `len`, `n`: index/length/count arguments.
* A `string_view` never owns its data — it's just a `(pointer, length)` pair into someone else's buffer.

---

### Constructors

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| *(constructor)* | default | `string_view sv;` | O(1) | Empty view — `data() == nullptr`, `size() == 0`. |
| *(constructor)* | from C-string | `string_view sv(cstr)` | O(n) (calls `strlen`) | Views a null-terminated C-string; length is computed once at construction. |
| *(constructor)* | from sized buffer | `string_view sv(cstr, len)` | O(1) | Views exactly `len` characters starting at `cstr` — does **not** call `strlen`, and the buffer need not be null-terminated. |
| *(constructor)* | from string | `string_view sv(str)` | O(1) | Implicitly views an existing `std::string`'s buffer (no copy, no allocation). |
| *(constructor)* | copy | `string_view sv(other)` | O(1) | Trivial copy — just copies the pointer and length. |

---

### Methods

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| `char` | operator[] | `sv[i]` | O(1) | No bounds check. |
| `char` | at(i) | `sv.at(i)` | O(1) | Bounds-checked, throws `out_of_range`. |
| `char` | front() / back() | `sv.front()` / `sv.back()` | O(1) | First / last character. Undefined if empty. |
| `const char*` | data() | `sv.data()` | O(1) | Raw pointer + `size()` describe the view. **Not** guaranteed null-terminated — never pass `.data()` alone to a C API expecting a C-string. |
| `size_t` | size() / length() | `sv.size()` | O(1) | Character count. |
| `bool` | empty() | `sv.empty()` | O(1) | Whether the view has zero length. |
| `string_view` | substr(pos, len) | `sv.substr(pos, len)` | O(1) | Returns a **new view** into the same buffer — no allocation, unlike `std::string::substr` which copies. |
| `bool` | starts_with / ends_with (C++20) | `sv.starts_with("http")`<br>`sv.ends_with(".txt")` | O(k), k = prefix/suffix length | Prefix / suffix check. |
| `void` | remove_prefix(n) / remove_suffix(n) | `sv.remove_prefix(n)`<br>`sv.remove_suffix(n)` | O(1) | Shifts the view's start forward / end backward by `n` characters — replaces manual pointer arithmetic while parsing. |
| `size_t` | find / rfind | `sv.find("abc")`<br>`sv.rfind("abc")` | O(n*m) worst case | Same semantics as `std::string::find`/`rfind`, no allocation. Returns `string_view::npos` if not found. |
| `size_t` | find_first_of / find_last_of / find_first_not_of / find_last_not_of | `sv.find_first_of("aeiou")` (etc.) | O(n*k) | Same semantics as the corresponding `std::string` members. |
| `bool` | contains (C++23) | `sv.contains("abc")` | O(n*m) | Shorter than `find != npos`. |
| `int` | compare | `sv1.compare(sv2)` | O(min(n, m)) | `0` = equal, `<0` = less, `>0` = greater. |
| `size_t` | copy(buf, len, pos) | `sv.copy(buf, len, pos)` | O(len) | Copies into a raw `char*` buffer. Does **not** null-terminate. |
| `void` | swap(other) | `sv1.swap(sv2)` | O(1) | Swaps the pointer/length pair with another view. |
| `iterator` | begin() / end() | `sv.begin()` / `sv.end()` | O(1) | Iterators for range-based loops / `<algorithm>` functions. |
| `bool` | operator== (view vs string/const char*) | `sv == "abc"` | O(n) | Comparison works transparently across `string`, `string_view`, and `const char*` — no allocation needed on either side. |

> **Dangling view.** A `string_view` into a temporary `std::string` (e.g. one returned by value from a function) dangles the instant that temporary is destroyed — a common and easy-to-miss source of UB. Never store a `string_view` that outlives the object it was constructed from.
