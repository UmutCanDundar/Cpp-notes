# `<exception>` `<stdexcept>` `<system_error>`

### Notation
* `what_arg`: a human-readable error message, either `const std::string&` or `const char*`.
* `ec`: an `error_code` instance. `ev`: a raw integer error value. `cat`: an `error_category` (e.g. `std::generic_category()`, `std::system_category()`).
* `e`: an exception object (any type). `p`: an `exception_ptr`.

---

### Constructors

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| *(constructor)* | logic_error / runtime_error family | `logic_error(what_arg)`<br>`runtime_error(what_arg)` | O(n), n = message length | All standard subclasses — `invalid_argument`, `out_of_range`, `length_error`, `domain_error` (of `logic_error`); `overflow_error`, `underflow_error`, `range_error` (of `runtime_error`) — inherit this exact constructor signature from their base. |
| *(constructor)* | error_code, default | `error_code ec;` | O(1) | Default-constructs with value `0` and `system_category()` (i.e. represents "no error"). |
| *(constructor)* | error_code, from value + category | `error_code ec(ev, cat)` | O(1) | Constructs from a numeric error value and an `error_category`. |
| *(constructor)* | error_code, from enum | `error_code ec(my_enum_value)` | O(1) | Implicit conversion via ADL-found `make_error_code(my_enum_value)`, if the enum specializes `std::is_error_code_enum`. |
| *(constructor)* | system_error | `system_error(ec)`<br>`system_error(ec, what_arg)`<br>`system_error(ev, cat)`<br>`system_error(ev, cat, what_arg)` | O(n), n = message length | Wraps an `error_code` (built directly, or from a value + category pair) as a throwable exception, with an optional `what_arg` prefix message. |

---

### Methods

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| `const char*` | what() | `e.what()` | O(1) | Virtual; returns the stored message. On `system_error`, combines the custom `what_arg` (if any) with the wrapped `error_code`'s message. |
| `int` | value() | `ec.value()` | O(1) | The raw numeric error value stored in an `error_code`. |
| `const error_category&` | category() | `ec.category()` | O(1) | The `error_category` this code's value is interpreted against. |
| `string` | message() | `ec.message()`<br>`cat.message(ev)` | O(1) | Human-readable description of the code · same, called directly on a category for a raw value. |
| `bool` | operator bool() | `if (ec) ...` | O(1) | `true` if `ec.value() != 0` (i.e. an actual error is set). |
| `void` | clear() | `ec.clear()` | O(1) | Resets to value `0` / `system_category()` ("no error"). |
| `const char*` | name() | `cat.name()` | O(1) | Short identifying name of the category, e.g. `"generic"` or `"system"`. |
| `bool` | equivalent() | `cat.equivalent(ev, cond)` | O(1) | Checks whether an error value in this category maps to a given portable `error_condition` — lets cross-platform code compare OS-specific codes generically. |
| `const error_category&` | generic_category() | `std::generic_category()` | O(1) | Singleton category for POSIX-style (`errno`) error values. |
| `const error_category&` | system_category() | `std::system_category()` | O(1) | Singleton category for OS-native error values (`errno` on POSIX, `GetLastError()` on Windows). |
| `const error_code&` | code() | `system_error_instance.code()` | O(1) | The `error_code` wrapped inside a `system_error` exception. |
| `exception_ptr` | current_exception() | `std::current_exception()` | O(1) | Captures the exception currently being handled (call inside a `catch` block) as a type-erased, copyable handle. |
| `[[noreturn]] void` | rethrow_exception() | `std::rethrow_exception(p)` | O(1) | Throws the exception held by `p`. Used to rethrow a captured exception later or on another thread. |
| `exception_ptr` | make_exception_ptr() | `std::make_exception_ptr(e)` | O(1) | Wraps any exception object `e` into an `exception_ptr` directly, without needing a `throw`/`catch` round-trip. |
| `[[noreturn]] void` | terminate() | `std::terminate()` | O(1) | Called when exception handling can't proceed (e.g. exception escaping a `noexcept` function, or an unhandled exception). Invokes the current terminate handler (default: calls `abort()`). |
| `terminate_handler` | set_terminate() / get_terminate() | `std::set_terminate(f)`<br>`std::get_terminate()` | O(1) | Installs a custom handler to run before termination · retrieves the currently installed handler. |
| `int` | uncaught_exceptions() | `std::uncaught_exceptions()` | O(1) | Count of exceptions currently being processed (unwound-through). Used in destructors/scope-guards to detect "are we unwinding due to an exception" (C++17; replaced the older bool-returning `uncaught_exception()`). |
| `[[noreturn]] void` | throw_with_nested() | `std::throw_with_nested(e)` | O(1) | Throws `e` wrapped in a `nested_exception`, capturing whatever exception is currently active (if any) as its nested cause. |
| `void` | rethrow_if_nested() | `std::rethrow_if_nested(e)` | O(1) | If `e` (caught via `catch`) carries a nested exception (was thrown with `throw_with_nested`), rethrows that nested exception; otherwise does nothing. |

> **`throw` on the hot path — avoid.** Exceptions have near-zero cost when not thrown, but throwing is expensive (stack unwinding) — never use exceptions for expected/frequent control flow in latency-sensitive code. Prefer `error_code`/`std::expected`-style returns for expected failure paths.
