# Perfect Forwarding

Perfect forwarding passes arguments to another function while preserving their value category (lvalue/rvalue) and const-ness. It's essential for writing generic wrapper functions (like factories) that behave exactly as if the caller called the target function directly.

## Forwarding (Universal) References

A forwarding reference is `T&&` where `T` is a deduced template parameter (or `auto&&`). It can bind to both lvalues and rvalues.

```cpp
template <typename T>
void wrapper(T&& arg) { /* arg is a forwarding reference */ }
```

## Forwarding Reference vs Rvalue Reference

`T&&` in a template is a forwarding reference only if `T` is deduced. `Widget&&` (concrete type) is always an rvalue reference, not forwarding.

```cpp
template <typename T> void f(T&& x);      // forwarding reference
void g(std::string&& x);                  // plain rvalue reference
```

## std::forward

`std::forward<T>(arg)` conditionally casts `arg` back to an rvalue only if `T` was deduced as a non-reference (i.e., the original argument was an rvalue).

```cpp
template <typename T>
void wrapper(T&& arg) {
    target(std::forward<T>(arg)); // preserves lvalue/rvalue-ness
}
```

## auto&& for Perfect Forwarding

`auto&&` behaves like a forwarding reference outside templates, useful in range-based for loops and lambdas.

```cpp
for (auto&& elem : container) { /* works for any value category */ }
```

## decltype(auto) and Perfect Returning

`decltype(auto)` deduces the return type exactly as the expression's type, preserving references — useful for forwarding return values.

```cpp
template <typename F, typename... Args>
decltype(auto) call(F&& f, Args&&... args) {
    return std::forward<F>(f)(std::forward<Args>(args)...);
}
```

## Deferred Perfect Returning

Sometimes you must store a result before returning it (e.g., after some other work). Using `decltype(auto)` on a named variable still requires care to avoid dangling references.

```cpp
decltype(auto) compute() {
    static int x = 5;
    return (x); // returns int&, deliberate
}
```

## Perfect Forwarding of Return Values

When a wrapper function returns the result of the wrapped call, forward it too, using `decltype(auto)` and `std::forward`, to preserve reference-ness.

```cpp
template <typename F, typename... Args>
decltype(auto) invoke(F&& f, Args&&... args) {
    return std::forward<F>(f)(std::forward<Args>(args)...);
}
```

## Perfect Forwarding in STL

STL uses perfect forwarding heavily, e.g. `emplace_back`, `make_unique`, `make_shared` forward constructor arguments without extra copies.

```cpp
std::vector<std::pair<int,std::string>> v;
v.emplace_back(1, "hello"); // args forwarded to pair constructor
```

## Typical Mistakes & Misconceptions

Forgetting `std::forward` and just using `arg` directly turns every forwarded argument into an lvalue, defeating the purpose. Also, applying `std::forward` more than once to the same variable is a bug.

```cpp
template <typename T>
void bad(T&& arg) {
    target(arg); // WRONG: always passes as lvalue
}
```

## Guidelines

- Use `T&&` with deduced `T` for forwarding references, never for a single fixed type.
- Always pair forwarding references with `std::forward`.
- Use variadic templates (`Args&&...`) for forwarding multiple arguments.
- Prefer `decltype(auto)` when the return type must preserve value category.
