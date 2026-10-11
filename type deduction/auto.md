# `auto` Cheat Sheet

What `auto`, `const auto`, `auto&`, `const auto&`, `auto&&`, `auto*` and `decltype(auto)` deduce for variables, objects, member access, function calls, operators, lambdas and literals — plus pointers, braces, return types, parameters, range-for, structured bindings and `template<auto>`.

---

## The rules

Apart from `decltype(auto)`, `auto` uses the **template argument deduction** rules: `auto x = e;` deduces exactly like `template<class T> void f(T)` called with `e`.

| Declaration | Deduces like | What happens to the initializer's type | Binds to rvalues? |
|---|---|---|---|
| `auto x = e;` | `f(T)` | Drops the reference and top-level `const`/`volatile`. Arrays decay to pointers, functions to function pointers. | yes (copy or move) |
| `const auto x = e;` | `f(T)` + const | Same as `auto`, then adds `const`. | yes |
| `auto& x = e;` | `f(T&)` | Drops only the reference. `const`/`volatile` are kept, arrays and functions do **not** decay. | only if the deduced type is `const` (e.g. a const class prvalue) — otherwise **error** |
| `const auto& x = e;` | `f(const T&)` | Adds `const`. Accepts anything and extends the lifetime of a temporary. | yes |
| `auto&& x = e;` | `f(T&&)` — forwarding reference | lvalue of type `U` → `U&` (const kept); rvalue of type `U` → `U&&`. | yes |
| `auto* x = e;` | `f(T*)` | `e` must be a pointer (arrays and functions decay to one). Pointee `const` is kept, the pointer's own top-level `const` is dropped. | — |
| `decltype(auto) x = e;` | `decltype(e) x = e;` | Uses the two `decltype` rules: unparenthesized name / member access → declared type; anything else → by value category (`T`, `T&`, `T&&`). | depends |
| `auto x{e};` | — | C++17: the type of `e` (exactly one element allowed). | yes |
| `auto x = {e1, e2};` | — | `std::initializer_list<T>`; all elements must have the same type. | yes |

`auto` never deduces a reference by itself: a reference appears only when you write `&` / `&&`, or when `decltype(auto)` is applied to a lvalue / xvalue that is not a plain name.

---

## Which one should I write?

| Situation | Write | Why |
|---|---|---|
| Read-only access, no copy | `const auto&` | Binds to lvalues and rvalues, never copies. |
| Modify the original in place | `auto&` | Alias to the object. Refuses rvalues. Keeps constness (a const object gives `const T&`). |
| Cheap value (int, double, pointer, iterator) or you want an independent copy | `auto` | Decays and drops cv/ref; copies (or moves) the value. |
| Generic code: accept lvalue or rvalue and keep its category; also the safe form for proxy elements such as `vector<bool>` | `auto&&` | Forwarding reference. |
| The initializer must be a pointer and you want that visible | `auto*` | Does not compile for non-pointers; same result as `auto` for pointers. |
| Store / return **exactly** what an expression yields (wrappers, forwarding returns) | `decltype(auto)` | Keeps references and value category. Watch the parentheses. |

---

## Setup

```cpp
struct Nec { int x; int& r; static int y; };
struct K { int q; };
struct S { int a; const int b; int& c; };

int a{}; int g{};
Nec n{1, a};
const Nec cn{1, a};

int i = 0; const int ci = 0; int& ref = i; int&& rr = 5; volatile int vi = 0;
int* ptr = &i; int arr[3];
const int* cp = &i;          // pointer to const int
int* const pc = &i;          // const pointer to int
const int* const cpc = &i;   // const pointer to const int

int  f(int);
int& fr();                   // returns lvalue
int&& fx();                  // returns xvalue
const int fc();              // returns const prvalue
const K mk();                // returns const class prvalue
auto l = [](int v) { return v; };
using std::move;
```

`closure` in the tables is the unnamed class the compiler generates for a lambda.

---

## 1. Plain variables

