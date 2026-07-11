# `<exception>` `<stdexcept>` `<system_error>`

| API | Priority | Description |
|-----|----------|-------------|
| `std::exception` | memorize | Base class for all standard exceptions. `.what()` returns a `const char*` message. |
| `std::runtime_error` / `std::logic_error` | memorize | Common standard exception subclasses to throw for recoverable errors. |
| `std::invalid_argument` / `std::out_of_range` / `std::length_error` | know | More specific `logic_error` subclasses for precise error signaling. |
| `std::overflow_error` / `std::underflow_error` | know | Arithmetic-related standard exceptions. |
| `throw` on hot path | avoid | Exceptions have near-zero cost when not thrown, but throwing is expensive (stack unwinding) — never use exceptions for expected/frequent control flow in latency-sensitive code. |
| `std::exception_ptr` | know | Type-erased handle to an exception, for capturing and rethrowing across contexts (e.g., threads). |
| `std::current_exception()` / `std::rethrow_exception(p)` | know | Capture the currently-handled exception and rethrow it later. |
| `std::terminate()` | know | Called when exception handling fails (e.g., exception from a noexcept function, or unhandled exception). |
| `std::uncaught_exceptions()` | know | Returns the count of exceptions currently being processed — used to detect "are we unwinding" in destructors (e.g. scope guards). |
| `std::error_code` | know | Lightweight, allocation-free error representation — an alternative to exceptions for expected failure paths (e.g. filesystem, networking APIs). |
| `std::error_category` | know | Defines the domain/meaning of an `error_code`'s numeric value (e.g. `std::generic_category()`). |
| `std::system_error` | know | Exception type that wraps an `error_code`, thrown by OS-facing standard library facilities like `<thread>` and `<filesystem>`. |
