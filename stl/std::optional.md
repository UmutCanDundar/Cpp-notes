# `<optional>` (C++17)

| API | Priority | Description |
|-----|----------|-------------|
| `std::optional<T>` | know | Value may or may not exist. No heap allocation — storage is inline (like a tagged union). |
| `.has_value()` / `operator bool()` | memorize | Check whether a value is present. |
| `.value()` | know | Access the value, throws `std::bad_optional_access` if empty. |
| `operator*` / `operator->` | memorize | Unchecked access — faster, but UB if empty. Use when presence is already verified. |
| `.value_or(default)` | memorize | Returns the value, or a fallback if empty — avoids manual if/else. |
| `std::nullopt` | memorize | Represents "no value," assignable to any `optional<T>`. |
| `.reset()` | know | Clears the optional back to empty state, destroying the contained value if any. |
| `.emplace(args...)` | know | Constructs the value in place, avoiding a temporary + move. |
| `optional<T&>` | avoid | Not allowed pre-C++23 — use `optional<reference_wrapper<T>>` or a pointer instead. |