| Expression | Category | `auto` | `const auto` | `auto&` | `const auto&` | `auto&&` | `decltype(auto)` |
|---|---|---|---|---|---|---|---|
| `i` | lvalue | `int` | `const int` | `int&` | `const int&` | `int&` | `int` |
| `ci` | lvalue | `int` | `const int` | `const int&` | `const int&` | `const int&` | `const int` |
| `ref` | lvalue | `int` | `const int` | `int&` | `const int&` | `int&` | `int&` |
| `rr` | lvalue | `int` | `const int` | `int&` | `const int&` | `int&` | *error* |
| `ptr` | lvalue | `int*` | `int* const` | `int*&` | `int* const&` | `int*&` | `int*` |
| `arr` | lvalue | `int*` | `int* const` | `int(&)[3]` | `const int(&)[3]` | `int(&)[3]` | *error* |
| `cp` | lvalue | `const int*` | `const int* const` | `const int*&` | `const int* const&` | `const int*&` | `const int*` |
| `pc` | lvalue | `int*` | `int* const` | `int* const&` | `int* const&` | `int* const&` | `int* const` |
| `cpc` | lvalue | `const int*` | `const int* const` | `const int* const&` | `const int* const&` | `const int* const&` | `const int* const` |
| `vi` | lvalue | `int` | `const int` | `volatile int&` | `const volatile int&` | `volatile int&` | `volatile int` |

