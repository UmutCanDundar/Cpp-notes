# Template Parameter Types — What Can Be Written Inside `template<...>`

| Type | Syntax | Description |
|------|--------|-------------|
| Type parameter | `template<typename T>` / `template<class T>` | Most common. T can be any type. `typename` and `class` are equivalent here. |
| Default type | `template<typename T = int>` | Uses int if T is not provided. Works just like a function default argument. |
| Non-type (value) | `template<int N>` / `template<size_t N>` / `template<bool B>` | Compile-time constant value. This is how `array<int,4>` works — 4 is a non-type parameter. |
| Default value | `template<int N = 64>` | Uses 64 if N is not provided. |
| Template template | `template<template<typename> class C>` | Takes a template as a parameter. E.g. C = vector can be passed, used as C<int> inside. |
| Variadic type | `template<typename... Ts>` | Zero or more types. `sizeof...(Ts)` gives the count. `Ts...` for pack expansion. |
| Variadic non-type | `template<auto... Vs>` (C++17) | Mixed-type compile-time values. `auto` works for any non-type. |
| Concept constraint | `template<std::integral T>` (C++20) | Requires T to be an integer type. Readable replacement for enable_if. |
| Requires clause | `template<typename T> requires std::is_integral_v<T>` (C++20) | For more complex conditions. Multiple conditions with `&&`. |
| `= void` pattern | `template<typename T, typename = void>` | Second parameter is a SFINAE placeholder. Caller doesn't provide it, compiler fills it in. Used with enable_if. |

> **Common pattern:** `template<typename T, typename = std::enable_if_t<std::is_integral_v<T>>>` — If T is an integer, this function/struct exists; otherwise it is ignored (SFINAE).
