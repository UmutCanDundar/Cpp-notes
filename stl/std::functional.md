# `<functional>`

### Notation
* `R(Args...)`: the call signature a `std::function` erases to — return type `R`, parameter types `Args...`.
* `fn`, `f`: any callable (function pointer, lambda, functor, member-pointer).
* `x`, `y`: arbitrary values passed to a comparator/arithmetic function object.

---

### Constructors (`std::function<R(Args...)>`)

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| *(constructor)* | default | `function<R(Args...)> f;` | O(1) | Constructs an empty `function` — calling it throws `std::bad_function_call`. |
| *(constructor)* | from callable | `function<R(Args...)> f(fn)` | O(1) (may heap-allocate) | Wraps any callable matching the signature. Small callables may use small-object optimization; larger ones (most lambdas with captures) heap-allocate. |
| *(constructor)* | copy / move | `function<R(Args...)> f(other)` | O(1)–O(size of captured state) | Copies (may allocate) · moves (steals the heap block if any). |

---

### Methods

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| `R` | operator() | `f(args...)` | O(1) (virtual call) + cost of `fn` | Invokes the wrapped callable. Type erasure + (often) heap allocation + virtual call — never on the hot path; use templates or raw function pointers instead. |
| `bool` | operator bool() | `if (f) ...` | O(1) | True if `f` wraps a callable (not default/moved-from empty). |
| `const type_info&` | target_type | `f.target_type()` | O(1) | RTTI info describing the wrapped callable's type. |
| `void` | swap | `f.swap(other)` | O(1) | Exchanges the wrapped callables. |
| — | Lambda + `auto` parameter | *(language feature, not a function)* | O(0) (fully inlined) | A template (generic) lambda used in place of `std::function`. Zero overhead — gets inlined at the call site. Preferred over `std::function` wherever the target type is knowable at compile time. |
| `invoke_result_t<F,Args...>` | invoke | `std::invoke(fn, args...)` | O(1) + cost of `fn` | Uniformly calls any callable — plain functions, lambdas, function objects, and (unlike calling directly) member function/data pointers too. |
| *(bind expression)* | bind | `std::bind(fn, args...)` | O(1) (allocates) | Partial application / argument reordering. Made largely redundant by lambdas — has overhead and is hard to read; avoid in new code. |
| *(bind expression, C++20)* | bind_front | `std::bind_front(fn, arg1, arg2)` | O(1) | Binds the leading arguments only (no placeholders needed, unlike `bind`) — simpler, but a lambda is usually still clearer/faster. |
| *(bind expression, C++23)* | bind_back | `std::bind_back(fn, arg1, arg2)` | O(1) | Same as `bind_front`, but binds the trailing arguments. |
| `bool` | less\<\> / greater\<\> | `std::less<>{}`<br>`std::greater<>{}` | O(1) call | Transparent (template-less, deduced-argument) comparators for `sort`, `priority_queue`, associative containers. The `<>` empty-template form additionally allows heterogeneous lookup (e.g. comparing a `map<string,...>` key against a `string_view`). |
| `size_t` | hash\<T\> | `std::hash<T>{}(x)` | O(size of x) typically | Hash function object used by `unordered_map`/`unordered_set`. Specialize for custom key types. |
| `T&` | reference_wrapper\<T\> / ref / cref | `std::ref(x)`<br>`std::cref(x)` | O(1) | Makes a reference copyable/assignable so it can be stored in a container (e.g. `vector<reference_wrapper<T>>`) or passed through `std::bind`/`std::function`. `cref` produces a const reference wrapper. |
| *(callable)* | mem_fn | `std::mem_fn(&Class::method)` | O(1) | Wraps a member function pointer as a regular callable. Superseded by lambdas/`invoke` in almost all cases — avoid in new code. |
| *(predicate)* | not_fn | `std::not_fn(pred)` | O(1) | Negates a predicate's result. Rarely needed once you can just write `!pred(x)` directly in a lambda. |
| `T` | plus / minus / multiplies / divides / modulus / negate | `std::plus<>{}(x, y)` (etc.) | O(1) call | Function-object wrappers around `+ - * / %` and unary `-`. Mainly useful as a policy/comparator parameter to algorithms like `std::accumulate` or `std::transform`. |
| `bool` | equal_to / not_equal_to / greater_equal / less_equal | `std::equal_to<>{}(x, y)` (etc.) | O(1) call | Function-object wrappers around `== != >= <=`, same use case as above. |
| `bool` | logical_and / logical_or / logical_not | `std::logical_and<>{}(x, y)` (etc.) | O(1) call | Function-object wrappers around `&& \|\| !`. |
