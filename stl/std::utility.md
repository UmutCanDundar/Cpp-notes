# `<utility>`

### Notation
* `T`, `U`: the element types of a `pair`. `a`, `b`: values.
* `x`: a value being cast/forwarded/exchanged. `obj`, `new_val`: target of `exchange`.
* `Is...`: a compile-time sequence of `size_t` indices, used in template metaprogramming.

---

### Constructors (`std::pair<T,U>`)

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| *(constructor)* | default | `pair<T,U> p;` | O(1) | Default-constructs `.first` and `.second`. |
| *(constructor)* | from values | `pair<T,U> p(a, b)` | O(1) | Constructs `.first` from `a`, `.second` from `b` (copy or move as appropriate). |
| *(constructor)* | copy / move | `pair<T,U> p(other)` | O(1) | Copies/moves both elements. |
| *(constructor)* | piecewise_construct | `pair<T,U> p(std::piecewise_construct, t1, t2)` | O(1) + cost of each ctor | Constructs `.first`/`.second` in place from tuples of arguments `t1`/`t2`, for element types that aren't simply copy/move-constructible from a single value. |

---

### Methods

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| `T` / `U` | .first / .second | `p.first` / `p.second` | O(1) | Direct member access — used pervasively for `map`/`unordered_map` iterator pairs. |
| `pair<T,U>` | make_pair(a, b) | `std::make_pair(a, b)` | O(1) | Creates a pair with element types deduced from the arguments (decays references/const, unlike direct construction). |
| `T&&` | move(x) | `std::move(x)` | O(1) | Casts `x` to an rvalue reference, so the next constructor/assignment picks the move overload instead of copy. **Does not itself move anything** — it's purely a cast. |
| `T&&` / `T&` | forward\<T\>(x) | `std::forward<T>(x)` | O(1) | Perfect forwarding — preserves whether the original argument was an lvalue or rvalue when passing it on inside a template function. |
| `T` | exchange(obj, new_val) | `std::exchange(obj, new_val)` | O(1) + cost of `T`'s move | Assigns `new_val` to `obj`, returns `obj`'s **old** value. Common in move constructors to null-out a source (e.g. `other.ptr_ = nullptr` in one line while keeping the old pointer to store). |
| `void` | swap(a, b) | `std::swap(a, b)` | O(1) for most types | Exchanges two values via move-construct + move-assign. Prefer a type's own member/ADL `swap` over this generic version when the type defines one (e.g. `vector::swap` is O(1) pointer swap, not element-wise). |
| `const T&` | as_const(x) | `std::as_const(x)` | O(1) | Returns a `const&` to `x` — useful to force calling a `const`-qualified overload from non-const context. |
| — | in_place / in_place_type_t / in_place_index_t | `std::in_place`<br>`std::in_place_type<T>`<br>`std::in_place_index<I>` | N/A | Tag types passed to `optional`/`variant`/`any` constructors to build the contained value in place, avoiding a temporary + move. |
| *(type)* | index_sequence\<Is...\> / make_index_sequence\<N\> | `std::make_index_sequence<N>{}` | O(1) (compile-time) | Compile-time integer sequence `0, 1, ..., N-1` — used for unpacking tuples/parameter packs via template metaprogramming (e.g. `std::apply`'s implementation). |
| `bool` | cmp_less / cmp_greater / cmp_equal (etc., C++20) | `std::cmp_less(signed_val, unsigned_val)` | O(1) | Safe comparison between signed and unsigned integers without the usual implicit-conversion UB/surprises (e.g. comparing `-1 < 0u` normally gives the wrong answer due to promotion). |
| `T&&` | declval\<T\>() | `decltype(std::declval<T>().method())` | O(1) (compile-time only, unevaluated) | Produces a fake rvalue reference to `T` for use in `decltype`/SFINAE contexts, without needing an actual object — never called at runtime. |
| `U` | to_underlying (C++23) | `std::to_underlying(enum_value)` | O(1) | Converts a scoped `enum class` value to its underlying integer type, without needing a manual `static_cast<underlying_type_t<E>>`. |
