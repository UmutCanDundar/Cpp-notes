# C++ Standard Attributes Reference

All standard `[[attribute]]`s, since which C++ version, what they apply to, and what they do.

| Attribute | Since | Applies to | Effect |
|---|---|---|---|
| `[[nodiscard]]` | C++17 | function, class/enum (applies to its return) | Warns if the return value is discarded (not stored/used) |
| `[[nodiscard("reason")]]` | C++20 | same as above | Same, warning message includes the reason string |
| `[[noreturn]]` | C++11 | function | Promises the function never returns normally (always throws, calls `std::terminate`/`exit`, infinite loops, etc.) — enables better optimization and missing-return diagnostics |
| `[[deprecated]]` | C++14 | function, class, enum, typedef, variable, namespace | Warns on use, marking the entity as deprecated |
| `[[deprecated("reason")]]` | C++14 | same as above | Same, warning includes the reason string |
| `[[maybe_unused]]` | C++17 | variable, function, parameter, class, enum, typedef | Suppresses "unused entity" warnings |
| `[[fallthrough]]` | C++17 | null statement (`;`) inside a `switch` | Documents intentional fallthrough between `case` labels, suppresses fallthrough warnings |
| `[[likely]]` | C++20 | statement (usually a branch of `if`/`switch`, or a label) | Hints to the optimizer that this path is the likely one |
| `[[unlikely]]` | C++20 | statement | Hints that this path is unlikely — used for error branches, edge cases |
| `[[no_unique_address]]` | C++20 | non-static data member | Allows the compiler to overlap this member's storage with another (typically for empty types), reducing struct size |
| `[[carries_dependency]]` | C++11 | function parameter, function return | Propagates memory-order dependency information for `memory_order_consume` (rarely used — `memory_order_consume` is effectively deprecated in practice) |
| `[[assume(expr)]]` | C++23 | statement | Tells the optimizer to assume `expr` is always true; `expr` is **not evaluated at runtime** — UB if it's actually false at that point |
| `[[indeterminate]]` | C++26 | variable declaration | Marks a variable as intentionally left indeterminate (opts out of default erasure of uninitialized values for security-hardened builds) |
| `[[assert]]` | proposed / not standard | — | Not currently a standard C++ attribute — don't confuse with `assert()` macro or `[[assume]]` |

## Notes on select attributes

**`[[nodiscard]]` on constructors** — applying it to a constructor warns if the constructed object is immediately discarded (e.g., `MyGuard();` instead of `MyGuard g;`), useful for RAII guard types.

**`[[likely]]`/`[[unlikely]]` placement** — goes on the statement itself, not the condition:
```cpp
if (ptr) [[likely]] {
    use(ptr);
} else [[unlikely]] {
    handle_error();
}
```

**`[[no_unique_address]]` MSVC caveat** — historically not honored correctly by MSVC even when accepted syntactically (fixed behavior requires opting in via a compiler flag on older MSVC versions); check current compiler docs if size reduction is required.

**`[[maybe_unused]]` vs `(void)x;`** — `[[maybe_unused]]` is the modern, self-documenting replacement for the old `(void)x;` unused-suppression idiom.

**Unknown/unrecognized attributes** — the standard requires compilers to ignore attributes they don't recognize rather than error, so vendor-specific attributes (`[[gnu::...]]`, `[[msvc::...]]`, `[[clang::...]]`) can coexist safely in portable code.

## Common non-standard (vendor) attributes worth knowing

| Attribute | Vendor | Effect |
|---|---|---|
| `[[gnu::always_inline]]` / `__attribute__((always_inline))` | GCC/Clang | Forces inlining regardless of optimizer heuristics |
| `[[gnu::noinline]]` | GCC/Clang | Prevents inlining |
| `[[gnu::pure]]` / `[[gnu::const]]` | GCC/Clang | Declares the function has no side effects (const = also ignores global state) — enables aggressive optimization |
| `[[gnu::hot]]` / `[[gnu::cold]]` | GCC/Clang | Marks a function as frequently/rarely executed for code layout optimization |
| `[[msvc::forceinline]]` | MSVC | Forces inlining |
| `[[clang::fallthrough]]` | Clang (pre-C++17) | Predecessor to standard `[[fallthrough]]` |
