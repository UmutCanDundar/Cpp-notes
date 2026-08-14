# C++ Exception Guide: What Throws, What Doesn't, and When to Mark `noexcept`

A practical reference for deciding whether a function you write can be safely marked `noexcept`.

## Table of Contents

- [The Core Rule](#the-core-rule)
- [Part 1: Operations That Never Throw](#part-1-operations-that-never-throw)
- [Part 2: Operations That Can Throw](#part-2-operations-that-can-throw)
- [Part 3: Standard Library — Function by Function](#part-3-standard-library--function-by-function)
- [Part 4: Special Member Functions — Should They Be `noexcept`?](#part-4-special-member-functions--should-they-be-noexcept)
- [Part 5: Decision Checklist](#part-5-decision-checklist)
- [Part 6: What Happens If a `noexcept` Function Throws Anyway](#part-6-what-happens-if-a-noexcept-function-throws-anyway)
- [References](#references)

---

## The Core Rule

> **A function can only be honestly marked `noexcept` if *every* operation inside it is either `noexcept` itself, or its exceptions are caught and handled internally.**

`noexcept` is not a promise you can casually attach — the compiler doesn't verify it for you at compile time (no error if you lie), but lying is catastrophic at runtime (see [Part 6](#part-6-what-happens-if-a-noexcept-function-throws-anyway)). So the analysis has to be manual and conservative: trace every subexpression, every function call, every implicit conversion, every temporary's constructor/destructor.

---

## Part 1: Operations That Never Throw

These are safe to build a `noexcept` function around.

### Built-in / fundamental operations
- Arithmetic on built-in types: `+ - * / % ++ -- & | ^ ~ << >> && || !` on `int`, `double`, `bool`, pointers, etc.
  - **Caveat:** integer division/modulo by zero and signed overflow are **UB**, not exceptions — `noexcept` doesn't protect you from UB, it only concerns the *exception mechanism*.
- Comparison operators (`== != < > <= >=`) on built-in types.
- Pointer dereference, pointer arithmetic (assuming no UB).
- Array subscript `operator[]` on built-in arrays.
- `sizeof`, `alignof`, `typeid` on a non-polymorphic type or a dereferenced non-null polymorphic pointer used as an lvalue (see caveats in Part 2 for the null-pointer case).
- `static_cast`, `const_cast`, `reinterpret_cast` — never throw.
- Reading/writing a local variable, a reference, a plain member variable.

### Standard library — always `noexcept`
- All **move constructors and move assignment operators** of standard containers and `std::string` (as long as the allocator's move operations don't throw, which is the default case).
- `std::swap` for built-in types and for standard containers.
- `std::vector::size()`, `empty()`, `capacity()`, `begin()`, `end()`, `data()` — pure accessors.
- `std::unique_ptr` operations: constructor from raw pointer, `release()`, `reset()`, `get()`, move ops, destructor.
- `std::shared_ptr` copy constructor, destructor, `get()`, `use_count()`.
- All of `<type_traits>` (`std::is_same`, `std::enable_if`, etc.) — compile-time only, nothing to throw.
- `std::numeric_limits<T>::max()`, `min()`, etc.
- `std::optional`'s `has_value()`, `operator bool()`, and `operator*()` (unchecked access — UB if empty, not a throw).
- `std::atomic` load/store/exchange operations.
- Destructors of virtually all standard library types (see Part 4 — destructors are implicitly `noexcept` unless you go out of your way to change that).
- `std::string::size()`, `length()`, `empty()`, `c_str()`, `data()`, `clear()` (does not throw — releases capacity or not, but doesn't allocate).
- `std::span`, `std::string_view` — essentially all operations (they don't own memory, nothing to allocate).

### Language guarantees
- `noexcept(expr)` operator itself never throws — it's evaluated at compile time.
- Lambda captures by value/reference of types whose copy/move constructors are `noexcept`.

---

## Part 2: Operations That Can Throw

Treat every one of these as a "throw until proven otherwise" — if your function contains one of these unguarded, it is **not safe to mark `noexcept`** unless you wrap it in a `try/catch`.

### Memory allocation
- `new` (unless using `new(std::nothrow)`) — throws `std::bad_alloc` on failure.
- Any container operation that may allocate: `std::vector::push_back`, `emplace_back`, `reserve`, `resize`, `insert`; `std::string::append`, `+=`, `reserve`; `std::map`/`std::set`/`std::unordered_map` insertions.
  - **Exception (pun intended):** `push_back`/`emplace_back` *can* be `noexcept` in specific scenarios only if you're moving from a `noexcept` move constructor **and** no reallocation happens — but you cannot generally count on this from the caller's side, so treat these as throwing.
- `std::make_unique`, `std::make_shared` — throw `std::bad_alloc` (or exceptions from T's constructor).

### Element access with bounds checking
- `std::vector::at()`, `std::string::at()`, `std::array::at()`, `std::map::at()` — throw `std::out_of_range` (or in the map case, `std::out_of_range` too).
- `std::any_cast` on the wrong type — throws `std::bad_any_cast`.
- `std::get<T>(variant)` on the wrong alternative — throws `std::bad_variant_access`.
- `std::optional::value()` on an empty optional — throws `std::bad_optional_access`.
  - Contrast with `operator*()` on `std::optional`, which is unchecked/UB, not throwing.
- `std::stoi`, `std::stol`, `std::stod`, etc. — throw `std::invalid_argument` or `std::out_of_range`.

### Casts and RTTI
- `dynamic_cast<T&>` (reference form) — throws `std::bad_cast` if the cast fails.
  - The pointer form `dynamic_cast<T*>` does **not** throw — it returns `nullptr` on failure instead.
- `typeid(*ptr)` where `ptr` is `nullptr` and the pointed-to type is polymorphic — throws `std::bad_typeid`.

### Concurrency
- `std::thread::join()`, `std::thread::detach()` — can throw `std::system_error`.
- `std::mutex::lock()` — can throw `std::system_error` (though rare in practice).
- `std::future::get()` — rethrows whatever exception was stored in the shared state, or throws `std::future_error`.
- `std::promise::set_value()` — can throw `std::future_error`.

### I/O and filesystem
- `std::filesystem` functions **without** an `error_code&` output parameter throw `std::filesystem::filesystem_error` on failure. The overloads *with* an `error_code&` parameter do not throw.
- `std::fstream` operations throw only if you've explicitly enabled exceptions via `stream.exceptions(...)` — by default, iostreams set failure flags instead of throwing.
- `std::regex` construction — throws `std::regex_error` on an invalid pattern.

### Locale and formatting
- `std::locale` construction with an unknown locale name — throws `std::runtime_error`.
- `std::format` (C++20) — throws `std::format_error` on a bad format string.

### User-defined code
- Any call to a user-defined function/constructor/operator that is not itself marked (and truthfully) `noexcept`.
- Any expression involving an object whose copy constructor may throw (e.g., copying a `std::string`, `std::vector`, or any type that allocates).
- Re-throwing (`throw;`) inside a catch block.

---

## Part 3: Standard Library — Function by Function

| Function / Operation | Throws? | Notes |
|---|---|---|
| `std::vector::operator[]` | No (UB if out of range) | vs. `.at()` which throws |
| `std::vector::at()` | Yes — `std::out_of_range` | |
| `std::vector::push_back` | Yes — `std::bad_alloc`, or T's ctor | May be conditionally noexcept internally, but callers can't rely on it |
| `std::vector::pop_back` | No | On non-empty vector; UB if empty |
| `std::vector::clear()` | No | |
| `std::string::operator[]` | No (UB if out of range, except `s[s.size()]` returns null terminator) | |
| `std::string::at()` | Yes — `std::out_of_range` | |
| `std::map::operator[]` | Yes — `std::bad_alloc` possible (inserts if key absent) | |
| `std::map::at()` | Yes — `std::out_of_range` | |
| `std::map::find()` | No | |
| `std::unique_ptr` (ctor, dtor, move, `reset`, `release`) | No | |
| `std::shared_ptr` (copy ctor) | No | Ref count increment, not allocation |
| `std::make_shared` | Yes — `std::bad_alloc` | |
| `dynamic_cast<T&>` | Yes — `std::bad_cast` | |
| `dynamic_cast<T*>` | No | Returns `nullptr` instead |
| `std::stoi` / `std::stod` etc. | Yes | |
| `new` | Yes — `std::bad_alloc` | Unless `new(std::nothrow)` |
| `new(std::nothrow)` | No | Returns `nullptr` on failure |
| `delete` | No (should never throw — see Part 4) | |
| `std::thread::join()` | Yes — `std::system_error` | |
| `std::mutex::lock()` | Yes (rare) — `std::system_error` | |
| `std::mutex::unlock()` | No | |
| `std::optional::operator*` | No (UB if empty) | |
| `std::optional::value()` | Yes — `std::bad_optional_access` | |
| `std::variant::get<T>` (free function `std::get`) | Yes — `std::bad_variant_access` | |
| `std::any_cast` | Yes — `std::bad_any_cast` | |
| `std::filesystem::*` (no `error_code` overload) | Yes | |
| `std::filesystem::*` (with `error_code&`) | No | |
| Move constructors of standard containers | No | Assuming default allocator |
| Copy constructors of standard containers | Yes | May allocate |
| `std::swap` (containers, built-ins) | No | |

---

## Part 4: Special Member Functions — Should They Be `noexcept`?

| Member Function | Recommendation |
|---|---|
| **Destructor** | **Always effectively `noexcept` — never let one throw.** Destructors are implicitly `noexcept(true)` unless a base/member destructor is explicitly `noexcept(false)`. If a destructor throws while another exception is already propagating (e.g., during stack unwinding), `std::terminate` is called. Never write a throwing destructor. |
| **Move constructor** | **Mark `noexcept` whenever genuinely possible.** This matters enormously in practice: `std::vector` will only use your move constructor during reallocation if it is `noexcept` (verified via `std::move_if_noexcept`); otherwise it falls back to the (slower, but exception-safe) copy constructor. A throwing move constructor silently costs you performance everywhere `std::vector` and friends are used. |
| **Move assignment operator** | Same reasoning as move constructor — mark `noexcept` if it truly can't throw (typically: only pointer/handle swapping, no allocation). |
| **Swap function** (member or free `swap`) | **Should always be `noexcept`.** It's a foundational building block for the strong exception-safety guarantee (copy-and-swap idiom) and for `std::swap`-based algorithms. If your `swap` can throw, you undermine every exception-safe pattern built on top of it. |
| **Copy constructor / copy assignment** | Usually **not** `noexcept` if the type owns a resource (allocates memory, opens a file, etc.) — copying such a resource can fail. Only mark `noexcept` if you truly never allocate/acquire resources in the copy (e.g., a plain aggregate of trivial types). |
| **Default constructor** | `noexcept` if it doesn't allocate or acquire resources; not `noexcept` if it does (e.g., default-constructing a `std::vector` *is* `noexcept`, but default-constructing a type that opens a file is not). |
| **Hash function (`operator()` for `std::hash` specializations)** | Should be `noexcept` — required for use in `unordered_map`/`unordered_set` without silent overhead. |
| **Comparison operators (`==`, `<=>`, etc.)** | `noexcept` if comparing the underlying members doesn't throw (usually true for value types without heap-allocating members involved in the comparison logic itself, though comparing `std::string`s, for instance, does not throw). |

---

## Part 5: Decision Checklist

Before marking your function `noexcept`, walk through this:

1. **Does it call `new`, or any container method that can allocate (`push_back`, `insert`, `resize`, `reserve`, string concatenation, etc.)?** → Not `noexcept`, unless wrapped in try/catch with a sensible fallback.
2. **Does it call any function without a `noexcept` guarantee (including virtual functions, function pointers, `std::function`, or template parameters of unknown type)?** → Not `noexcept`, unless you can *prove* (e.g., via `static_assert(noexcept(...))`) that the actual instantiations used are noexcept.
3. **Does it use `.at()`, `dynamic_cast<T&>`, `std::get<T>`, `.value()` on optional/expected, or `std::stoi`-family functions?** → Not `noexcept`.
4. **Does it lock a mutex, join a thread, or otherwise touch the OS/concurrency primitives?** → Not `noexcept`.
5. **Is it a destructor, a swap, or a move operation that only manipulates pointers/handles/trivial members (no allocation, no throwing sub-operations)?** → Safe to mark `noexcept`.
6. **Does it only perform arithmetic, comparisons, and access on already-owned, already-initialized data (no allocation, no bounds-checked access)?** → Safe to mark `noexcept`.
7. **If your function is a template, do you actually know the throwing behavior of `T`'s operations at every instantiation?** → If not, use a conditional `noexcept(noexcept(...))` expression rather than an unconditional guarantee. Example:
   ```cpp
   template <typename T>
   void swap_impl(T& a, T& b) noexcept(std::is_nothrow_move_constructible_v<T> &&
                                        std::is_nothrow_move_assignable_v<T>) {
       T tmp = std::move(a);
       a = std::move(b);
       b = std::move(tmp);
   }
   ```

**Rule of thumb:** if you have to think for more than a few seconds about whether something *might* throw, don't mark it `noexcept` — the downside of a wrongly-marked `noexcept` function (an unrecoverable `std::terminate`) is far worse than the downside of missing out on a minor optimization opportunity.

---

## Part 6: What Happens If a `noexcept` Function Throws Anyway

If an exception tries to propagate out of a function marked `noexcept`, the C++ runtime calls **`std::terminate()`** immediately. This:

- Does **not** run stack unwinding in the normal sense — it's implementation-defined whether stack unwinding happens at all before `terminate` is invoked (in practice, most implementations do *not* fully unwind, meaning destructors of intervening objects may or may not run).
- Cannot be caught by any `try/catch`, no matter how it's nested — a `noexcept` boundary is a hard stop.
- By default calls `std::abort()`, killing the process.

This is why `noexcept` should be treated as a **hard contract**, not a hint — getting it wrong doesn't produce a "handled exception with extra steps," it produces an unrecoverable crash, often with no useful diagnostic about *why* the exception was thrown in the first place.

---

