# Forward Declaration and Specialization

| Concept | Example | Description |
|---------|---------|-------------|
| Forward declaration | `template<typename T> struct Foo;` | Just declares existence, no body. Used in header files to break circular dependencies. Pointer/reference is enough — full definition not needed. |
| Full specialization | `template<> struct Foo<int> { ... };` | Empty `template<>` means: "all parameters are fixed". Completely different implementation for int. |
| Partial specialization | `template<typename T> struct Foo<T*> { ... };` | Special behavior only for pointer types. Some parameters remain free. |
| Function template full spec. | `template<> void bar<double>() { ... }` | Functions cannot be partially specialized, only fully. Use overloads for partial behavior. |
| if constexpr (C++17) | `if constexpr (std::is_pointer_v<T>) { return *val; } else { return val; }` | Compile-time branching. The untaken branch is never compiled. Replaces most old specialization patterns. |
| Explicit instantiation | `template class Foo<int>;` | Forces compilation for that type in a .cpp file. Used in non-header-only template libraries. |
| Extern template | `extern template class Foo<int>;` | Do not instantiate in this translation unit, it exists elsewhere. Reduces compile time. |

> **Pattern:** First write the general `template<typename T> struct X { ... };`, then add special cases with `template<> struct X<int> { ... };`. The compiler selects the most specific match.
