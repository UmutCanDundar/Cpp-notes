# `<type_traits>`

| API | Priority | Description |
|-----|----------|-------------|
| `std::is_integral_v<T>` | memorize | Compile-time type check. All `is_X_v` forms follow this pattern. |
| `std::is_floating_point_v<T>` | know | Compile-time check for float/double/long double. |
| `std::is_same_v<T, U>` | memorize | Compile-time type equality check. Used constantly in `if constexpr` and static_assert. |
| `std::is_pointer_v<T>` / `is_reference_v<T>` | know | Check if T is a pointer / reference type. |
| `std::is_trivially_copyable_v<T>` | memorize | Required for a type to be safe with `memcpy` and for `std::atomic<T>`. |
| `std::is_nothrow_move_constructible_v<T>` | know | Checks if move ctor is noexcept — decides whether STL containers move or copy on reallocation. |
| `std::decay_t<T>` | memorize | Remove ref and const, array→pointer. Used in perfect forwarding templates. |
| `std::remove_reference_t<T>` | memorize | Strips `&`/`&&` from a type. Needed inside `std::move`'s own implementation. |
| `std::remove_cv_t<T>` | know | Strips `const`/`volatile` from a type. |
| `std::conditional_t<B, T, F>` | know | Compile-time ternary for types: picks T if B is true, else F. |
| `std::enable_if_t<B,T>` | avoid | SFINAE. Verbose and hard to read. Replaced by `requires` clauses/concepts in C++20. |
| `std::void_t<...>` | know | SFINAE detection idiom helper — checks if an expression is well-formed. |
| `std::underlying_type_t<E>` | know | Gets the underlying integer type of an enum. |
| `std::is_base_of_v<Base, Derived>` | know | Compile-time inheritance check. |
