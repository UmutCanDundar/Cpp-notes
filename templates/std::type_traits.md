# `<type_traits>` — Type Traits (C++11 → C++26)

Type traits are **compile-time** questions and transformations about types: "is `T` an integer?", "give me `T` without `const`", "can `U` be converted to `T`?". They cost nothing at run time; the only cost is template instantiation when compiling.

### Notation (what goes inside `<>`)
* `T`, `U`: any types. `Ts...`: a pack of types. `Args...`: a pack of argument types.
* `B`: a compile-time `bool` constant. `Bs...`: a pack of traits (or `bool` constants).
* `v`: a compile-time constant value of type `T` (used by `integral_constant`).
* `N`: a compile-time `unsigned` index, e.g. which array dimension. `Len`, `Align`: `size_t` constants (size and alignment).
* `Base`, `Derived`: class types. `From`, `To`: source and destination types of a conversion or assignment. `E`: an enum type.
* `Fn`: a callable type (function, function pointer, lambda, functor, member pointer). `R`: an expected return type. `F`: the "false" type of `conditional`.
* `= default`: shown after a parameter when it has a default, e.g. `enable_if<B, T = void>`.
* **`_v` suffix** (C++17): variable template, `X_v<T>` is `X<T>::value`. **`_t` suffix** (C++14): alias template, `X_t<T>` is `typename X<T>::type`. Always prefer these shorter forms.
* **Verified**: every `→` result below was compiled and evaluated with g++ 13.3 (`-std=c++23`). Entries marked "not available in libstdc++ 13" produced a compile error there. C++26 entries come from the standard and were not compiled.

Types used in the examples:

```cpp
struct S { int a; double b; };              struct S2 { int c; double d; };
struct Empty {};                            struct Padded { char c; int i; };
struct Poly { virtual ~Poly() = default; virtual void f() {} };
struct Abs { virtual void f() = 0; };       struct Fin final {};
struct Base {}; struct Derived : Base {};   struct Priv : private Base {};
union U { int i; float f; };
enum class E : unsigned char { a };         enum UE { x };
struct NoDef { NoDef(int); };               struct Expl { explicit Expl(int); };
struct PrivDtor { private: ~PrivDtor(); };  struct Throwing { Throwing(Throwing&&) noexcept(false); };
```

---

## 1. The three shapes of a trait

| Returns | Shape | Usage | Note |
|---------|-------|-------|------|
| `bool` | **Question**: "does `T` have property X?" | `std::is_integral<T>::value`<br>`std::is_integral_v<T>` | The trait is a class that derives from `std::true_type` or `std::false_type`. Also has `::value_type` (`bool`), `::type`, an implicit conversion to `bool`, and (C++14) `operator()`. |
| type | **Transformation**: "give me `T` modified" | `typename std::remove_const<T>::type`<br>`std::remove_const_t<T>` | The result is the nested `::type`; `_t` removes the `typename ...::type` boilerplate. |
| `size_t` | **Property query**: "how big/deep/aligned?" | `std::rank_v<T>`<br>`std::extent_v<T, N = 0>`<br>`std::alignment_of_v<T>` | A number, not a `bool`. |
| varies | **Function**: not a class template | `std::is_constant_evaluated()`<br>`std::is_corresponding_member(m1, m2)` | Called like a normal `constexpr` function. |

---

## 2. Foundations

| Returns | Trait | Usage | Note |
|---------|-------|-------|------|
| class | integral_constant | `integral_constant<T, v>`<br>`integral_constant<int, 5>::value` → `5` | The base of every trait: wraps a compile-time value. Members: `value`, `value_type`, `type`, `operator value_type()`, (C++14) `operator()()`. |
| class | bool_constant (C++17) | `bool_constant<B>`<br>`bool_constant<true>::value` → `true` | Alias for `integral_constant<bool, B>`. |
| class | true_type / false_type | `true_type`, `false_type`<br>`is_same_v<true_type, bool_constant<true>>` → `true` | `bool_constant<true>` and `bool_constant<false>`. What question traits derive from; also used as *tags* to select an overload at compile time. |
| type | void_t (C++17) | `void_t<Ts...>`<br>`void_t<int>` → `void` | Always `void` if all `Ts...` are valid types, otherwise a substitution failure. The key piece of the *detection idiom* (section 14). |
| type | type_identity (C++20) | `type_identity<T>`<br>`type_identity_t<int>` → `int` | Maps `T` to itself. Its use is to put `T` in a *non-deduced context*, so a template argument is taken from other parameters only. |
| `T&&` | declval (`<utility>`) | `std::declval<T>()` | Fake value of type `T&&` for use inside `decltype`/`noexcept` only; never call it at run time. |
| `bool` | is_constant_evaluated (C++20) | `std::is_constant_evaluated()`<br>`constexpr bool b = is_constant_evaluated();` → `true`<br>`bool b = is_constant_evaluated();` evaluated at run time → `false` | Function: `true` while being evaluated at compile time. In C++23 `if consteval` is the cleaner spelling. |
| `bool` | is_within_lifetime (C++26) | `std::is_within_lifetime(p)` | Is the object `p` points to within its lifetime (e.g. which union member is active) during constant evaluation. Not available in libstdc++ 13. |

---

## 3. Primary type categories

Every type belongs to **exactly one** of these (cv-qualifiers are ignored). Use them to ask "what kind of thing is `T`?".


