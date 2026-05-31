# `<functional>` `<type_traits>` `<concepts>`

| API | Priority | Description |
|-----|----------|-------------|
| `std::function<R(Args)>` | avoid | Type erasure + heap allocation + virtual call. Never on hot path. Use templates or raw function pointers. |
| Lambda + auto parameter | memorize | Template lambda instead of std::function. Zero overhead, gets inlined. |
| `std::invoke(fn, args...)` | know | Uniformly call any callable. Works for functions, lambdas, member pointers. |
| `std::bind` | avoid | Made redundant by lambdas. Has overhead. |
| `std::less<>` / `greater<>` | know | Comparators for sort and priority_queue. Transparent versions (empty template) allow heterogeneous lookup. |
| `std::hash<T>` | know | Hash for unordered_map. Specialize for custom types. |
| `std::is_integral_v<T>` | memorize | `<type_traits>`. Compile-time type check. All `is_X_v` forms. |
| `std::decay_t<T>` | memorize | Remove ref and const, array→pointer. Used in perfect forwarding templates. |
| `std::enable_if_t<B,T>` | know | SFINAE. Replaced by `requires` in C++20. |
| `std::integral` / `std::floating_point` | know | C++20 concepts. Readable constraints instead of enable_if. |
| `std::constructible_from<T,Args...>` | know | Can T be constructed from Args? Template constraint. |
