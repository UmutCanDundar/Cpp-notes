# Commonly Used Type Traits — `<type_traits>`

| Trait | Returns | Description |
|-------|---------|-------------|
| `std::is_same_v<T,U>` | bool | Are T and U the same type? |
| `std::is_integral_v<T>` | bool | Is T an integer type? (int, long, char...) |
| `std::is_floating_point_v<T>` | bool | Is T a floating point type? (float, double...) |
| `std::is_pointer_v<T>` | bool | Is T a pointer? |
| `std::is_reference_v<T>` | bool | Is T a reference? (& or &&) |
| `std::is_const_v<T>` | bool | Is T const? |
| `std::is_base_of_v<B,D>` | bool | Is B a base class of D? |
| `std::is_convertible_v<F,T>` | bool | Can F be implicitly converted to T? |
| `std::is_trivially_copyable_v<T>` | bool | Can it be copied with memcpy? — Important in HFT. |
| `std::is_standard_layout_v<T>` | bool | Is the layout C-compatible? |
| `std::remove_reference_t<T>` | type | Removes & or && from T. |
| `std::remove_const_t<T>` | type | Removes const. |
| `std::decay_t<T>` | type | Removes ref + array/function → pointer. Used before `std::forward`. |
| `std::conditional_t<B,T,F>` | type | T if B is true, F otherwise. |
| `std::enable_if_t<B,T>` | type / none | T exists if B is true, otherwise ignored via SFINAE. |
| `std::void_t<...>` (C++17) | void | Placeholder for checking multiple traits. Substitution fails if any expression is invalid. |
| `std::invoke_result_t<F,Args...>` | type | Return type of F(Args...). |