| Returns | Trait | Usage | Note |
|---------|-------|-------|------|
| `bool` | is_void | `is_void<T>` / `is_void_v<T>`<br>`is_void_v<void>` → `true`<br>`is_void_v<const void>` → `true`<br>`is_void_v<int>` → `false` | True for `void` (cv-qualified too). Guards code that cannot hold a `void` value, such as a function result. |
| `bool` | is_null_pointer (C++14) | `is_null_pointer<T>`<br>`is_null_pointer_v<nullptr_t>` → `true`<br>`is_pointer_v<nullptr_t>` → `false` | True only for `std::nullptr_t`. It is **not** a pointer type, so `is_pointer_v<nullptr_t>` is false. |
| `bool` | is_integral | `is_integral<T>`<br>`is_integral_v<bool>` → `true`<br>`is_integral_v<char>` → `true`<br>`is_integral_v<float>` → `false`<br>`is_integral_v<const int>` → `true`<br>`is_integral_v<int&>` → `false` | `bool`, the `char` types, `wchar_t`, `short`, `int`, `long`, `long long` and their unsigned forms. cv is ignored, references are not integral (strip with `remove_cvref_t` first). |
| `bool` | is_floating_point | `is_floating_point<T>`<br>`is_floating_point_v<long double>` → `true`<br>`is_floating_point_v<int>` → `false` | `float`, `double`, `long double`. |
| `bool` | is_array | `is_array<T>`<br>`is_array_v<int[3]>` → `true`<br>`is_array_v<int[]>` → `true`<br>`is_array_v<array<int,3>>` → `false` | C-style arrays, bounded or unbounded. `std::array` is a class, not an array. |
| `bool` | is_enum | `is_enum<T>`<br>`is_enum_v<E>` → `true`<br>`is_enum_v<UE>` → `true` | Scoped (`enum class`) and unscoped enums. |
| `bool` | is_union | `is_union<T>`<br>`is_union_v<U>` → `true`<br>`is_class_v<U>` → `false` | `union` types. |
| `bool` | is_class | `is_class<T>`<br>`is_class_v<S>` → `true`<br>`is_class_v<U>` → `false`<br>`is_class_v<E>` → `false` | `class`/`struct` types, but **not** unions and not enums. |
| `bool` | is_function | `is_function<T>`<br>`is_function_v<int(int)>` → `true`<br>`is_function_v<int(*)(int)>` → `false` | A function *type* such as `int(int)`. A function pointer, lambda or functor is not. |
| `bool` | is_pointer | `is_pointer<T>`<br>`is_pointer_v<int*>` → `true`<br>`is_pointer_v<int* const>` → `true`<br>`is_pointer_v<int S::*>` → `false`<br>`is_pointer_v<nullptr_t>` → `false` | Object and function pointers. Not member pointers, not `nullptr_t`. |
| `bool` | is_lvalue_reference | `is_lvalue_reference<T>`<br>`is_lvalue_reference_v<int&>` → `true`<br>`is_lvalue_reference_v<int&&>` → `false` | `T&`. |
| `bool` | is_rvalue_reference | `is_rvalue_reference<T>`<br>`is_rvalue_reference_v<int&&>` → `true` | `T&&`. |
| `bool` | is_member_object_pointer | `is_member_object_pointer<T>`<br>`is_member_object_pointer_v<int S::*>` → `true`<br>`is_member_object_pointer_v<void (Poly::*)()>` → `false` | Pointer to a data member (`int S::*`). |
| `bool` | is_member_function_pointer | `is_member_function_pointer<T>`<br>`is_member_function_pointer_v<void (Poly::*)()>` → `true` | Pointer to a member function. |
| `bool` | is_reflection (C++26) | `is_reflection<T>`<br>*(not compiled)* | True for the reflection value type `std::meta::info` introduced with static reflection. |


---

## 4. Composite type categories

Groups of the primary categories. Use them for broader questions.


| Returns | Trait | Usage | Note |
|---------|-------|-------|------|
| `bool` | is_fundamental | `is_fundamental<T>`<br>`is_fundamental_v<void>` → `true`<br>`is_fundamental_v<nullptr_t>` → `true`<br>`is_fundamental_v<int*>` → `false` | `void`, `nullptr_t` and arithmetic types: the built-in types. |
| `bool` | is_arithmetic | `is_arithmetic<T>`<br>`is_arithmetic_v<bool>` → `true`<br>`is_arithmetic_v<E>` → `false` | Integral or floating point (`bool` counts, enums do not). |
| `bool` | is_scalar | `is_scalar<T>`<br>`is_scalar_v<int*>` → `true`<br>`is_scalar_v<E>` → `true`<br>`is_scalar_v<S>` → `false` | Arithmetic, enum, pointer, member pointer, `nullptr_t`: types that hold a single value. |
| `bool` | is_object | `is_object<T>`<br>`is_object_v<int>` → `true`<br>`is_object_v<void>` → `false`<br>`is_object_v<int&>` → `false`<br>`is_object_v<int(int)>` → `false` | Anything that is not a function, reference or `void`: you can create an object of this type. |
| `bool` | is_compound | `is_compound<T>`<br>`is_compound_v<int*>` → `true`<br>`is_compound_v<int>` → `false` | Everything that is **not** fundamental (pointers, references, arrays, classes, functions...). |
| `bool` | is_reference | `is_reference<T>`<br>`is_reference_v<int&&>` → `true` | `T&` or `T&&`. |
| `bool` | is_member_pointer | `is_member_pointer<T>`<br>`is_member_pointer_v<int S::*>` → `true` | Member object pointer or member function pointer. |
| `bool` | is_bounded_array (C++20) | `is_bounded_array<T>`<br>`is_bounded_array_v<int[3]>` → `true`<br>`is_bounded_array_v<int[]>` → `false` | Array with a known size (`int[3]`). |
| `bool` | is_unbounded_array (C++20) | `is_unbounded_array<T>`<br>`is_unbounded_array_v<int[]>` → `true` | Array of unknown size (`int[]`). |
| `bool` | is_scoped_enum (C++23) | `is_scoped_enum<T>`<br>`is_scoped_enum_v<E>` → `true`<br>`is_scoped_enum_v<UE>` → `false` | True for `enum class`, false for plain `enum`. |


