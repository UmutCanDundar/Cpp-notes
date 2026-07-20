# Exception Handling (Advanced Level)

Exceptions handle error conditions without cluttering normal control flow. Advanced exception techniques let you build robust libraries with clear guarantees about program state after errors.

## Exception Guarantees (Basic / Strong / Noexcept)

- **Basic guarantee**: no leaks, object stays in a valid (but possibly changed) state.
- **Strong guarantee**: operation either fully succeeds or has no effect (commit-or-rollback).
- **Noexcept guarantee**: the operation never throws.

```cpp
void strongOp(std::vector<int>& v) {
    std::vector<int> copy = v; // work on copy
    copy.push_back(42);
    v = std::move(copy); // commit only if no exception thrown
}
```

## noexcept Specifier

Declares that a function will not throw exceptions; if it does, `std::terminate` is called. Helps the compiler optimize and informs callers.

```cpp
void f() noexcept { /* guaranteed not to throw */ }
```

## noexcept Operator

Checks at compile time whether an expression is declared `noexcept`, useful for conditional `noexcept` specifications.

```cpp
template <typename T>
void f() noexcept(noexcept(T().doWork())) {}
```

## When to Throw, When to Catch

Throw for truly exceptional, unrecoverable-at-this-level errors. Catch only where you can meaningfully handle or translate the error; avoid catching just to log and rethrow everywhere.

## What to Throw, What to Catch

Throw objects derived from `std::exception` for standard compatibility. Catch by `const&` to avoid slicing and unnecessary copies.

```cpp
throw std::runtime_error("bad input");
// ...
catch (const std::exception& e) { std::cerr << e.what(); }
```

## std::exception_ptr / std::current_exception / std::rethrow_exception

These let you capture an exception, store or transfer it (e.g., across threads), and rethrow it later.

```cpp
std::exception_ptr eptr;
try { risky(); }
catch (...) { eptr = std::current_exception(); }
if (eptr) std::rethrow_exception(eptr);
```

## std::nested_exception

Allows chaining exceptions, capturing an "inner" exception's context when throwing a new one, useful for layered error reporting.

```cpp
try { risky(); }
catch (...) { std::throw_with_nested(std::runtime_error("context")); }
```

## Polymorphic Exception

Refers to designing an exception class hierarchy so exceptions can be caught by base class references, allowing generic handlers to work across many derived error types.

```cpp
class MyError : public std::runtime_error {
public:
    MyError(const std::string& msg) : std::runtime_error(msg) {}
};
```

## Exception Dispatcher

A pattern where a central function catches an exception and dispatches to different handlers based on its dynamic type (often via multiple `catch` clauses or visitor-like logic).

```cpp
try { risky(); }
catch (const std::invalid_argument& e) { /* handle */ }
catch (const std::out_of_range& e) { /* handle */ }
catch (const std::exception& e) { /* fallback */ }
```

## Errors hierarchy

```
std::exception
├── std::logic_error         
│   ├── std::invalid_argument
│   ├── std::domain_error
│   ├── std::length_error
│   └── std::out_of_range
├── std::runtime_error        
│   ├── std::range_error
│   ├── std::overflow_error
│   └── std::underflow_error
├── std::bad_alloc           
├── std::bad_cast            
├── std::bad_typeid
├── std::bad_optional_access  
├── std::bad_variant_access
└── std::bad_function_call
```

