# `<utility>` `<tuple>` `<optional>` `<variant>` `<any>`

| API | Priority | Description |
|-----|----------|-------------|
| `std::pair<T,U>` | memorize | Two values. `.first` / `.second`. Used for map iterator pairs. |
| `std::make_pair(a, b)` | memorize | Create a pair. |
| `std::tuple<Ts...>` | know | N values. Access with `std::get<0>(t)`. |
| `std::tie(a, b) = t` | know | Unpack tuple into variables. |
| `std::move(x)` | memorize | Cast to rvalue. Triggers move instead of copy. Does not actually move — it casts. |
| `std::forward<T>(x)` | memorize | Perfect forwarding. Preserves lvalue/rvalue in template functions. |
| `std::exchange(obj, new_val)` | know | Returns old value, assigns new. Common in move constructors. |
| `std::optional<T>` | know | Value may or may not exist. `has_value()` / `value()` / `value_or()`. No allocation. |
| `std::variant<Ts...>` | memorize | Type-safe union. Dispatch with `std::visit`. Replaces virtual. Present in your project. |
| `std::visit(fn, var)` | memorize | Apply fn to the active type in the variant. |
| `std::holds_alternative<T>(var)` | know | Is the variant currently holding type T? |
| `std::any` | avoid | Type erasure + heap allocation. Never on hot path. |