---

## 5. Type properties

Questions about a type's cv-qualification, memory behaviour and class design.


| Returns | Trait | Usage | Note |
|---------|-------|-------|------|
| `bool` | is_const / is_volatile | `is_const<T>`, `is_volatile<T>`<br>`is_const_v<const int>` → `true`<br>`is_const_v<const int*>` → `false`<br>`is_const_v<int* const>` → `true`<br>`is_const_v<const int&>` → `false` | **Top-level** cv only: `const int*` is a pointer to const, the pointer itself is not const. References are never const. |
| `bool` | is_trivially_copyable | `is_trivially_copyable<T>`<br>`is_trivially_copyable_v<S>` → `true`<br>`is_trivially_copyable_v<string>` → `false` | Safe to copy with `memcpy`: serialization, shared memory, sending over a socket. `std::string` is not. |
| `bool` | is_standard_layout | `is_standard_layout<T>`<br>`is_standard_layout_v<S>` → `true`<br>`is_standard_layout_v<Poly>` → `false` | C-compatible layout: safe to pass to C code, `offsetof` works. Virtual functions break it. |
| `bool` | is_trivial (deprecated C++26) | `is_trivial<T>`<br>`is_trivial_v<int>` → `true`<br>`is_trivial_v<string>` → `false` | Trivially copyable **and** trivially default constructible. Deprecated in C++26: ask the two properties separately. |
| `bool` | is_pod (deprecated C++20) | `is_pod<T>`<br>`is_pod_v<S>` → `true` | Trivial and standard layout. Deprecated: use `is_trivially_copyable` / `is_standard_layout`. |
| `bool` | is_literal_type (removed C++20) | `is_literal_type<T>`<br>*(removed)* | Was: can be used in `constexpr` evaluation. Removed from the standard in C++20. |
| `bool` | is_empty | `is_empty<T>`<br>`is_empty_v<Empty>` → `true`<br>`is_empty_v<S>` → `false` | Class with no non-static data members and no virtual members: eligible for the empty base optimization. |
| `bool` | is_polymorphic | `is_polymorphic<T>`<br>`is_polymorphic_v<Poly>` → `true`<br>`is_polymorphic_v<S>` → `false` | Has at least one virtual function (declared or inherited): `dynamic_cast` and RTTI work. |
| `bool` | is_abstract | `is_abstract<T>`<br>`is_abstract_v<Abs>` → `true` | Has a pure virtual function: cannot be instantiated. |
| `bool` | is_final (C++14) | `is_final<T>`<br>`is_final_v<Fin>` → `true` | Class declared `final`. |
| `bool` | is_aggregate (C++17) | `is_aggregate<T>`<br>`is_aggregate_v<S>` → `true`<br>`is_aggregate_v<string>` → `false`<br>`is_aggregate_v<int[3]>` → `true` | Can be initialized member by member with braces (`S{1, 2.0}`): plain structs and arrays. |
| `bool` | is_signed / is_unsigned | `is_signed<T>`, `is_unsigned<T>`<br>`is_signed_v<int>` → `true`<br>`is_signed_v<unsigned>` → `false`<br>`is_signed_v<float>` → `true`<br>`is_signed_v<bool>` → `false`<br>`is_signed_v<E>` → `false`<br>`is_unsigned_v<bool>` → `true`<br>`is_unsigned_v<float>` → `false` | Arithmetic types only. Floating point is signed; `bool` is **unsigned**; enums are neither. |
| `bool` | has_virtual_destructor | `has_virtual_destructor<T>`<br>`has_virtual_destructor_v<Poly>` → `true`<br>`has_virtual_destructor_v<S>` → `false` | Deleting through a base pointer is safe. Check this for base classes. |
| `bool` | has_unique_object_representations (C++17) | `has_unique_object_representations<T>`<br>`has_unique_object_representations_v<int>` → `true`<br>`has_unique_object_representations_v<float>` → `false`<br>`has_unique_object_representations_v<Padded>` → `false` | Equal values have identical bytes (no padding, no float quirks): safe to hash or compare by bytes. |
| `bool` | is_implicit_lifetime (C++23) | `is_implicit_lifetime<T>`<br>*not available in libstdc++ 13* | Objects of this type can be created implicitly (for example by `memcpy` or `malloc` into raw storage). |


---

## 6. Supported operations