- `ci`: `auto&` deduces `const int&` — `auto&` is *not* "non-const reference"; constness comes from the initializer.
- `rr` is declared `int&&`, but as an expression it is a name, hence an lvalue → `auto&&` deduces `int&`. `decltype(auto) v = rr;` is an error: it means `int&& v = rr;`, which cannot bind an lvalue.
- `arr`: `auto` decays to `int*`; `auto&`, `const auto&`, `auto&&` keep the array type. `decltype(auto) v = arr;` is an error (an array cannot be copy-initialized from an array).
- `vi`: `auto` drops `volatile`; `auto&` keeps it.
- `cp`, `pc`, `cpc`: `auto` drops only the **top-level** `const` (the pointer's own), never the pointee's (see section 8).

## 2. The object itself

| Expression | Category | `auto` | `const auto` | `auto&` | `const auto&` | `auto&&` | `decltype(auto)` |
|---|---|---|---|---|---|---|---|
| `n` | lvalue | `Nec` | `const Nec` | `Nec&` | `const Nec&` | `Nec&` | `Nec` |
| `cn` | lvalue | `Nec` | `const Nec` | `const Nec&` | `const Nec&` | `const Nec&` | `const Nec` |
| `Nec{1,a}` | prvalue | `Nec` | `const Nec` | *error* | `const Nec&` | `Nec&&` | `Nec` |
| `move(n)` | xvalue | `Nec` | `const Nec` | *error* | `const Nec&` | `Nec&&` | `Nec&&` |

- `auto&` on a prvalue or xvalue is an error: a non-const lvalue reference cannot bind to a temporary.
- `auto&&` binds the temporary and extends its lifetime. `decltype(auto) v = move(n);` gives `Nec&&` — a reference into `n`, no new object.
- A **const class prvalue** behaves differently from a scalar one (`const K mk();`):

| Declaration | Deduced |
|---|---|
| `auto& v = mk();` | `const K&` — binds, because `T` is deduced as `const K` |
| `auto v = mk();` | `K` |
| `auto&& v = mk();` | `const K&&` |
| `decltype(auto) v = mk();` | `const K` — class prvalues keep their `const`; scalar prvalues do not (`decltype(auto) v = fc();` gives plain `int`) |

## 3. Member access

| Expression | Category | `auto` | `const auto` | `auto&` | `const auto&` | `auto&&` | `decltype(auto)` |
|---|---|---|---|---|---|---|---|
| `n.x` | lvalue | `int` | `const int` | `int&` | `const int&` | `int&` | `int` |
| `n.r` | lvalue | `int` | `const int` | `int&` | `const int&` | `int&` | `int&` |
| `n.y` | lvalue | `int` | `const int` | `int&` | `const int&` | `int&` | `int` |
| `cn.x` | lvalue | `int` | `const int` | `const int&` | `const int&` | `const int&` | `int` |
| `Nec{1,a}.x` | xvalue | `int` | `const int` | *error* | `const int&` | `int&&` | `int` |
| `Nec{1,a}.r` | lvalue | `int` | `const int` | `int&` | `const int&` | `int&` | `int&` |
| `move(n).x` | xvalue | `int` | `const int` | *error* | `const int&` | `int&&` | `int` |

- Same two rules as `decltype(e)` / `decltype((e))` for `decltype(auto)`: `n.x` is an unparenthesized member access, so it yields the **declared type** `int`; the `const` of the object is lost (`cn.x` → `int`), while `auto&` keeps it (`const int&`).
- Reference member `n.r`: always an lvalue, so `auto` copies the `int`, `auto&` / `auto&&` alias `a`, `decltype(auto)` gives `int&`.
- `Nec{1,a}.x` is an xvalue: `auto&&` gives `int&&` and keeps the temporary alive; `auto&` is an error.

## 4. Functions

| Expression | Category | `auto` | `const auto` | `auto&` | `const auto&` | `auto&&` | `decltype(auto)` |
|---|---|---|---|---|---|---|---|
| `f` | lvalue | `int(*)(int)` | `int(* const)(int)` | `int(&)(int)` | `int(&)(int)` | `int(&)(int)` | *error* |
| `&f` | prvalue | `int(*)(int)` | `int(* const)(int)` | *error* | `int(* const&)(int)` | `int(*&&)(int)` | `int(*)(int)` |
| `f(1)` | prvalue | `int` | `const int` | *error* | `const int&` | `int&&` | `int` |
| `fr()` | lvalue | `int` | `const int` | `int&` | `const int&` | `int&` | `int&` |
| `fx()` | xvalue | `int` | `const int` | *error* | `const int&` | `int&&` | `int&&` |
| `fc()` | prvalue | `int` | `const int` | *error* | `const int&` | `int&&` | `int` |

- A function name is an lvalue of function type: `auto` (and `const auto`) decay to a function pointer, `auto&` / `const auto&` / `auto&&` keep the function reference, `decltype(auto) v = f;` is an error (a function type cannot be an object).
- A call yields its return type: `int&` → lvalue (`auto` copies, `auto&` aliases), `int&&` → xvalue, `const int` → scalar prvalue whose `const` is discarded.

## 5. Operators

| Expression | Category | `auto` | `const auto` | `auto&` | `const auto&` | `auto&&` | `decltype(auto)` |
|---|---|---|---|---|---|---|---|
| `i + 3` | prvalue | `int` | `const int` | *error* | `const int&` | `int&&` | `int` |
| `++i` | lvalue | `int` | `const int` | `int&` | `const int&` | `int&` | `int&` |
| `i++` | prvalue | `int` | `const int` | *error* | `const int&` | `int&&` | `int` |
| `i = 5` | lvalue | `int` | `const int` | `int&` | `const int&` | `int&` | `int&` |
| `*ptr` | lvalue | `int` | `const int` | `int&` | `const int&` | `int&` | `int&` |
| `&i` | prvalue | `int*` | `int* const` | *error* | `int* const&` | `int*&&` | `int*` |
| `arr[0]` | lvalue | `int` | `const int` | `int&` | `const int&` | `int&` | `int&` |
| `i > 0` | prvalue | `bool` | `const bool` | *error* | `const bool&` | `bool&&` | `bool` |

- `++i` and `i = 5` return lvalues; `i++` returns a prvalue copy; `*ptr` and `arr[0]` are lvalues; `&i` is a prvalue pointer.
- Comparison yields a `bool` prvalue.

## 6. Lambdas

| Expression | Category | `auto` | `const auto` | `auto&` | `const auto&` | `auto&&` | `decltype(auto)` |
|---|---|---|---|---|---|---|---|
| `l` | lvalue | `closure` | `const closure` | `closure&` | `const closure&` | `closure&` | `closure` |
| `l(1)` | prvalue | `int` | `const int` | *error* | `const int&` | `int&&` | `int` |
| `[](int v){ return v; }` | prvalue | `closure` | `const closure` | *error* | `const closure&` | `closure&&` | `closure` |

- `auto f = l;` makes a **copy** of the closure (including its captures); use `auto&` / `const auto&` to avoid it.
- A lambda expression is a prvalue of its closure type, so `auto&` is an error.

## 7. Literals

| Expression | Category | `auto` | `const auto` | `auto&` | `const auto&` | `auto&&` | `decltype(auto)` |
|---|---|---|---|---|---|---|---|
| `5` | prvalue | `int` | `const int` | *error* | `const int&` | `int&&` | `int` |
| `3.14` | prvalue | `double` | `const double` | *error* | `const double&` | `double&&` | `double` |
| `'c'` | prvalue | `char` | `const char` | *error* | `const char&` | `char&&` | `char` |
| `"hi"` | lvalue | `const char*` | `const char* const` | `const char(&)[3]` | `const char(&)[3]` | `const char(&)[3]` | `const char(&)[3]` |
| `nullptr` | prvalue | `std::nullptr_t` | `std::nullptr_t` | *error* | `std::nullptr_t&` | `std::nullptr_t&&` | `std::nullptr_t` |
| `true` | prvalue | `bool` | `const bool` | *error* | `const bool&` | `bool&&` | `bool` |

- A string literal is an **lvalue** of type `const char[3]`: `auto` decays it to `const char*`, `auto&` keeps the array, `decltype(auto)` gives `const char(&)[3]`.
- All other literals are prvalues, so `auto&` fails; `const auto&` and `auto&&` extend the temporary.
- `nullptr` is `std::nullptr_t`, not a pointer type, so `auto*` does not accept it.

---

## 8. Pointers: the `auto*` family

Initializers are pointers (`ptr`, `cp`, `pc`, `cpc`), arrays and functions (which decay), or non-pointers (error).

| Initializer | `auto*` | `auto* const` | `const auto*` | `auto*&` | `auto**` |
|---|---|---|---|---|---|
| `ptr` | `int*` | `int* const` | `const int*` | `int*&` | *error* |
| `cp` | `const int*` | `const int* const` | `const int*` | `const int*&` | *error* |
| `pc` | `int*` | `int* const` | `const int*` | *error* | *error* |
| `cpc` | `const int*` | `const int* const` | `const int*` | *error* | *error* |
| `&i` | `int*` | `int* const` | `const int*` | *error* | *error* |
| `&ci` | `const int*` | `const int* const` | `const int*` | *error* | *error* |
| `arr` | `int*` | `int* const` | `const int*` | *error* | *error* |
| `"hi"` | `const char*` | `const char* const` | `const char*` | *error* | *error* |
| `&ptr` | `int**` | `int** const` | `int* const*` | *error* | `int**` |
| `i` | *error* | *error* | *error* | *error* | *error* |
| `nullptr` | *error* | *error* | *error* | *error* | *error* |
| `f` | `int(*)(int)` | `int(* const)(int)` | *error* | *error* | *error* |

- `auto*` requires a pointer initializer: `i` and `nullptr` are errors. Arrays and functions decay first, so `auto* p = arr;` and `auto* p = f;` work.
- The pointer's own top-level `const` is dropped (`pc`, `cpc`); `auto* const` puts it back.
- `const auto*` makes the **pointee** `const`; when the initializer already points to `const` it still works (`auto` becomes `int`).
- `auto*&` binds a reference to the pointer variable itself, so it needs a non-const pointer lvalue (`ptr`, `cp`); `pc` is a const pointer and `&i` is a prvalue → error.
- For a pointer initializer, `auto` and `auto*` give the same type; `auto*` only documents intent and rejects non-pointers.

---

## 9. Braces and initializer lists

| Declaration | Type of `v` |
|---|---|
| `auto v = 1;` | `int` |
| `auto v = {1};` | `std::initializer_list<int>` |
| `auto v = {1, 2, 3};` | `std::initializer_list<int>` |
| `auto v = {i};` | `std::initializer_list<int>` |
| `auto v{1};` | `int` |
| `auto v{i};` | `int` |
| `auto v = {1, 2.0};` | *error* |
| `auto v = {};` | *error* |
| `auto v{1, 2};` | *error* |
| `const auto& v = {1, 2};` | `const std::initializer_list<int>&` |
| `auto&& v = {1, 2};` | `std::initializer_list<int>&&` |
| `decltype(auto) v = {1};` | *error* |
| `for (auto x : {1,2,3})` | `int` (type of `x`) |

- `auto v{1};` is `int` (C++17 rule); `auto v = {1};` is `std::initializer_list<int>`. The two look alike but differ.
- `auto v{1, 2};` is an error (direct-list-init needs exactly one element); `auto v = {1, 2.0};` is an error (elements must have the same type); `auto v = {};` is an error (no element to deduce from).
- `decltype(auto) v = {1};` is an error: a braced list is not an expression, so `decltype` has nothing to deduce from.
- In a range-for over `{1,2,3}`, the range is an `initializer_list<int>`, so `x` is `int`.

---

## 10. `decltype(auto)` and parentheses

Same two rules as `decltype(e)` vs `decltype((e))`: `decltype(auto) v = e;` behaves like `decltype(e)`, and `decltype(auto) v = (e);` like `decltype((e))`.

| Expression `e` | `decltype(auto) v = e;` | `decltype(auto) v = (e);` |
|---|---|---|
| `i` | `int` | `int&` |
| `ref` | `int&` | `int&` |
| `rr` | *error* | `int&` |
| `ci` | `const int` | `const int&` |
| `ptr` | `int*` | `int*&` |
| `arr` | *error* | `int(&)[3]` |
| `f` | *error* | `int(&)(int)` |
| `n.x` | `int` | `int&` |
| `n.r` | `int&` | `int&` |
| `cn.x` | `int` | `const int&` |
| `fr()` | `int&` | `int&` |
| `std::move(i)` | `int&&` | `int&&` |
| `Nec{1,a}` | `Nec` | `Nec` |

- `rr`, `arr` and `f` fail without parentheses (the declared type is `int&&`, an array, a function — none can be initialized from the lvalue name), but work with parentheses because the expression then yields `T&`.
- Parentheses turn a plain object name into a reference: `i` → `int`, `(i)` → `int&`. This is the root of the classic dangling-return bug in section 11.

---

## 11. Function return types

`return e;` inside a function whose return type is written with each form. (The type shown is the function's return type.)

| Body | `auto` | `auto&` | `const auto&` | `auto&&` | `decltype(auto)` |
|---|---|---|---|---|---|
| `return i;` | `int` | `int&` | `const int&` | `int&` | `int` |
| `return (i);` | `int` | `int&` | `const int&` | `int&` | `int&` |
| `return ref;` | `int` | `int&` | `const int&` | `int&` | `int&` |
| `return ci;` | `int` | `const int&` | `const int&` | `const int&` | `const int` |
| `return rr;` | `int` | `int&` | `const int&` | `int&` | *error* |
| `return std::move(i);` | `int` | *error* | `const int&` | `int&&` | `int&&` |
| `return n.x;` | `int` | `int&` | `const int&` | `int&` | `int` |
| `return (n.x);` | `int` | `int&` | `const int&` | `int&` | `int&` |
| `return n.r;` | `int` | `int&` | `const int&` | `int&` | `int&` |
| `return f(1);` | `int` | *error* | `const int&` | `int&&` | `int` |
| `return arr;` | `int*` | `int(&)[3]` | `const int(&)[3]` | `int(&)[3]` | *error* |
| `return 5;` | `int` | *error* | `const int&` | `int&&` | `int` |

- `auto f() { return ref; }` returns **by value** (a copy); only `auto&` / `decltype(auto)` return a reference.
- `decltype(auto) f() { int x = 0; return (x); }` returns `int&` to a local — **dangling**. `return x;` would be safe (`int`).
- `auto&& f() { return 5; }` and `auto&& f() { return f(1); }` return `int&&` to a temporary that dies at the `return` — dangling as well.
- `auto&` return from a prvalue (`return 5;`, `return f(1);`) does not compile.
- All `return` statements must deduce the same type (`if (b) return 1; return 2.0;` is an error), and `return {1};` is an error for `auto`.
- Lambda return types follow the same table:

| Lambda | Type returned by the call |
|---|---|
| `[]{ return i; }` | `int` |
| `[]{ return ref; }` | `int` |
| `[]{ return (i); }` | `int` |
| `[]() -> auto& { return i; }` | `int&` |
| `[]() -> const auto& { return i; }` | `const int&` |
| `[]() -> decltype(auto) { return i; }` | `int` |
| `[]() -> decltype(auto) { return (i); }` | `int&` |
| `[]() -> auto&& { return std::move(i); }` | `int&&` |

---

## 12. Function parameters (`auto` parameters and generic lambdas)

`void f(auto x)` is `template<class T> void f(T x)` (C++20; generic lambdas `[](auto x)` since C++14). Each column is the parameter's type for the given argument.

| Argument | `auto x` | `auto& x` | `const auto& x` | `auto&& x` | `auto* x` |
|---|---|---|---|---|---|
| `i` | `int` | `int&` | `const int&` | `int&` | *error* |
| `ci` | `int` | `const int&` | `const int&` | `const int&` | *error* |
| `ref` | `int` | `int&` | `const int&` | `int&` | *error* |
| `rr` | `int` | `int&` | `const int&` | `int&` | *error* |
| `5` | `int` | *error* | `const int&` | `int&&` | *error* |
| `std::move(i)` | `int` | *error* | `const int&` | `int&&` | *error* |
| `arr` | `int*` | `int(&)[3]` | `const int(&)[3]` | `int(&)[3]` | `int*` |
| `f` | `int(*)(int)` | `int(&)(int)` | `int(&)(int)` | `int(&)(int)` | `int(*)(int)` |
| `"hi"` | `const char*` | `const char(&)[3]` | `const char(&)[3]` | `const char(&)[3]` | `const char*` |
| `ptr` | `int*` | `int*&` | `int* const&` | `int*&` | `int*` |
| `cp` | `const int*` | `const int*&` | `const int* const&` | `const int*&` | `const int*` |
| `&i` | `int*` | *error* | `int* const&` | `int*&&` | `int*` |

- `auto&& x` is a **forwarding reference**, not an rvalue reference: lvalue argument → `T&`, rvalue argument → `T&&`.
- `auto x` receives a decayed copy: `arr` → `int*`, `f` → function pointer, `"hi"` → `const char*`.
- `auto& x` rejects rvalues (`5`, `std::move(i)`, `&i`); `const auto&` accepts everything.
- `auto* x` requires a pointer argument; arrays, functions and string literals decay to one.

---

## 13. Range-based `for`

`for (FORM x : container)`; the table shows the type of `x`.

| Container | `auto` | `const auto` | `auto&` | `const auto&` | `auto&&` |
|---|---|---|---|---|---|
| `std::vector<int> v` | `int` | `const int` | `int&` | `const int&` | `int&` |
| `const std::vector<int> cv` | `int` | `const int` | `const int&` | `const int&` | `const int&` |
| `std::vector<bool> vb` | `std::_Bit_reference` | `const std::_Bit_reference` | *error* | `const std::_Bit_reference&` | `std::_Bit_reference&&` |
| `const std::vector<bool> cvb` | `bool` | `const bool` | *error* | `const bool&` | `bool&&` |
| `std::map<int,int> m` | `std::pair<const int, int>` | `const std::pair<const int, int>` | `std::pair<const int, int>&` | `const std::pair<const int, int>&` | `std::pair<const int, int>&` |
| `const std::map<int,int> cm` | `std::pair<const int, int>` | `const std::pair<const int, int>` | `const std::pair<const int, int>&` | `const std::pair<const int, int>&` | `const std::pair<const int, int>&` |
| `int arr2[3]` | `int` | `const int` | `int&` | `const int&` | `int&` |

- `std::map` elements are `std::pair<const K, V>`: `auto` **copies** the pair every iteration, `const auto&` does not.
- `std::vector<bool>` yields a proxy object (`std::_Bit_reference` in libstdc++), not `bool&`: `auto&` is an error, `auto&&` is the form that works. Beware: `for (auto x : vb) x = false;` still **writes through to the vector** (verified), because the copied proxy refers to the vector's bit — unlike `for (auto x : vector<int>)`, where the copy is independent.
- A `const std::vector<bool>` yields plain `bool` prvalues.
- Writing the element type by hand in a map loop (`const std::pair<int,int>& p : m`) does not match `pair<const int,int>` and silently builds a temporary copy; prefer `const auto&`.

With structured bindings over `std::map<int,int>` (`decltype(k)` | `decltype(v)`):

| Loop | `decltype(k)` \| `decltype(v)` |
|---|---|
| `for (auto [k,v] : m)` | `const int` | `int` |
| `for (auto& [k,v] : m)` | `const int` | `int` |
| `for (const auto& [k,v] : m)` | `const int` | `const int` |
| `for (auto&& [k,v] : m)` | `const int` | `int` |

---

## 14. Structured bindings

`FORM [x, y, z] = obj;` with `struct S { int a; const int b; int& c; };`. The `auto` form is applied to a hidden variable `e` (`auto` → copy, `auto&` → reference, ...); `x`, `y`, `z` are aliases for `e`'s members. The table shows `decltype(x)` | `decltype(y)` | `decltype(z)`.

| Object | Form | `decltype(x)` \| `decltype(y)` \| `decltype(z)` |
|---|---|---|
| `s` | `auto` | `int` \| `const int` \| `int&` |
| `s` | `auto&` | `int` \| `const int` \| `int&` |
| `s` | `const auto&` | `const int` \| `const int` \| `int&` |
| `s` | `auto&&` | `int` \| `const int` \| `int&` |
| `cs` | `auto` | `int` \| `const int` \| `int&` |
| `cs` | `auto&` | `const int` \| `const int` \| `int&` |
| `cs` | `const auto&` | `const int` \| `const int` \| `int&` |
| `cs` | `auto&&` | `const int` \| `const int` \| `int&` |
| `S{1,2,g}` | `auto` | `int` \| `const int` \| `int&` |
| `S{1,2,g}` | `auto&` | *error* |
| `S{1,2,g}` | `const auto&` | `const int` \| `const int` \| `int&` |
| `S{1,2,g}` | `auto&&` | `int` \| `const int` \| `int&` |

- `decltype(name)` of a binding is the member's declared type plus the `const` of the hidden object `e`: `const auto&` or a const object turns `x` into `const int`.
- The reference member stays `int&` in every row: copying `S` copies the reference, `const` does not pass through it, and `z` still aliases `g`.
- `auto [x, y, z] = cs;` drops the `const` of `cs` (the hidden copy is a plain `S`), so `x` is `int`; `auto&` on a const object keeps it (`const int`).
- `auto& [x, y, z] = S{...};` is an error (non-const lvalue reference to a temporary).

---

## 15. `template<auto V>` (C++17)

The non-type parameter's type is deduced by value, like `auto x = V;` (decays, drops cv).

| Argument | `decltype(V)` |
|---|---|
| `5` | `int` |
| `5u` | `unsigned int` |
| `5L` | `long` |
| `'c'` | `char` |
| `true` | `bool` |
| `3.14` | `double` |
| `nullptr` | `std::nullptr_t` |
| `&f` | `int(*)(int)` |
| `&Nec::x` | `int Nec::*` |

---

## 16. Misc deductions

| Declaration | Type of `v` |
|---|---|
| `constexpr auto v = 5;` | `const int` |
| `auto v = true ? 1 : 2.0;` | `double` |
| `auto v = i + 1L;` | `long` |
| `auto v = 'a' + 1;` | `int` |
| `auto v = 1u - 2;` | `unsigned int` |
| `auto v = 5 / 2.0f;` | `float` |
| `auto v = i < 3;` | `bool` |
| `auto v = sizeof(int);` | `unsigned long` |
| `auto v = 1ull;` | `unsigned long long` |
| `auto v = 1.f;` | `float` |
| `auto v = u8"x";` | `const char8_t*` |
| `auto v = new int;` | `int*` |
| `auto v = *ptr;` | `int` |
| `auto& v = *ptr;` | `int&` |
| `auto v = 1, w = 2;` | `int` |
| `auto v = 1, w = 2.0;` | *error* |
| `auto v = []{};` | `closure` |
| `auto v = std::move(ci);` | `int` |
| `auto v = std::string("x");` | `std::string` |
| `auto v = vi + 0;` | `int` |

- `constexpr auto` gives `const T` (constexpr implies const on the variable).
- Arithmetic follows the usual conversions: `'a' + 1` is `int`, `1u - 2` is `unsigned int` (wraps), `i + 1L` is `long`, `5 / 2.0f` is `float`, `sizeof` is `unsigned long` (`size_t`).
- Several declarators in one declaration must all deduce the same type (`auto v = 1, w = 2.0;` is an error).
- `auto v = *ptr;` copies the `int`; `auto& v = *ptr;` aliases it.
- `auto v = []{};` stores a closure; each lambda expression has its own distinct closure type.

---

## Common pitfalls

1. **`auto` copies.** `auto x = vec;` or `auto s = obj.getRef();` copies the object. Use `auto&` / `const auto&` when you only need access.
2. **`auto x = vb[0]` on `vector<bool>`** is a proxy (`std::_Bit_reference`), not `bool`; it keeps referring to the vector and dangles if the vector dies or reallocates.
3. **`decltype(auto) f() { int x; return (x); }`** returns a dangling `int&`. Drop the parentheses.
4. **`auto&&` is not an rvalue reference** — it is a forwarding reference; a lvalue gives `T&`.
5. **`auto x = {1};`** is an `initializer_list<int>`, not an `int`; `auto x{1};` is `int`.
6. **`for (auto x : map)`** copies every `pair<const K, V>`; use `const auto&` (or `auto&` to modify values).
7. **`auto s = "text";`** is `const char*`, not `std::string` or `std::string_view`; use `auto& s = "text";` for the array or write the type.
8. **`const auto&` extends a temporary's lifetime** only for the reference itself; returning that reference from a function does not (section 11).
