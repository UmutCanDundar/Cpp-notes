# Template Deduction, Forwarding References & Reference Collapsing

A cheat sheet for what happens inside a function like this:

```cpp
template <typename T>
void foo(T&& val) {
    T x = std::move(val);   // ...and 8 other variants, see below
}
```

---

## Table of contents

1. [Reference collapsing](#1-reference-collapsing)
2. [How `T` is deduced](#2-how-t-is-deduced)
3. [What `move(val)`, `val` and `forward<T>(val)` actually are](#3-what-moveval-val-and-forwardtval-actually-are)
4. [The master table: 9 declarations × 3 cases](#4-the-master-table-9-declarations--3-cases)
5. [`decltype` of `val` and `x`](#5-decltype-of-val-and-x)
6. [Pitfalls](#6-pitfalls)
7. [Verify it yourself](#7-verify-it-yourself)

---

## 1. Reference collapsing

You cannot write a reference to a reference directly (`int& &&` is an error), but it appears through templates, `typedef`/`using`, and `decltype`. The compiler then collapses it:

| Written form | Collapses to | Rule of thumb |
|---|---|---|
| `T& &` | `T&` | |
| `T& &&` | `T&` | |
| `T&& &` | `T&` | **any `&` present → lvalue reference** |
| `T&& &&` | `T&&` | only `&& &&` stays an rvalue reference |

Applied to the template parameter `T`:

| `T` | `T&` | `T&&` | `T` (by value) |
|---|---|---|---|
| `string` | `string&` | `string&&` | `string` (a new object) |
| `string&` | `string&` | `string&` | `string&` (a reference, no new object) |
| `string&&` | `string&` | `string&&` | `string&&` (a reference, no new object) |

> `T&` is **always** an lvalue reference. `T&&` is an rvalue reference **only** if `T` is not an lvalue reference.

---

## 2. How `T` is deduced

### 2.1 Forwarding reference `T&&`

`T&&` in a function template parameter, where `T` is deduced for that function, is a **forwarding reference**. The rule:

- argument is an **lvalue** of type `U` → `T = U&`
- argument is an **rvalue** (prvalue or xvalue) of type `U` → `T = U`

`T` is **never** deduced as `U&&`.

| Call | Argument category | Deduced `T` | Type of `val` |
|---|---|---|---|
| `foo(s)` (`string s`) | lvalue | `string&` | `string&` |
| `foo(cs)` (`const string cs`) | const lvalue | `const string&` | `const string&` |
| `foo(string{"x"})` | prvalue | `string` | `string&&` |
| `foo("x"s)` | prvalue | `string` | `string&&` |
| `foo(move(s))` | xvalue | `string` | `string&&` |
| `foo(move(cs))` | const xvalue | `const string` | `const string&&` |
| `foo(a)` (`int a`) | lvalue | `int&` | `int&` |
| `foo(4)` | prvalue | `int` | `int&&` |
| `foo("abc")` | lvalue (string literal) | `const char(&)[4]` | `const char(&)[4]` |
| `foo<string&&>(string{})` | *explicit `T`* | `string&&` | `string&&` |
| `foo({1, 2})` | braced-init-list | ❌ cannot deduce `T` | — |

The only way to get `T = string&&` is to spell it explicitly. Deduction will never produce it.

### 2.2 Other parameter forms, for contrast

| Parameter | `foo(s)` | `foo(cs)` | `foo(string{})` | `foo(move(s))` |
|---|---|---|---|---|
| `T val` | `T = string` | `T = string` | `T = string` | `T = string` |
| `T& val` | `T = string` | `T = const string` | ❌ | ❌ |
| `const T& val` | `T = string` | `T = string` | `T = string` | `T = string` |
| `T&& val` | `T = string&` | `T = const string&` | `T = string` | `T = string` |

By-value `T` drops references and top-level `const`. Only `T&&` encodes the value category of the argument into `T`.

---

## 3. What `move(val)`, `val` and `forward<T>(val)` actually are

Inside `foo`, three different expressions can appear on the right-hand side:

| Expression | Type | Value category | Notes |
|---|---|---|---|
| `val` | `remove_reference_t<T>` | **always lvalue** | A named variable is an lvalue, even if its declared type is `string&&`. |
| `move(val)` | `remove_reference_t<T>&&` | **always xvalue** | Unconditional cast to rvalue. Moves nothing by itself. |
| `forward<T>(val)` | `T&&` | xvalue if `T` is not an lvalue ref, **lvalue** if `T = U&` | Conditional cast: restores the caller's value category. |

Concretely, per case:

| Case | `T` | `val` | `move(val)` | `forward<T>(val)` |
|---|---|---|---|---|
| **A** | `string` | lvalue | `string&&` xvalue | `string&&` xvalue |
| **B** | `string&` | lvalue | `string&&` xvalue | `string&` **lvalue** |
| **C** | `string&&` | lvalue | `string&&` xvalue | `string&&` xvalue |

Remember the binding rules the table below relies on:

- non-const lvalue reference `X&` binds to **lvalues only**
- rvalue reference `X&&` binds to **rvalues (xvalues/prvalues) only**
- a by-value object `X x = expr;` is **copy-constructed** from an lvalue and **move-constructed** from an rvalue

---

## 4. The master table: 9 declarations × 3 cases

The three cases:

| Case | `T` | How you get it | Type of `val` |
|---|---|---|---|
| **A** | `string` | `foo(string{..})`, `foo("x"s)`, `foo(move(s))` | `string&&` |
| **B** | `string&` | `foo(s)` | `string&` |
| **C** | `string&&` | only `foo<string&&>(...)` | `string&&` |

`Valid?` is the compile result. `Outcome` says what physically happens at runtime.

Terms used in `Outcome`:

- **move-construct**: a **new** `string` is created and steals the source's buffer (source is left valid but unspecified).
- **copy-construct**: a **new** `string` is created with its own copy.
- **bind**: `x` is only an **alias**. No new object, nothing is copied or moved.

### Case A: `T = string`, `val` is `string&&` (declared), an lvalue (as an expression)

| # | Declaration | Type of `x` | Right-hand side | RHS type / category | Valid? | Outcome |
|---|---|---|---|---|---|---|
| 1 | `T x = move(val);` | `string` | `move(val)` | `string&&` xvalue | ✅ | **move-construct** |
| 2 | `T x = val;` | `string` | `val` | `string` lvalue | ✅ | **copy-construct** (not a move, even though `val` is `string&&`!) |
| 3 | `T x = forward<T>(val);` | `string` | `forward<string>(val)` | `string&&` xvalue | ✅ | **move-construct** |
| 4 | `T& x = move(val);` | `string&` | `move(val)` | `string&&` xvalue | ❌ | non-const `&` cannot bind an rvalue |
| 5 | `T& x = val;` | `string&` | `val` | `string` lvalue | ✅ | **bind** to the object `val` refers to |
| 6 | `T& x = forward<T>(val);` | `string&` | `forward<string>(val)` | `string&&` xvalue | ❌ | non-const `&` cannot bind an rvalue |
| 7 | `T&& x = move(val);` | `string&&` | `move(val)` | `string&&` xvalue | ✅ | **bind** (no move happens) |
| 8 | `T&& x = val;` | `string&&` | `val` | `string` lvalue | ❌ | `&&` cannot bind an lvalue, and `val` is one |
| 9 | `T&& x = forward<T>(val);` | `string&&` | `forward<string>(val)` | `string&&` xvalue | ✅ | **bind** (no move happens) |

### Case B: `T = string&`, `val` is `string&`

| # | Declaration | Type of `x` | Right-hand side | RHS type / category | Valid? | Outcome |
|---|---|---|---|---|---|---|
| 1 | `T x = move(val);` | `string&` | `move(val)` | `string&&` xvalue | ❌ | `string&` cannot bind an rvalue |
| 2 | `T x = val;` | `string&` | `val` | `string` lvalue | ✅ | **bind** to caller's object |
| 3 | `T x = forward<T>(val);` | `string&` | `forward<string&>(val)` | `string&` lvalue | ✅ | **bind** to caller's object |
| 4 | `T& x = move(val);` | `string&` | `move(val)` | `string&&` xvalue | ❌ | `string&` cannot bind an rvalue |
| 5 | `T& x = val;` | `string&` | `val` | `string` lvalue | ✅ | **bind** |
| 6 | `T& x = forward<T>(val);` | `string&` | `forward<string&>(val)` | `string&` lvalue | ✅ | **bind** |
| 7 | `T&& x = move(val);` | `string&` (`& &&` → `&`) | `move(val)` | `string&&` xvalue | ❌ | collapsed to `string&`, cannot bind an rvalue |
| 8 | `T&& x = val;` | `string&` (`& &&` → `&`) | `val` | `string` lvalue | ✅ | **bind** |
| 9 | `T&& x = forward<T>(val);` | `string&` (`& &&` → `&`) | `forward<string&>(val)` | `string&` lvalue | ✅ | **bind** |

### Case C: `T = string&&` (explicit), `val` is `string&&`

| # | Declaration | Type of `x` | Right-hand side | RHS type / category | Valid? | Outcome |
|---|---|---|---|---|---|---|
| 1 | `T x = move(val);` | `string&&` | `move(val)` | `string&&` xvalue | ✅ | **bind** (nothing is moved, `x` is just a reference) |
| 2 | `T x = val;` | `string&&` | `val` | `string` lvalue | ❌ | `&&` cannot bind an lvalue |
| 3 | `T x = forward<T>(val);` | `string&&` | `forward<string&&>(val)` | `string&&` xvalue | ✅ | **bind** |
| 4 | `T& x = move(val);` | `string&` (`&& &` → `&`) | `move(val)` | `string&&` xvalue | ❌ | `string&` cannot bind an rvalue |
| 5 | `T& x = val;` | `string&` (`&& &` → `&`) | `val` | `string` lvalue | ✅ | **bind** |
| 6 | `T& x = forward<T>(val);` | `string&` (`&& &` → `&`) | `forward<string&&>(val)` | `string&&` xvalue | ❌ | `string&` cannot bind an rvalue |
| 7 | `T&& x = move(val);` | `string&&` | `move(val)` | `string&&` xvalue | ✅ | **bind** |
| 8 | `T&& x = val;` | `string&&` | `val` | `string` lvalue | ❌ | `&&` cannot bind an lvalue |
| 9 | `T&& x = forward<T>(val);` | `string&&` | `forward<string&&>(val)` | `string&&` xvalue | ✅ | **bind** |

### Compact summary

| # | Declaration | A `string` | B `string&` | C `string&&` |
|---|---|---|---|---|
| 1 | `T x = move(val);` | ✅ move | ❌ | ✅ bind |
| 2 | `T x = val;` | ✅ **copy** | ✅ bind | ❌ |
| 3 | `T x = forward<T>(val);` | ✅ move | ✅ bind | ✅ bind |
| 4 | `T& x = move(val);` | ❌ | ❌ | ❌ |
| 5 | `T& x = val;` | ✅ bind | ✅ bind | ✅ bind |
| 6 | `T& x = forward<T>(val);` | ❌ | ✅ bind | ❌ |
| 7 | `T&& x = move(val);` | ✅ bind | ❌ | ✅ bind |
| 8 | `T&& x = val;` | ❌ | ✅ bind | ❌ |
| 9 | `T&& x = forward<T>(val);` | ✅ bind | ✅ bind | ✅ bind |

Rows 3 and 9 compile in **every** case and preserve the caller's value category: that is perfect forwarding. Row 5 is the only other universally valid one, but it always yields an lvalue reference and never moves anything.

### Notes on `int`

The table is identical for `int` (`foo(a)` → B, `foo(4)` → A, `foo<int&&>(4)` → C). The only difference: "move-construct" and "copy-construct" are the same operation for `int`, so you cannot observe the difference.

### Lifetime

In case A with a temporary argument (`foo(string{"temp"})`), the temporary lives until the end of the full expression containing the call. Every `bind` row above is therefore safe inside `foo`. It would dangle only if `foo` returned or stored that reference beyond the call.

---

## 5. `decltype` of `val` and `x`

A named variable is always an lvalue expression, but `decltype` has two modes (see the first rule of `decltype`):

- `decltype(name)` → the **declared type**
- `decltype((name))` → type + value category of the **expression** (lvalue → `T&`)

| Expression | A (`string`) | B (`string&`) | C (`string&&`) |
|---|---|---|---|
| `decltype(val)` | `string&&` | `string&` | `string&&` |
| `decltype((val))` | `string&` | `string&` | `string&` |

For `x`, `decltype(x)` is the "Type of `x`" column in section 4, and `decltype((x))` is `string&` for every valid declaration, because `x` is always an lvalue expression.

Practical consequence:

```cpp
consume(x);                              // always passes an lvalue
consume(std::move(x));                   // always passes an rvalue
consume(std::forward<decltype(x)>(x));   // restores the category from x's declared type
```

---

## 6. Pitfalls

1. **`T x = val;` does not move.** In case A `val` is declared `string&&`, but as an expression it is an lvalue, so you get a copy. Use `std::forward<T>(val)` (or `std::move` if you really always want to move).
2. **`std::move(val)` on a forwarding reference is dangerous.** With an lvalue argument (case B), it either fails to compile (`T x = move(val)`) or, with `T x` / `auto x`, silently steals from the caller's object.
3. **`T&&` is not always an rvalue reference.** It is a forwarding reference when `T` is deduced. `T = string&` collapses it to `string&`.
4. **`is_rvalue_reference_v<T>` is only true in case C**, which deduction never produces. To ask "was the argument an lvalue?", use `is_lvalue_reference_v<T>`. You cannot tell prvalue from xvalue inside `foo`; both give `T = string`.
5. **Value category of an expression:** use double parentheses, e.g. `is_rvalue_reference_v<decltype((std::move(s)))>`, or the plain call form `decltype(std::move(s))` (a function call yields its return type, which already encodes the category).
6. **`T x` has three different meanings**: a new object (A), an lvalue reference (B), an rvalue reference (C). Never assume it is a copy.
7. **`foo({1, 2})` does not compile**: a braced-init-list has no type to deduce `T` from.

---