"Can I construct / assign / destroy / swap this type, and is it cheap?" Each operation comes in three flavours:

* plain `is_X`: the operation is valid.
* `is_trivially_X`: it is valid **and trivial** (a bit copy, no user code).
* `is_nothrow_X`: it is valid **and `noexcept`** (important for `std::vector` reallocation, which only moves if the move constructor cannot throw).


| Returns | Trait | Usage | Note |
|---------|-------|-------|------|
| `bool` | is_constructible | `is_constructible<T, Args...>`<br>`is_trivially_constructible<T, Args...>`<br>`is_nothrow_constructible<T, Args...>`<br>`is_constructible_v<string, const char*>` → `true`<br>`is_constructible_v<int, int*>` → `false`<br>`is_constructible_v<string, size_t, char>` → `true` | Can `T(args...)` be formed? The general form; the ones below are shortcuts for common argument lists. |
| `bool` | is_default_constructible | `is_default_constructible<T>` (+ `trivially_` / `nothrow_`)<br>`is_default_constructible_v<NoDef>` → `false`<br>`is_default_constructible_v<int>` → `true`<br>`is_trivially_default_constructible_v<int>` → `true`<br>`is_trivially_default_constructible_v<string>` → `false` | `T()` is valid. A trivial default constructor leaves the object uninitialized. |
| `bool` | is_copy_constructible | `is_copy_constructible<T>` (+ `trivially_` / `nothrow_`)<br>`is_copy_constructible_v<unique_ptr<int>>` → `false`<br>`is_trivially_copy_constructible_v<S>` → `true` | `T(const T&)` is valid. |
| `bool` | is_move_constructible | `is_move_constructible<T>` (+ `trivially_` / `nothrow_`)<br>`is_move_constructible_v<unique_ptr<int>>` → `true`<br>`is_nothrow_move_constructible_v<string>` → `true`<br>`is_nothrow_move_constructible_v<Throwing>` → `false` | `T(T&&)` is valid. `is_nothrow_move_constructible` decides whether containers move or copy on reallocation. |
| `bool` | is_assignable | `is_assignable<To, From>`<br>`is_trivially_assignable<To, From>`<br>`is_nothrow_assignable<To, From>`<br>`is_assignable_v<int&, int>` → `true`<br>`is_assignable_v<int, int>` → `false` | Is `declval<To>() = declval<From>()` valid? Note the order (destination first) and that `To` must be an lvalue reference to be assignable. |
| `bool` | is_copy_assignable / is_move_assignable | `is_copy_assignable<T>`, `is_move_assignable<T>` (+ `trivially_` / `nothrow_`)<br>`is_copy_assignable_v<const int>` → `false`<br>`is_copy_assignable_v<string>` → `true`<br>`is_move_assignable_v<unique_ptr<int>>` → `true` | `t = other` for an lvalue / rvalue `other`. |
| `bool` | is_destructible | `is_destructible<T>` (+ `trivially_` / `nothrow_`)<br>`is_destructible_v<int>` → `true`<br>`is_destructible_v<PrivDtor>` → `false`<br>`is_trivially_destructible_v<int>` → `true`<br>`is_trivially_destructible_v<string>` → `false`<br>`is_nothrow_destructible_v<string>` → `true` | The destructor is callable. A trivially destructible type needs no cleanup code (can be abandoned or `memcpy`ed without running destructors). |
| `bool` | is_swappable (C++17) | `is_swappable<T>`<br>`is_swappable_with<T, U>`<br>`is_nothrow_swappable<T>`<br>`is_nothrow_swappable_with<T, U>`<br>`is_swappable_v<int>` → `true`<br>`is_swappable_with_v<int&, double&>` → `false`<br>`is_nothrow_swappable_v<string>` → `true` | `swap(a, b)` is valid (found via ADL). The `_with` forms swap two different types. |


---

## 7. Property queries

Return a number instead of a `bool`.


| Returns | Trait | Usage | Note |
|---------|-------|-------|------|
| `size_t` | alignment_of | `alignment_of<T>` / `alignment_of_v<T>`<br>`alignment_of_v<double>` → `8`<br>`alignment_of_v<char>` → `1` | Alignment requirement in bytes (same as `alignof(T)`). |
| `size_t` | rank | `rank<T>` / `rank_v<T>`<br>`rank_v<int[2][3]>` → `2`<br>`rank_v<int>` → `0` | Number of array dimensions; 0 for a non-array. |
| `size_t` | extent | `extent<T, N = 0>` / `extent_v<T, N = 0>`<br>`extent_v<int[2][3]>` → `2`<br>`extent_v<int[2][3], 1>` → `3`<br>`extent_v<int[]>` → `0`<br>`extent_v<int>` → `0` | Size of dimension `N` (default: the first); 0 if that dimension is unbounded or `T` is not an array. |


---

## 8. Type relationships

Questions about **two (or more) types** at once.


