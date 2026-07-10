# `<functional>`

| API | Priority | Description |
|-----|----------|-------------|
| `std::function<R(Args)>` | avoid | Type erasure + heap allocation + virtual call. Never on hot path. Use templates or raw function pointers. |
| Lambda + auto parameter | memorize | Template lambda instead of std::function. Zero overhead, gets inlined. |
| `std::invoke(fn, args...)` | know | Uniformly call any callable. Works for functions, lambdas, member pointers. |
| `std::bind` | avoid | Made redundant by lambdas. Has overhead, hard to read. |
| `std::bind_front` | know | C++20, simpler than `std::bind` for partial application, but a lambda is usually still clearer/faster. |
| `std::less<>` / `std::greater<>` | know | Comparators for sort and priority_queue. Transparent versions (empty template) allow heterogeneous lookup. |
| `std::hash<T>` | know | Hash for unordered_map. Specialize for custom types. |
| `std::reference_wrapper<T>` / `std::ref` / `std::cref` | know | Makes a reference copyable/assignable, e.g. to store references in a container. |
| `std::mem_fn` | avoid | Wraps a member function pointer as a callable. Superseded by lambdas in almost all cases. |
| `std::not_fn` | know | Negates a predicate. Rarely needed once you can just write `!pred(x)` in a lambda. |
