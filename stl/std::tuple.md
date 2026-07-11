# `<tuple>`

| API | Priority | Description |
|-----|----------|-------------|
| `std::tuple<Ts...>` | know | Fixed-size heterogeneous collection of N values. Access with `std::get<0>(t)`. |
| `std::make_tuple(a, b, c)` | know | Creates a tuple with deduced element types. |
| `std::tie(a, b) = t` | know | Unpack tuple into existing variables (references). Classic idiom for multi-return. |
| `std::get<I>(t)` / `std::get<T>(t)` | memorize | Access element by index or (if unique) by type. |
| `auto [a, b] = t;` | memorize | Structured bindings (C++17) — cleaner than `std::tie` for unpacking. |
| `std::tuple_size_v<T>` | know | Compile-time number of elements in a tuple type. |
| `std::tuple_cat(t1, t2)` | know | Concatenates multiple tuples into one. |
| `std::apply(fn, tuple)` | know | Calls `fn` with the tuple's elements unpacked as arguments. |
| `std::ignore` | know | Placeholder in `std::tie` to skip an element you don't care about. |
