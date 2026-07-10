# `<string_view>` (C++17)

| API | Priority | Description |
|-----|----------|-------------|
| `std::string_view` | memorize | Non-owning string reference. No allocation. Use instead of `const string&` in function parameters. |
| `.substr(pos, len)` | memorize | Returns string_view — no allocation! Different from `std::string::substr`. |
| `.starts_with` / `.ends_with` | memorize | C++20. Prefix/suffix check. |
| `.remove_prefix(n)` / `.remove_suffix(n)` | memorize | Shift the view. Instead of pointer arithmetic while parsing. |
| `.find` / `.rfind` | know | Same semantics as `std::string::find`, no allocation. |
| `.data()` / `.size()` | memorize | Raw pointer + length. Careful: `.data()` is NOT guaranteed null-terminated. |
| `operator==` (view vs string/const char*) | know | Comparison works transparently across string, string_view, and char*. |
| Dangling view | careful | A string_view into a temporary `std::string` (e.g. from a function returning by value) dangles immediately — a common source of UB. |