| Returns | Trait | Usage | Note |
|---------|-------|-------|------|
| `bool` | is_same | `is_same<T, U>` / `is_same_v<T, U>`<br>`is_same_v<int, int>` → `true`<br>`is_same_v<int, const int>` → `false`<br>`is_same_v<int, signed int>` → `true`<br>`is_same_v<char, signed char>` → `false`<br>`is_same_v<long, long long>` → `false`<br>`is_same_v<int, int&>` → `false` | Exactly the same type: cv and references matter; `char`, `signed char` and `unsigned char` are three different types, as are `long` and `long long` even when equally wide. |
| `bool` | is_base_of | `is_base_of<Base, Derived>`<br>`is_base_of_v<Base, Derived>` → `true`<br>`is_base_of_v<Derived, Base>` → `false`<br>`is_base_of_v<S, S>` → `true`<br>`is_base_of_v<int, int>` → `false`<br>`is_base_of_v<Base, Priv>` → `true` | `Base` is a base of `Derived`, **or the same class**, any access level (private base still counts). Class types only. Argument order: base first. |
| `bool` | is_convertible | `is_convertible<From, To>`<br>`is_nothrow_convertible<From, To>` (C++20)<br>`is_convertible_v<int, double>` → `true`<br>`is_convertible_v<double*, int*>` → `false`<br>`is_convertible_v<Derived*, Base*>` → `true`<br>`is_convertible_v<int, Expl>` → `false`<br>`is_constructible_v<Expl, int>` → `true`<br>`is_nothrow_convertible_v<int, double>` → `true` | Implicit conversion `From` to `To` is valid. **Explicit constructors do not count** (compare with `is_constructible`). |
| `bool` | is_invocable (C++17) | `is_invocable<Fn, Args...>`<br>`is_invocable_r<R, Fn, Args...>`<br>`is_nothrow_invocable<Fn, Args...>`<br>`is_nothrow_invocable_r<R, Fn, Args...>`<br>`is_invocable_v<void(*)(int), int>` → `true`<br>`is_invocable_v<void(*)(int), string>` → `false`<br>`is_invocable_r_v<double, int(*)()>` → `true`<br>`is_nothrow_invocable_v<int(*)(int) noexcept, int>` → `true` | Can `Fn` be called with `Args...` (works for functions, lambdas, functors and member pointers, like `std::invoke`); the `_r` forms also require the result to convert to `R`. |
| `bool` | is_layout_compatible (C++20) | `is_layout_compatible<T, U>`<br>`is_layout_compatible_v<S, S2>` → `true` | `T` and `U` are layout-compatible: the same member types in the same order (relevant for type punning through unions and `memcpy`). |
| `bool` | is_pointer_interconvertible_base_of (C++20) | `is_pointer_interconvertible_base_of<Base, Derived>`<br>`is_pointer_interconvertible_base_of_v<Base, Derived>` → `true` | A pointer to `Derived` can be reinterpreted as a pointer to `Base` (standard-layout inheritance). |
| `bool` | is_pointer_interconvertible_with_class / is_corresponding_member (C++20) | `is_pointer_interconvertible_with_class(mp)`<br>`is_corresponding_member(mp1, mp2)`<br>*(functions on member pointers)* | Constexpr functions (not class templates) that compare member pointers for layout compatibility. |
| `bool` | reference_constructs_from_temporary (C++23) | `reference_constructs_from_temporary<To, From>`<br>`reference_converts_from_temporary<To, From>`<br>`reference_constructs_from_temporary_v<const int&, int>` → `true`<br>`reference_constructs_from_temporary_v<int&&, int>` → `true`<br>`reference_converts_from_temporary_v<const int&, int>` → `true`<br>`reference_constructs_from_temporary_v<const int&, int&>` → `false` | Would binding a reference of type `To` to a `From` value create a **temporary** (dangling-reference check)? |
| `bool` | is_virtual_base_of (C++26) | `is_virtual_base_of<Base, Derived>`<br>*not available in libstdc++ 13* | `Base` is a **virtual** base class of `Derived`. |


---

## 9. Modifications

Transformation traits: take a type, return another. The `::type` result is reached through the `_t` alias.


