# `<optional>` (C++17)

### Notation
* `T`: the wrapped value type. `opt`, `opt1`, `opt2`: instances of `optional<T>`.
* `args...`: constructor arguments forwarded to build a `T` in place.

---

### Constructors

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| *(constructor)* | default / nullopt | `optional<T> opt;`<br>`optional<T> opt(std::nullopt)` | O(1) | Constructs an empty (disengaged) optional. Both forms are equivalent. |
| *(constructor)* | from value | `optional<T> opt(value)` | O(size of T) | Constructs engaged, copying/moving `value` in. |
| *(constructor)* | in_place | `optional<T> opt(std::in_place, args...)` | O(1) + cost of `T`'s constructor | Constructs the contained `T` directly in place from `args...`, avoiding a temporary + move. |
| *(constructor)* | copy / move | `optional<T> opt(other)` | O(size of T) / O(1) (if `T`'s move is O(1)) | Copies (if engaged) · moves (leaves `other` engaged but moved-from, **not** disengaged). |

---

### Methods

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| `bool` | has_value() / operator bool() | `opt.has_value()`<br>`if (opt) ...` | O(1) | Check whether a value is present. Equivalent forms. |
| `T&` | value() | `opt.value()` | O(1) | Access the value; throws `std::bad_optional_access` if empty. |
| `T&` | operator* / operator-> | `*opt`<br>`opt->member` | O(1) | Unchecked access — faster, but **UB if empty**. Use only when presence is already verified. |
| `T` | value_or(default) | `opt.value_or(default)` | O(1) + cost of copy | Returns the value if present, or a fallback if empty — avoids manual `if`/`else`. |
| `void` | reset() | `opt.reset()` | O(1) + `T`'s destructor | Clears the optional back to the empty state, destroying the contained value if any. |
| `T&` | emplace(args...) | `opt.emplace(args...)` | O(1) + cost of `T`'s constructor | Destroys any existing value, then constructs a new `T` in place from `args...` — avoids a temporary + move. |
| `void` | swap(other) | `opt1.swap(opt2)` | O(1) or O(size of T), depending on engaged states | Exchanges contents (and engaged/disengaged state) with another optional. |
| `bool` | operator== / operator!= / operator\< (etc.) | `opt1 == opt2` | O(1) + `T`'s comparison | Two empty optionals compare equal; empty compares less than any engaged value; otherwise compares the contained values. |
| — | nullopt | `std::nullopt` | N/A | A constant representing "no value," assignable to any `optional<T>`. |
| `optional<T>` | make_optional | `std::make_optional(value)`<br>`std::make_optional<T>(args...)` | O(size of T) | Convenience factory — deduces `T`, or forwards `args...` to construct `T` in place. |
| — | `optional<T&>` | *(not allowed pre-C++23)* | N/A | References aren't supported as `T` before C++23 — use `optional<reference_wrapper<T>>` or a raw pointer instead. |

> **No heap allocation.** Storage for the contained value is inline inside the `optional` object itself (like a tagged union) — `sizeof(optional<T>)` is roughly `sizeof(T) + 1` (plus padding), not a pointer.
