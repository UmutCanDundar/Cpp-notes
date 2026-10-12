# `auto` vs `decltype` vs `decltype(auto)` — Overview

How the three deduction mechanisms differ on the same expression.

---

## Full comparison

Setup: `int i; const int ci; int& ref; int&& rr; int* ptr; int arr[3]; struct Nec { int x; int& r; }; Nec n; const Nec cn; int f(int); int& fr(); int&& fx();` and a lambda `l`.

| Expression | Category | `auto` | `auto&` | `auto&&` | `decltype(e)` | `decltype((e))` | `decltype(auto)` | `decltype(auto)` is the same as |
|---|---|---|---|---|---|---|---|---|
| `i` | lvalue | `int` | `int&` | `int&` | `int` | `int&` | `int` | `auto` |
| `(i)` | lvalue | `int` | `int&` | `int&` | `int&` | `int&` | `int&` | `auto&`, `auto&&` |
| `ci` | lvalue | `int` | `const int&` | `const int&` | `const int` | `const int&` | `const int` | none |
| `ref` | lvalue | `int` | `int&` | `int&` | `int&` | `int&` | `int&` | `auto&`, `auto&&` |
| `rr` | lvalue | `int` | `int&` | `int&` | `int&&` | `int&` | *error* | — (declaration fails) |
| `(rr)` | lvalue | `int` | `int&` | `int&` | `int&` | `int&` | `int&` | `auto&`, `auto&&` |
| `n.x` | lvalue | `int` | `int&` | `int&` | `int` | `int&` | `int` | `auto` |
| `(n.x)` | lvalue | `int` | `int&` | `int&` | `int&` | `int&` | `int&` | `auto&`, `auto&&` |
| `cn.x` | lvalue | `int` | `const int&` | `const int&` | `int` | `const int&` | `int` | `auto` |
| `(cn.x)` | lvalue | `int` | `const int&` | `const int&` | `const int&` | `const int&` | `const int&` | `auto&`, `auto&&` |
| `fr()` | lvalue | `int` | `int&` | `int&` | `int&` | `int&` | `int&` | `auto&`, `auto&&` |
| `fx()` | xvalue | `int` | *error* | `int&&` | `int&&` | `int&&` | `int&&` | `auto&&` |
| `f(1)` | prvalue | `int` | *error* | `int&&` | `int` | `int` | `int` | `auto` |
| `*ptr` | lvalue | `int` | `int&` | `int&` | `int&` | `int&` | `int&` | `auto&`, `auto&&` |
| `i + 3` | prvalue | `int` | *error* | `int&&` | `int` | `int` | `int` | `auto` |
| `++i` | lvalue | `int` | `int&` | `int&` | `int&` | `int&` | `int&` | `auto&`, `auto&&` |
| `move(n)` | xvalue | `Nec` | *error* | `Nec&&` | `Nec&&` | `Nec&&` | `Nec&&` | `auto&&` |
| `Nec{1,a}` | prvalue | `Nec` | *error* | `Nec&&` | `Nec` | `Nec` | `Nec` | `auto` |
| `arr` | lvalue | `int*` | `int(&)[3]` | `int(&)[3]` | `int[3]` | `int(&)[3]` | *error* | — (declaration fails) |
| `f` | lvalue | `int(*)(int)` | `int(&)(int)` | `int(&)(int)` | `int(int)` | `int(&)(int)` | *error* | — (declaration fails) |
| `l` | lvalue | `closure` | `closure&` | `closure&` | `closure` | `closure&` | `closure` | `auto` |
| `"hi"` | lvalue | `const char*` | `const char(&)[3]` | `const char(&)[3]` | `const char(&)[3]` | `const char(&)[3]` | `const char(&)[3]` | `auto&`, `auto&&` |
| `5` | prvalue | `int` | *error* | `int&&` | `int` | `int` | `int` | `auto` |

Totals over the 23 rows: `decltype(auto)` equals `auto` in **8**, `auto&` in **9**, `auto&&` in **11**, and `decltype(e)` in **20** (the other 3 fail to declare).

---

## `decltype(auto)` = `auto&&` with two exceptions

This one rule predicted `decltype(auto)` correctly in all 20 rows that compile:

| Exception | Effect | Example (`auto&&` vs `decltype(auto)`) |
|---|---|---|
| 1. The expression is a **prvalue** | plain value `T` instead of `T&&` | `f(1)`: `int&&` vs `int` |
| 2. The expression is a **bare name or bare member access** | declared type instead of a reference | `i`: `int&` vs `int` · `ci`: `const int&` vs `const int` · `n.x`: `int&` vs `int` |
| Otherwise | identical to `auto&&` | `(i)`, `fr()`, `*ptr`, `++i`: `int&` · `fx()`, `move(n)`: `T&&` |

Consequences worth remembering:

- `decltype(auto) x = i;` is a **copy** (`int`), not an alias. It equals `auto& x = i;` only when the expression is already an lvalue-returning one (`ref`, `fr()`, `*ptr`, `(i)`).
- `decltype(auto)` keeps `const` but loses the object's `const` on member access: `cn.x` gives `int` (declared type), while `(cn.x)` gives `const int&`.
- `auto&` rejects every prvalue and xvalue (*error* cells above); `auto&&` and `decltype(auto)` accept them.

---

## Pitfalls

| Code | Result | Note |
|---|---|---|
| `decltype(auto) f() { return i; }` | `int` | bare name: declared type, a copy |
| `decltype(auto) f() { return (i); }` | `int&` | parenthesized: lvalue → reference; **dangling** if `i` is a local |
| `auto f() { return (i); }` | `int` | `auto` ignores the parentheses |
| `const decltype(auto) v = i;` | *error* | `decltype(auto)` must be the only type specifier |
| `decltype(auto)& v = i;` | *error* | cannot add `&` |
| `decltype(auto) v = {1};` | *error* | a braced list is not an expression |

---
