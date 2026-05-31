# `<string_view>` `<charconv>` `<format>`

| API | Priority | Description |
|-----|----------|-------------|
| `std::string_view` | memorize | Non-owning string reference. No allocation. Use instead of `const string&` in function parameters. |
| `.substr(pos, len)` | memorize | Returns string_view — no allocation! Different from `std::string::substr`. |
| `.starts_with` / `.ends_with` | memorize | C++20. Prefix/suffix check. |
| `.remove_prefix(n)` / `.remove_suffix(n)` | memorize | Shift the view. Instead of pointer arithmetic while parsing. |
| `std::from_chars(first, last, val)` | memorize | C++17. String → number. No allocation, no exceptions. Fastest parse method. Check error with `result.ec`. |
| `std::to_chars(first, last, val)` | memorize | C++17. Number → char buffer. Faster than snprintf, no allocation. |
| `std::format("{}", val)` | careful | C++20. Python f-string style. Allocates. Great for log/debug, not on hot path. |
| `std::format_to(out, "{}", val)` | know | Write to existing buffer. Better than format but still has overhead. |
