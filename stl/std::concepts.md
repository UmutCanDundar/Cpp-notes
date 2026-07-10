# `<concepts>` (C++20)

| API | Priority | Description |
|-----|----------|-------------|
| `std::integral<T>` / `std::floating_point<T>` | know | Readable constraints instead of `enable_if`. Use directly on `auto` parameters. |
| `std::same_as<T, U>` | know | Constraint form of `is_same_v`. |
| `std::convertible_to<From, To>` | know | Checks implicit convertibility as a constraint. |
| `std::constructible_from<T,Args...>` | know | Can T be constructed from Args? Template constraint. |
| `std::default_initializable<T>` | know | Constraint checking T has a usable default constructor. |
| `std::copyable<T>` / `std::movable<T>` | know | Constraint checking copy/move semantics are available. |
| `std::invocable<F, Args...>` | know | Checks F can be called with Args — used to constrain callback template parameters. |
| `std::totally_ordered<T>` | know | Checks all six comparison operators are valid for T (often satisfied via `<=>`). |
| `std::equality_comparable<T>` | know | Checks `==`/`!=` are valid for T. |
