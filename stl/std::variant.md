# `<variant>` (C++17)

| API | Priority | Description |
|-----|----------|-------------|
| `std::variant<Ts...>` | memorize | Type-safe union. Dispatch with `std::visit`. Replaces virtual dispatch for a closed set of types — no vtable, no heap allocation. |
| `std::visit(fn, var)` | memorize | Apply `fn` to whichever type is currently active — usually with an overload-set lambda (variant overloader idiom). |
| `std::holds_alternative<T>(var)` | know | Is the variant currently holding type T? |
| `std::get<T>(var)` / `std::get<I>(var)` | know | Access by type or index, throws `std::bad_variant_access` on mismatch. |
| `std::get_if<T>(&var)` | memorize | Non-throwing access — returns `T*` or `nullptr`. Preferred on hot paths over exception-throwing `get`. |
| `var.index()` | know | Returns the zero-based index of the currently active alternative. |
| `std::variant_size_v<V>` | know | Compile-time count of alternatives in the variant type. |
| `std::monostate` | know | Empty placeholder type, used as the first alternative to give a variant a valid default-constructed "empty" state. |
