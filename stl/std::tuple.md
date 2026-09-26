# `<tuple>`

### Notation
* `Ts...`: the tuple's element types. `t`, `t1`, `t2`: instances of a `tuple`. `I`: a compile-time index.
* `a`, `b`, `c`, ...: individual values used to build a tuple, or variables to unpack one into.

---

### Constructors

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| *(constructor)* | default | `tuple<Ts...> t;` | O(1) per element (default-constructs each) | Requires every `Ts` to be default-constructible. |
| *(constructor)* | from values | `tuple<Ts...> t(a, b, c)` | O(1) per element | Constructs each element by copy/move from the given arguments. |
| *(constructor)* | from pair | `tuple<T1,T2> t(p)` | O(1) | Constructs a 2-element tuple from a `std::pair<T1,T2>`. |
| *(constructor)* | copy / move | `tuple<Ts...> t(other)` | O(1) per element | Copies/moves each element in turn. |

---

### Methods

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| `Ti&` | get\<I\>(t) / get\<T\>(t) | `std::get<0>(t)`<br>`std::get<int>(t)` | O(1) | Access element by compile-time index, or (only if the type is unique within the tuple) by type. |
| `tuple<Ts...>` | make_tuple(a, b, c) | `std::make_tuple(a, b, c)` | O(1) per element | Creates a tuple with element types deduced from the arguments (decays references/const, unlike direct construction). |
| — | tie(a, b) = t | `std::tie(a, b) = t;` | O(1) per element | Builds a tuple of **references** to existing variables, then assigns from `t` into them — the classic multi-return unpacking idiom pre-C++17. |
| — | ignore | `std::tie(a, std::ignore) = t;` | O(1) | Placeholder usable anywhere in a `tie()` to skip/discard the corresponding element. |
| `auto [a, b] = t;` | structured bindings (C++17) | `auto [a, b] = t;` | O(1) per element | Cleaner than `std::tie` for unpacking — declares and binds new variables in one step; also works directly on `pair`, `array`, and aggregate structs. |
| `tuple<Ts1..., Ts2...>` | tuple_cat(t1, t2) | `std::tuple_cat(t1, t2, ...)` | O(total elements) | Concatenates any number of tuples (and pairs/arrays) into a single flat tuple. |
| `R` | apply(fn, t) | `std::apply(fn, t)` | O(1) + cost of `fn` | Calls `fn` with the tuple's elements unpacked as positional arguments — turns a tuple back into a parameter list. |
| `size_t` | tuple_size_v\<T\> | `std::tuple_size_v<decltype(t)>` | O(1) (compile-time) | Compile-time number of elements in a tuple type. |
| `type` | tuple_element_t\<I,T\> | `std::tuple_element_t<0, decltype(t)>` | O(1) (compile-time) | Compile-time type of the `I`-th element — used in generic template code. |
| `void` | swap(other) | `t1.swap(t2)` | O(1) per element | Exchanges contents element-wise with another tuple of the same type. |
| `bool` | operator== / operator\< (etc.) | `t1 == t2` | O(1) per element | Lexicographic comparison, element by element (all elements must be comparable). |
