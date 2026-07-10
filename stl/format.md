# `<format>` (C++20)

| API | Priority | Description |
|-----|----------|-------------|
| `std::format("{}", val)` | careful | Python f-string style. Allocates a std::string. Great for log/debug, not on hot path. |
| `std::format_to(out, "{}", val)` | know | Write to existing buffer/iterator. Better than format but still has formatting overhead. |
| `std::format_to_n(out, n, "{}", val)` | know | Bounded write, stops at n characters — safe for fixed-size buffers. |
| `std::formatted_size("{}", val)` | know | Computes the output size without writing, useful for pre-sizing a buffer. |
| `std::print("{}\n", val)` | know | C++23. Formats and writes directly to a stream in one call. |
| Compile-time format string checking | know | Malformed `"{}"` placeholders are a compile error, unlike printf's runtime UB. |
