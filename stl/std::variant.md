# `<variant>` (C++17)

### Notation
* `Ts...`: the variant's alternative types. `var`: an instance of `variant<Ts...>`. `T`: one of the alternative types. `I`: a compile-time index into `Ts...`.
* `fn`: a callable (usually an overload-set lambda) passed to `visit`.

---

### Constructors

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| *(constructor)* | default | `variant<Ts...> var;` | O(1) + cost of first alternative's default ctor | Constructs holding a value-initialized instance of the **first** alternative type (which must be default-constructible unless `monostate` is placed first). |
| *(constructor)* | from value | `variant<Ts...> var(value)` | O(1) + cost of `T`'s constructor | Constructs holding whichever alternative best matches `value`'s type via overload resolution. |
| *(constructor)* | in_place | `variant<Ts...> var(std::in_place_type<T>, args...)`<br>`variant<Ts...> var(std::in_place_index<I>, args...)` | O(1) + cost of `T`'s constructor | Constructs the specified alternative directly in place from `args...`, avoiding a temporary + move. Disambiguates when multiple alternatives could otherwise match. |
| *(constructor)* | copy / move | `variant<Ts...> var(other)` | O(1) + cost of the active alternative's ctor | Copies/moves whichever alternative is currently active in `other`. |

---

### Methods

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| `R` | visit(fn, var) | `std::visit(fn, var)` | O(1) dispatch + cost of `fn` | Applies `fn` to whichever type is currently active — usually with an overload-set lambda (the "variant overloader" idiom: `overloaded{[](int){}, [](string&){}}`). Replaces virtual dispatch for a closed set of types — no vtable, no heap allocation. |
| `bool` | holds_alternative\<T\>(var) | `std::holds_alternative<T>(var)` | O(1) | Is the variant currently holding type `T`? |
| `T&` | get\<T\>(var) / get\<I\>(var) | `std::get<T>(var)`<br>`std::get<I>(var)` | O(1) | Access by type or index; throws `std::bad_variant_access` if that alternative isn't currently active. |
| `T*` | get_if\<T\>(&var) / get_if\<I\>(&var) | `std::get_if<T>(&var)` | O(1) | Non-throwing access — returns a pointer to the value, or `nullptr` if that alternative isn't active. Preferred on hot paths over exception-throwing `get`. |
| `size_t` | index() | `var.index()` | O(1) | Zero-based index of the currently active alternative within `Ts...`. |
| `bool` | valueless_by_exception() | `var.valueless_by_exception()` | O(1) | True only in the rare case where a previous `emplace`/assignment threw partway through, leaving the variant holding no value at all. |
| `T&` | emplace\<T\>(args...) / emplace\<I\>(args...) | `var.emplace<T>(args...)` | O(1) + cost of `T`'s constructor | Destroys the current alternative, then constructs the specified one in place from `args...`. |
| `void` | swap(other) | `var1.swap(var2)` | O(1) + cost of the active alternatives' swap/move | Exchanges contents (and active-alternative index) with another variant of the same type. |
| `size_t` | variant_size_v\<V\> | `std::variant_size_v<decltype(var)>` | O(1) (compile-time) | Compile-time count of alternatives in the variant type. |
| `type` | variant_alternative_t\<I,V\> | `std::variant_alternative_t<0, decltype(var)>` | O(1) (compile-time) | Compile-time type of the `I`-th alternative — used in generic template code. |
| *(type)* | monostate | `variant<monostate, T> var;` | N/A | Empty placeholder type, used as the first alternative to give a variant a valid, cheap-to-default-construct "empty" state (when none of `Ts...` is itself default-constructible, or you want an explicit "nothing selected yet" state). |
| *(exception)* | bad_variant_access | `catch (const std::bad_variant_access& e)` | O(1) | Thrown by `get<T>`/`get<I>` when the requested alternative isn't the one currently active. |
