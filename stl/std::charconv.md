# `<charconv>` (C++17)

| API | Priority | Description |
|-----|----------|-------------|
| `std::from_chars(first, last, val)` | memorize | String → number. No allocation, no exceptions, no locale. Fastest parse method. Check error with `result.ec`. |
| `std::to_chars(first, last, val)` | memorize | Number → char buffer. Faster than snprintf/std::to_string, no allocation. |
| `std::from_chars` with `fmt` (hex/scientific) | know | Optional format argument (`std::chars_format::hex`, `::scientific`) for non-decimal parsing. |
| `std::to_chars_result` / `std::from_chars_result` | know | Result structs holding `.ptr` (end position) and `.ec` (error code, compare to `std::errc{}`for success). |