| Returns | Trait | Usage | Note |
|---------|-------|-------|------|
| type (`::type`, `_t`) | remove_const / remove_volatile / remove_cv | `remove_const_t<T>`<br>`remove_volatile_t<T>`<br>`remove_cv_t<T>`<br>`remove_const_t<const int>` → `int`<br>`remove_const_t<const int*>` → `const int*`<br>`remove_const_t<int* const>` → `int*`<br>`remove_volatile_t<volatile int>` → `int`<br>`remove_cv_t<const volatile int>` → `int` | Removes **top-level** cv only: `const int*` stays (the pointee is const, not the pointer). |
| type (`::type`, `_t`) | add_const / add_volatile / add_cv | `add_const_t<T>`<br>`add_volatile_t<T>`<br>`add_cv_t<T>`<br>`add_const_t<int>` → `const int`<br>`add_const_t<int&>` → `int&`<br>`add_const_t<int*>` → `int* const`<br>`add_volatile_t<int>` → `volatile int`<br>`add_cv_t<int>` → `const volatile int` | Adds top-level cv. References cannot be const, so they are returned unchanged. |
| type (`::type`, `_t`) | remove_reference | `remove_reference_t<T>`<br>`remove_reference_t<int&>` → `int`<br>`remove_reference_t<int&&>` → `int` | Strips `&` / `&&`. |
| type (`::type`, `_t`) | add_lvalue_reference / add_rvalue_reference | `add_lvalue_reference_t<T>`<br>`add_rvalue_reference_t<T>`<br>`add_lvalue_reference_t<int>` → `int&`<br>`add_lvalue_reference_t<void>` → `void`<br>`add_lvalue_reference_t<int&&>` → `int&`<br>`add_rvalue_reference_t<int>` → `int&&`<br>`add_rvalue_reference_t<int&>` → `int&`<br>`add_rvalue_reference_t<void>` → `void` | Adds `&` / `&&` following the reference-collapsing rules. `void` has no reference, so it is returned unchanged (that is what makes them safe to use on `void`). |
| type (`::type`, `_t`) | remove_cvref (C++20) | `remove_cvref_t<T>`<br>`remove_cvref_t<const int&>` → `int` | Strips reference **and** top-level cv: the usual first step before asking `is_integral_v` and friends about a deduced `T`. |
| type (`::type`, `_t`) | remove_pointer | `remove_pointer_t<T>`<br>`remove_pointer_t<int*>` → `int`<br>`remove_pointer_t<int* const>` → `int`<br>`remove_pointer_t<int**>` → `int*`<br>`remove_pointer_t<int>` → `int` | Strips one pointer level (and the pointer's own cv). |
| type (`::type`, `_t`) | add_pointer | `add_pointer_t<T>`<br>`add_pointer_t<int>` → `int*`<br>`add_pointer_t<int&>` → `int*`<br>`add_pointer_t<void>` → `void*` | Adds one `*`; removes a reference first. |
| type (`::type`, `_t`) | make_signed / make_unsigned | `make_signed_t<T>`<br>`make_unsigned_t<T>`<br>`make_signed_t<unsigned>` → `int`<br>`make_signed_t<char>` → `signed char`<br>`make_unsigned_t<int>` → `unsigned int`<br>`make_unsigned_t<const int>` → `const unsigned int`<br>`make_unsigned_t<E>` → `unsigned char` | The signed / unsigned integer type of the same width, keeping cv. Works on integers and enums (not `bool`); `char` becomes `signed char`. |
| type (`::type`, `_t`) | remove_extent / remove_all_extents | `remove_extent_t<T>`<br>`remove_all_extents_t<T>`<br>`remove_extent_t<int[2][3]>` → `int[3]`<br>`remove_all_extents_t<int[2][3]>` → `int`<br>`remove_extent_t<int>` → `int` | Strips one / all array dimensions; non-arrays are unchanged. |


---

## 10. Miscellaneous transformations


| Returns | Trait | Usage | Note |
|---------|-------|-------|------|
| type (`::type`, `_t`) | enable_if | `enable_if<B, T = void>` / `enable_if_t<B, T = void>`<br>`enable_if_t<true>` → `void`<br>`enable_if_t<true, int>` → `int`<br>`enable_if_t<false>` → *error: no member `type`* | Has a `::type` only when `B` is true. Inside a template this failure is not an error but removes the overload (**SFINAE**), which is how you enable a function only for certain types. In C++20 prefer concepts / `requires`. |
| type (`::type`, `_t`) | conditional | `conditional<B, T, F>` / `conditional_t<B, T, F>`<br>`conditional_t<true, int, double>` → `int`<br>`conditional_t<false, int, double>` → `double` | Compile-time `?:` for types: `T` if `B`, else `F`. |
| type (`::type`, `_t`) | decay | `decay<T>` / `decay_t<T>`<br>`decay_t<int&>` → `int`<br>`decay_t<const int>` → `int`<br>`decay_t<int[3]>` → `int*`<br>`decay_t<int(int)>` → `int(*)(int)`<br>`decay_t<const int&>` → `int` | What happens to a type when passed **by value**: strips reference and cv, arrays become pointers, functions become function pointers. Same rule as `auto`. |
| type (`::type`, `_t`) | common_type | `common_type<Ts...>` / `common_type_t<Ts...>`<br>`common_type_t<int, double>` → `double`<br>`common_type_t<int, long, char>` → `long`<br>`common_type_t<char, short>` → `int`<br>`common_type_t<string, const char*>` → `std::string`<br>`common_type_t<int&, long&>` → `long` | The type every `Ts` can be converted to (the type of `cond ? a : b`). Used for example by `std::chrono` for mixed-unit arithmetic. Specializable for user types. |
| type (`::type`, `_t`) | common_reference (C++20) | `common_reference<Ts...>` / `common_reference_t<Ts...>`<br>`basic_common_reference<T, U, TQual, UQual>`<br>`common_reference_t<int&, const int&>` → `const int&`<br>`common_reference_t<int&, int&&>` → `const int&` | Like `common_type` but keeps references. `basic_common_reference` is the customization point for proxy-reference types. |
| type (`::type`, `_t`) | underlying_type | `underlying_type<E>` / `underlying_type_t<E>`<br>`underlying_type_t<E>` → `unsigned char`<br>`underlying_type_t<UE>` → `unsigned int` | The integer type an enum is stored as. |
| type (`::type`, `_t`) | invoke_result (C++17) | `invoke_result<Fn, Args...>` / `invoke_result_t<Fn, Args...>`<br>`invoke_result_t<int(*)(double), int>` → `int`<br>`invoke_result_t<decltype(&Poly::f), Poly&>` → `void` | Return type of calling `Fn` with `Args...`. Replaces `result_of` (deprecated in C++17, removed in C++20). |
| type (`::type`, `_t`) | unwrap_reference / unwrap_ref_decay (C++20) | `unwrap_reference_t<T>`<br>`unwrap_ref_decay_t<T>`<br>`unwrap_reference_t<reference_wrapper<int>>` → `int&`<br>`unwrap_ref_decay_t<reference_wrapper<int>>` → `int&`<br>`unwrap_ref_decay_t<const int&>` → `int` | Turn `reference_wrapper<X>` into `X&`; `unwrap_ref_decay` also applies `decay` otherwise. Used by `std::make_pair`/`make_tuple`. |
| type (`::type`, `_t`) | aligned_storage (deprecated C++23) | `aligned_storage<Len, Align = default-alignment>`<br>`sizeof(aligned_storage_t<5, 4>)` → `8` | Raw storage of at least `Len` bytes with alignment `Align`. Deprecated: use `alignas(Align) std::byte buf[Len];`. |
| type (`::type`, `_t`) | aligned_union (deprecated C++23) | `aligned_union<Len, Ts...>`<br>*(deprecated)* | Storage big enough for any of `Ts...`. Deprecated: use `std::variant` or `alignas(Ts...) std::byte buf[...]`. |
| type (`::type`, `_t`) | result_of (removed C++20) | `result_of<Fn(Args...)>`<br>*(removed)* | Old spelling of `invoke_result`. |


---

## 11. Logic on traits

Combine traits without writing `&&`/`||` on `::value` (these **short-circuit**: instantiation stops at the first decisive trait, so later traits may even be ill-formed for that `T`).


| Returns | Trait | Usage | Note |
|---------|-------|-------|------|
| `bool` | conjunction (C++17) | `conjunction<Bs...>` / `conjunction_v<Bs...>`<br>`conjunction_v<true_type, false_type>` → `false`<br>`conjunction_v<>` → `true` | Logical AND; stops at the first false trait. Empty pack is `true`. |
| `bool` | disjunction (C++17) | `disjunction<Bs...>` / `disjunction_v<Bs...>`<br>`disjunction_v<false_type, true_type>` → `true`<br>`disjunction_v<>` → `false` | Logical OR; stops at the first true trait. Empty pack is `false`. |
| `bool` | negation (C++17) | `negation<B>` / `negation_v<B>`<br>`negation_v<true_type>` → `false` | Logical NOT of a trait. |


---

## 12. Trait or concept?

`<concepts>` (C++20) has concept versions of many traits. They are used in `requires` clauses and constrained templates. They are **not always identical** to the trait (checked on g++ 13.3):

| Trait | Concept | Difference |
|---|---|---|
| `is_same_v<T, U>` | `same_as<T, U>` | Same result. |
| `is_base_of_v<Base, Derived>` | `derived_from<Derived, Base>` | **Argument order is swapped.** `derived_from` also needs an accessible, unambiguous base: for a private base `is_base_of_v` is `true` but `derived_from<Priv, Base>` is `false` (verified). |
| `is_convertible_v<From, To>` | `convertible_to<From, To>` | The concept additionally requires an explicit conversion to work. For `Expl` both are `false` for `int`, while `constructible_from<Expl, int>` is `true` (verified). |
| `is_constructible_v<T, Args...>` | `constructible_from<T, Args...>` | Concept also requires `T` to be destructible. |
| `is_assignable_v<To, From>` | `assignable_from<To, From>` | Concept requires `To` to be an lvalue reference: `assignable_from<int&, int>` is `true` (verified). |
| `is_integral_v<T>` | `integral<T>` | Same; `integral<bool>` is `true` (verified). Also `signed_integral` (`signed_integral<unsigned>` is `false`, verified) and `unsigned_integral`. |
| `is_floating_point_v<T>` | `floating_point<T>` | Same. |
| `is_invocable_v<Fn, Args...>` | `invocable<Fn, Args...>` | Same; `regular_invocable` additionally promises no state changes. |
| `is_destructible_v<T>` | `destructible<T>` | Same. |
| `is_swappable_v<T>` | `swappable<T>` | Same idea (uses `ranges::swap`). |
| `common_type_t` / `common_reference_t` | `common_with` / `common_reference_with` | Concepts check that a common type exists and both convert to it. |

---

## 13. Which standard added what

| Standard | Added |
|---|---|
| C++11 | The whole library: the categories and properties above, `is_constructible` family, `is_same`, `is_base_of`, `is_convertible`, `remove_*`/`add_*`, `make_signed`/`make_unsigned`, `enable_if`, `conditional`, `decay`, `common_type`, `underlying_type`, `result_of`, `aligned_storage`/`aligned_union`, `alignment_of`/`rank`/`extent`, `integral_constant`/`true_type`/`false_type`. |
| C++14 | `_t` alias for every transformation, `is_final`, `is_null_pointer`, `operator()` on `integral_constant`. |
| C++17 | `_v` variable for every query, `bool_constant`, `conjunction`/`disjunction`/`negation`, `void_t`, `is_swappable` family, `is_invocable` family and `invoke_result`, `is_aggregate`, `has_unique_object_representations`. |
| C++20 | `remove_cvref`, `type_identity`, `is_bounded_array`/`is_unbounded_array`, `is_nothrow_convertible`, `is_constant_evaluated`, `unwrap_reference`/`unwrap_ref_decay`, `common_reference`/`basic_common_reference`, `is_layout_compatible`, `is_pointer_interconvertible_base_of`/`_with_class`, `is_corresponding_member`. |
| C++23 | `is_scoped_enum`, `is_implicit_lifetime`, `reference_constructs_from_temporary`/`reference_converts_from_temporary`. |
| C++26 | `is_virtual_base_of`, `is_within_lifetime`, `is_reflection`. (`is_trivially_relocatable` and `is_replaceable` were adopted in 2025 but **removed from C++26** at the Kona meeting and are not part of it.) |

| Status | Traits |
|---|---|
| Deprecated | `is_pod` (C++20), `aligned_storage` / `aligned_union` (C++23), `is_trivial` (C++26). |
| Removed | `result_of` and `is_literal_type` (C++20). |

---

## 14. Patterns

All of these were compiled and run (g++ 13.3, `-std=c++23`).

```cpp
// 1. Enable an overload only for some types (SFINAE)
template<class T, std::enable_if_t<std::is_integral_v<T>, int> = 0>       T twice(T x) { return x * 2; }
template<class T, std::enable_if_t<std::is_floating_point_v<T>, int> = 0> T twice(T x) { return x * 2.0; }
// twice(3) -> 6, twice(1.5) -> 3, twice(std::string{}) does not compile

// 2. Branch at compile time (the untaken branch is not even compiled)
template<class T> auto deref_if_ptr(T x) { if constexpr (std::is_pointer_v<T>) return *x; else return x; }

// 3. Fail the build early with a readable message
struct Packet { int id; float v; };
static_assert(std::is_trivially_copyable_v<Packet>, "must be memcpy-safe");

// 4. Detection idiom: does T have .size()?
template<class T, class = void> struct has_size : std::false_type {};
template<class T> struct has_size<T, std::void_t<decltype(std::declval<T>().size())>> : std::true_type {};
// has_size<std::vector<int>>::value -> true, has_size<int>::value -> false

// 5. Combine traits with short-circuit
template<class T> using small_signed = std::conjunction<std::is_signed<T>, std::bool_constant<(sizeof(T) <= 4)>>;
// small_signed<int> -> true, small_signed<long long> -> false, small_signed<unsigned> -> false

// 6. Tag dispatch: pick an overload with true_type / false_type
template<class T> const char* kind_impl(T, std::true_type)  { return "integral"; }
template<class T> const char* kind_impl(T, std::false_type) { return "other"; }
template<class T> const char* kind(T x) { return kind_impl(x, std::is_integral<T>{}); }
```

---

## 15. Pitfalls

| Surprise | Why | Fix |
|---|---|---|
| `is_integral_v<bool>` and `<char>` are `true` | They are integer types | Exclude explicitly if you mean "numbers". |
| `is_integral_v<int&>` is `false`, `is_integral_v<const int>` is `true` | References are not integral; cv is ignored | `remove_cvref_t<T>` first. |
| `is_const_v<const int*>` is `false` | The pointer is not const, the pointee is | Use `is_const_v<remove_pointer_t<T>>`. |
| `is_same_v<int, const int>` / `<int, int&>` are `false` | cv and references are part of the type | `remove_cvref_t` or `decay_t` first. |
| `is_same_v<char, signed char>` and `<long, long long>` are `false` | Distinct types | Compare against the exact type you mean. |
| `is_signed_v<float>` is `true`, `is_signed_v<bool>` is `false`, `is_unsigned_v<bool>` is `true` | The traits cover all arithmetic types | Combine with `is_integral`. |
| `is_base_of_v<S, S>` is `true` but `is_base_of_v<int, int>` is `false` | A class counts as its own base; non-classes have no bases | Add `!is_same_v<A, B>` if needed. |
| `is_convertible_v<int, Expl>` is `false` but `is_constructible_v<Expl, int>` is `true` | `is_convertible` only checks **implicit** conversion | Pick the one that matches how you will write the code. |
| `is_assignable_v<int, int>` is `false` | You cannot assign to an rvalue `int` | `is_assignable_v<int&, int>`. |
| `add_lvalue_reference_t<void>` is `void` | There is no `void&` | Check `is_void` first if you need a real reference. |
| `remove_const_t<const int*>` does nothing | Only top-level const is removed | `remove_const_t<remove_pointer_t<T>>*` for the pointee. |
| `make_signed_t<char>` is `signed char` | `char`, `signed char`, `unsigned char` are distinct | Use a fixed-width type if exactness matters. |
| `enable_if_t<false>` outside a template is a hard error | SFINAE only works during template deduction | Put the condition on a template parameter of the function. |
| `is_trivially_copyable_v<std::string>` is `false` | It owns heap memory | Only `memcpy` types for which the trait is `true`. |
| Specializing a standard trait yourself is undefined behaviour | The library owns them | Only the customization points the standard names are allowed (for example `common_type`, `basic_common_reference`). |
| Many traits on an **incomplete** type are undefined behaviour | They need the full definition | Make sure the type is complete where the trait is evaluated. |
| `if constexpr (std::is_constant_evaluated())` is always `true` | The condition of `if constexpr` is itself constant evaluated | Use a plain `if`, or `if consteval` in C++23. |

---

*Source: g++ 13.3.0 (Ubuntu 24.04), `-std=c++23`; about 200 examples compiled and evaluated individually. Features marked C++26 follow the finalized standard (March 2026); trivial relocatability is not in it (removed at the Kona meeting, November 2025).*
