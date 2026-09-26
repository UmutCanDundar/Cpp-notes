# `<format>` (C++20)

### Notation
* `fmt`: a format string literal, e.g. `"{}"`, `"{:.2f}"`, `"{0} {1}"`.
* `args...`: the values to substitute into `fmt`'s placeholders.
* `out`: an output iterator (e.g. from `back_inserter`) to write formatted characters to.
* No user-facing classes with public constructors here — `formatter<T>` specialization is a customization point for custom types, not something you construct directly.

---

### Methods

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| `string` | format | `std::format("{}", val)` | O(n) | Python f-string style. Allocates and returns a new `std::string`. Great for log/debug, not on the hot path. |
| `OutputIt` | format_to | `std::format_to(out, "{}", val)` | O(n) | Writes to an existing buffer/output iterator instead of allocating a new string. Still has formatting overhead. |
| `format_to_n_result<OutputIt>` | format_to_n | `std::format_to_n(out, n, "{}", val)` | O(min(result_size, n)) | Bounded write, stops at `n` characters — safe for fixed-size buffers. Result struct also reports how many characters *would* have been written. |
| `size_t` | formatted_size | `std::formatted_size("{}", val)` | O(n) | Computes the output size without writing, useful for pre-sizing a buffer before calling `format_to`. |
| `string` | vformat | `std::vformat(fmt, std::make_format_args(args...))` | O(n) | Type-erased version of `format`, taking a runtime (non-literal) format string plus a pre-packed argument list. Used when `fmt` isn't known at compile time. |
| `OutputIt` | vformat_to | `std::vformat_to(out, fmt, std::make_format_args(args...))` | O(n) | Type-erased version of `format_to`, same relationship as `vformat` to `format`. |
| `void` | print (C++23) | `std::print("{}\n", val)` | O(n) | Formats and writes directly to a stream (`stdout` by default) in one call — skips the intermediate `std::string`. |
| `void` | println (C++23) | `std::println("{}", val)` | O(n) | Same as `print`, with a trailing newline appended automatically. |

> **Compile-time format string checking.** Malformed `"{}"` placeholders (wrong count, bad type for the spec) are a **compile error** when `fmt` is a string literal — unlike `printf`'s runtime undefined behavior on a mismatched format specifier.
