# `std::string` Methods

| Method | Usage | Note |
|--------|-------|------|
| size / length | `s.size()` | Character count, both are the same. |
| empty | `s.empty()` | Is it empty? Returns bool. |
| at | `s.at(i)` | Bounds-checked access, throws exception. |
| operator[] | `s[i]` | No bounds check, fast. |
| front / back | `s.front()` / `s.back()` | Reference to first / last character. |
| substr | `s.substr(pos, len)` | len characters from pos. If len omitted, goes to end. Returns new string. |
| find | `s.find("abc")` / `s.find('x', pos)` | Index of first match. Returns `string::npos` if not found. |
| rfind | `s.rfind("abc")` | Searches from the end, returns last match. |
| find_first_of | `s.find_first_of("aeiou")` | Finds any one of the given characters. |
| find_last_of | `s.find_last_of("aeiou")` | From the end, any of the given characters. |
| find_first_not_of | `s.find_first_not_of(" \t")` | First character NOT in the set — useful for trimming. |
| contains (C++20) | `s.contains("abc")` | Returns bool, shorter than `find != npos`. |
| starts_with (C++20) | `s.starts_with("http")` | Returns bool. |
| ends_with (C++20) | `s.ends_with(".txt")` | Returns bool. |
| append | `s.append("xyz")` / `s.append(3, 'x')` | Append to end. Same as `+=` but has repeat option. |
| insert | `s.insert(pos, "abc")` | Insert at position. |
| erase | `s.erase(pos, len)` | Delete len characters from pos. |
| replace | `s.replace(pos, len, "new")` | Replace len characters at pos with "new". |
| clear | `s.clear()` | Empty the string. |
| resize | `s.resize(n)` / `s.resize(n, 'x')` | Adjust size. If growing, fills with space or given char. |
| reserve | `s.reserve(n)` | Prevent reallocation. |
| c_str | `s.c_str()` | Returns null-terminated `const char*`. Use when passing to C APIs. |
| data | `s.data()` | Returns non-const `char*` in C++17. |
| compare | `s.compare(other)` | 0=equal, <0=less, >0=greater. Also supports substring comparison. |
| copy | `s.copy(buf, len, pos)` | Copy to char* buffer. |

> `string::npos` = max value of size_t (~18 quintillion). After find, always check with `if (pos != string::npos)`.
