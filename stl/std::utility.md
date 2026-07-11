# `<utility>`

| API | Priority | Description |
|-----|----------|-------------|
| `std::pair<T,U>` | memorize | Two values. `.first` / `.second`. Used for map iterator pairs. |
| `std::make_pair(a, b)` | memorize | Create a pair with deduced types. |
| `std::move(x)` | memorize | Cast to rvalue. Triggers move instead of copy. Does not actually move — it casts. |
| `std::forward<T>(x)` | memorize | Perfect forwarding. Preserves lvalue/rvalue in template functions. |
| `std::exchange(obj, new_val)` | know | Returns old value, assigns new. Common in move constructors to null-out a source. |
| `std::swap(a, b)` | memorize | Exchanges two values. Prefer the member/ADL `swap` for custom types over generic 3-move swap. |
| `std::as_const(x)` | know | Returns a `const&` to x, useful to force calling a const overload. |
| `std::in_place` / `std::in_place_type_t` | know | Tag types for constructing `optional`/`variant`/`any` in place, avoiding a temporary + move. |
| `std::index_sequence<Is...>` | know | Compile-time integer sequence, used for unpacking tuples/parameter packs via template metaprogramming. |
| `std::cmp_less` / `cmp_greater` etc. | know | C++20. Safe comparison between signed and unsigned integers without UB/warnings. |
