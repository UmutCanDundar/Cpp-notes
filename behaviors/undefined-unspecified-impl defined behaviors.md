# C++ Undefined, Unspecified, and Implementation-Defined Behavior

A curated reference covering the different categories of "unpredictable" behavior in C++, with concrete code examples, explanations, and why the standard leaves them this way.

## Table of Contents

- [Terminology](#terminology)
- [Part 1: Undefined Behavior (UB)](#part-1-undefined-behavior-ub)
  - [Memory Access](#memory-access)
  - [Arithmetic](#arithmetic)
  - [Pointers and References](#pointers-and-references)
  - [Object Lifetime](#object-lifetime)
  - [Sequencing](#sequencing)
  - [Type Safety and Aliasing](#type-safety-and-aliasing)
  - [Concurrency](#concurrency)
  - [Templates and Generic Code](#templates-and-generic-code)
  - [Miscellaneous UB](#miscellaneous-ub)
- [Part 2: Unspecified Behavior](#part-2-unspecified-behavior)
- [Part 3: Implementation-Defined Behavior](#part-3-implementation-defined-behavior)
- [Part 4: Ill-Formed, No Diagnostic Required (IFNDR)](#part-4-ill-formed-no-diagnostic-required-ifndr)
- [How to Catch UB in Practice](#how-to-catch-ub-in-practice)
- [References](#references)

---

## Terminology

C++ (like C) defines several tiers of "the standard does not fully pin down what happens":

| Category | Meaning | Example |
|---|---|---|
| **Undefined Behavior (UB)** | Anything can happen — a crash, wrong output, or silently "correct-looking" output. The compiler is allowed to assume UB never occurs and optimize accordingly. | Dereferencing a null pointer |
| **Unspecified Behavior** | The standard allows multiple valid outcomes, but the implementation must pick one consistently (not necessarily documented). | Order of evaluation of function arguments |
| **Implementation-Defined Behavior** | Like unspecified behavior, but the implementation **must document** its choice. | Size of `int`, signedness of `char` |
| **Locale-Specific Behavior** | Depends on locale settings. | Behavior of `toupper()` |
| **Ill-Formed, No Diagnostic Required (IFNDR)** | The program violates a rule that compilers are not required to diagnose; if compiled, behavior is undefined. | Violating the One Definition Rule (ODR) |

The key danger of UB is that the compiler's optimizer is legally permitted to assume it **never** occurs. This means UB doesn't just cause "wrong output" — it can eliminate seemingly unrelated code, remove null checks, or produce time-travel-like effects where a check *after* the UB is optimized away because the compiler proved the UB path "can't happen."

---

## Part 1: Undefined Behavior (UB)

### Memory Access

#### 1. Reading an uninitialized variable
```cpp
int x;
std::cout << x; // UB: indeterminate value
```

#### 2. Out-of-bounds array/container access
```cpp
int arr[5];
arr[5] = 1;      // UB: one past the end, writing is worse
std::vector<int> v(3);
v[10] = 1;       // UB: operator[] does not bounds-check
```

#### 3. Buffer overflow via C-style functions
```cpp
char buf[4];
strcpy(buf, "too long"); // UB: writes past the buffer
```

#### 4. Use-after-free / dangling pointer dereference
```cpp
int* p = new int(5);
delete p;
std::cout << *p; // UB: use after free
```

#### 5. Double free
```cpp
int* p = new int(5);
delete p;
delete p; // UB
```

#### 6. Stack buffer overrun via `alloca` / VLAs (in GCC/Clang extensions) exceeding stack size
```cpp
void f(size_t n) {
    int arr[n]; // Not standard C++, but where supported: unbounded n -> UB (stack overflow)
}
```

### Arithmetic

#### 7. Signed integer overflow
```cpp
int max = INT_MAX;
int overflowed = max + 1; // UB (unlike unsigned, which wraps predictably)
```

#### 8. Division by zero / modulo by zero
```cpp
int a = 10 / 0; // UB
int b = 10 % 0; // UB
```

#### 9. Division overflow: `INT_MIN / -1`
```cpp
int r = INT_MIN / -1; // UB: result not representable as int
```

#### 10. Shifting by a negative amount or by >= the bit width of the type
```cpp
int x = 1 << 32;  // UB if int is 32 bits
int y = 1 << -1;  // UB
```

#### 11. Shifting a negative number left (pre-C++20)
```cpp
int x = -1 << 1; // UB before C++20 (well-defined since C++20)
```

#### 12. Converting a floating-point value to an integer type that cannot hold it
```cpp
double d = 1e300;
int i = static_cast<int>(d); // UB: out of range
```

### Pointers and References

#### 13. Dereferencing a null pointer
```cpp
int* p = nullptr;
std::cout << *p; // UB
```

#### 14. Pointer arithmetic outside the bounds of an array (+ one-past-the-end)
```cpp
int arr[5];
int* p = arr + 10; // UB: pointer arithmetic goes far past the valid range
```

#### 15. Comparing pointers to unrelated objects with relational operators
```cpp
int a, b;
if (&a < &b) {} // UB: `<` between pointers to unrelated objects
```

#### 16. Subtracting pointers into different arrays
```cpp
int a[5], b[5];
ptrdiff_t d = &a[0] - &b[0]; // UB
```

#### 17. Returning a reference/pointer to a local (stack) variable
```cpp
int& f() {
    int x = 5;
    return x; // UB when caller uses the returned reference
}
```

#### 18. Binding a reference to a null object via a bad dereference
```cpp
int* p = nullptr;
int& r = *p; // UB, even before r is used
```

### Object Lifetime

#### 19. Using an object after its destructor has run
```cpp
struct S { ~S() {} };
S* p = new S();
p->~S();
p->~S(); // UB: destructor called twice
```

#### 20. Accessing a moved-from object in ways the type doesn't guarantee
```cpp
std::vector<int> v1 = {1,2,3};
std::vector<int> v2 = std::move(v1);
std::cout << v1[0]; // Not strictly UB for std::vector (valid but unspecified state),
                     // but for many user-defined types this IS UB if invariants are broken.
```

#### 21. Calling a virtual function during construction/destruction expecting derived behavior
```cpp
struct Base {
    Base() { f(); } // calls Base::f, NOT Derived::f — not UB itself,
    virtual void f() { std::cout << "Base"; }
};
struct Derived : Base {
    void f() override { std::cout << "Derived"; }
};
// Not UB, but a very common surprise (defined to call Base::f).
// It BECOMES UB if Derived::f() accesses derived members not yet initialized.
```

#### 22. Using an object before its constructor completes (via `this` escaping)
```cpp
struct S {
    S() { other_thread_uses(this); } // if the object is accessed before fully constructed -> UB territory
};
```

### Sequencing

#### 23. Unsequenced multiple modifications to the same scalar
```cpp
int i = 0;
i = i++ + 1;   // UB (pre-C++17 ambiguous sequencing)
i = i++ + i++; // UB
```

#### 24. Unsequenced read and write to the same object without intervening sequencing
```cpp
int i = 0;
int j = i + i++; // UB
```
> Note: C++17 tightened sequencing rules for many expressions (e.g., `f(x++, x)` still has unsequenced arguments, but assignment operators now sequence right side before left side). Always assume unsequenced modification is UB unless you know the exact C++17+ rule.

### Type Safety and Aliasing

#### 25. Strict aliasing violations
```cpp
float f = 1.0f;
int i = *reinterpret_cast<int*>(&f); // UB: violates strict aliasing rule
```
Use `std::bit_cast` (C++20) or `memcpy` instead.

#### 26. Type punning through a union (in C++, unlike C)
```cpp
union U { int i; float f; };
U u;
u.i = 42;
std::cout << u.f; // UB in C++ (well-defined in C)
```

#### 27. Invalid `static_cast` downcast
```cpp
struct Base {};
struct Derived : Base {};
Base b;
Derived& d = static_cast<Derived&>(b); // UB: b is not actually a Derived
```

#### 28. Calling a function through a pointer of the wrong type
```cpp
void f(int) {}
using FnPtr = void(*)(double);
FnPtr p = reinterpret_cast<FnPtr>(f);
p(3.14); // UB
```

#### 29. Modifying a `const` object
```cpp
const int x = 5;
int* p = const_cast<int*>(&x);
*p = 10; // UB: x was originally declared const
```

#### 30. Modifying a string literal
```cpp
char* s = const_cast<char*>("hello");
s[0] = 'H'; // UB: string literals are effectively const, often in read-only memory
```

### Concurrency

#### 31. Data races
```cpp
int counter = 0;
// Thread 1:
counter++;
// Thread 2 (running concurrently, no synchronization):
counter++;
// UB: unsynchronized concurrent read/write to the same non-atomic object
```

#### 32. Destroying a mutex while it is locked, or by another thread
```cpp
std::mutex m;
m.lock();
// destroying m here while locked -> UB
```

### Templates and Generic Code

#### 33. Violating the One Definition Rule (ODR) across translation units
```cpp
// file1.cpp
inline int f() { return 1; }
// file2.cpp
inline int f() { return 2; }
// Linking both -> ODR violation -> UB (often IFNDR, see Part 4)
```

#### 34. Instantiating a template in a way that violates its preconditions/undocumented invariants
```cpp
std::vector<bool>::reference r = ...; // dangling reference proxy misuse is a classic UB trap
```

### Miscellaneous UB

#### 35. Missing `return` in a non-void function
```cpp
int f() {
    // no return statement
} // UB if the caller uses the return value
```

#### 36. Recursion without a base case causing unbounded stack growth
Formally this is often described as "stack overflow," which the standard treats as implementation-defined/unspecified resource exhaustion, but in practice it manifests with UB-like unpredictability (crash, memory corruption).

#### 37. Calling `std::vector::front()`/`back()` on an empty container
```cpp
std::vector<int> v;
v.front(); // UB: empty container
```

#### 38. Dereferencing an invalidated iterator
```cpp
std::vector<int> v = {1,2,3};
auto it = v.begin();
v.push_back(4); // may reallocate, invalidating it
std::cout << *it; // UB
```

#### 39. Mismatched `new`/`delete` forms
```cpp
int* p = new int[10];
delete p; // UB: should be delete[] p;
```

#### 40. Calling `memcpy` with overlapping buffers
```cpp
char buf[10] = "hello";
memcpy(buf, buf + 1, 5); // UB: overlapping regions, use memmove instead
```

#### 41. Format string mismatches with `printf`-family functions
```cpp
int x = 5;
printf("%s", x); // UB: %s expects a char*, not an int
```

#### 42. Exceeding the recursion/template-instantiation implementation limits
Formally unspecified/QoI (quality of implementation), but effectively unpredictable across compilers.

#### 43. Accessing an inactive member of a union
```cpp
union U { int i; double d; } u;
u.i = 1;
std::cout << u.d; // UB: reading the member that wasn't last written
```
(Note: this differs from the "type punning" example above only in framing — both stem from the same rule: only the last-written union member is active.)

---

## Part 2: Unspecified Behavior

The standard permits multiple outcomes, but implementations don't have to document which one they picked, and it may even vary between calls.

- **Order of evaluation of function arguments** (until fully sequenced improvements in C++17, and even then, the order between *different* arguments remains unspecified):
  ```cpp
  f(g(), h()); // order of calls to g() and h() is unspecified
  ```
- **Order of evaluation of subexpressions** in most binary operators (e.g., `a() + b()`).
- **The order in which global objects with static storage duration are initialized across different translation units** (the "static initialization order fiasco").
- **Which overload is selected** when multiple valid overloads seem equally good in edge cases involving implementation-specific conversions.
- **The exact amount of memory allocated** by `new` beyond what was requested.
- **The value of an uninitialized `bool`** (technically UB to *read*, but even if read via `memcpy` as bytes, the specific bit pattern is unspecified).
- **Whether a particular standard library function allocates memory** (small-string/small-buffer optimizations, e.g., short strings in `std::string` may or may not heap-allocate).
- **The result of `sizeof(struct)`** with padding — padding bytes exist, but their exact layout (though total size is technically implementation-defined per ABI) is not something you can assume is portable across compilers.

---

## Part 3: Implementation-Defined Behavior

The implementation *must* pick one behavior and document it (typically in the compiler manual).

- **Size of fundamental types**: `sizeof(int)`, `sizeof(long)`, `sizeof(void*)` (e.g., 4 bytes vs 8 bytes depending on platform/ABI).
- **Signedness of `char`**: whether plain `char` is signed or unsigned.
- **Result of right-shifting a negative number** (`>>` on negative signed integers): arithmetic vs logical shift.
- **Rounding behavior of integer division** with negative operands (this was implementation-defined pre-C++11, and is now defined to truncate toward zero since C++11 — a good example of behavior moving from "implementation-defined" to "well-defined" over standard revisions).
- **The numeric value produced when converting an out-of-range floating value to an integer** (for cases the standard doesn't call UB).
- **Alignment requirements of types** beyond the guaranteed minimums.
- **The order in which destructors of function-local static objects run at program termination** relative to other implementation cleanup.
- **The exact set of characters in the "basic execution character set"** and how the compiler encodes them.
- **The behavior of `#pragma` directives** the compiler doesn't recognize (ignored, but exact reporting is implementation-defined).

---

## Part 4: Ill-Formed, No Diagnostic Required (IFNDR)

This is a special (and nasty) category: the program *violates a rule of the language*, but the standard does not require compilers to detect or report it. If such a program is compiled and run, its behavior is undefined.

Common sources:

- **One Definition Rule (ODR) violations** across translation units, especially with `inline` functions, templates, and `constexpr` variables that have different definitions in different translation units.
  ```cpp
  // a.cpp
  template<typename T> T twice(T x) { return x + x; }
  // b.cpp
  template<typename T> T twice(T x) { return x * 2; } // different definition, same signature -> ODR violation, IFNDR
  ```
- **A template that is never instantiated but would be ill-formed for all possible arguments** ("no diagnostic required" cases in `[temp.res]`).
- **Exceeding implementation limits** (e.g., extremely deep template recursion) is sometimes treated similarly — the compiler isn't obligated to detect and cleanly report it.

IFNDR is arguably scarier than ordinary UB because two different compilers — or even two different optimization levels of the *same* compiler — may silently produce different, seemingly valid binaries from the same ill-formed source, with no warning at all.

---

## How to Catch UB in Practice

Since UB is invisible until it isn't, tooling is essential:

| Tool | What it catches |
|---|---|
| **UndefinedBehaviorSanitizer (UBSan)** | `-fsanitize=undefined` — signed overflow, null derefs, invalid shifts, misaligned access, etc. |
| **AddressSanitizer (ASan)** | `-fsanitize=address` — out-of-bounds access, use-after-free, double-free |
| **ThreadSanitizer (TSan)** | `-fsanitize=thread` — data races |
| **MemorySanitizer (MSan)** | Reads of uninitialized memory |
| **Valgrind (memcheck)** | Memory errors, uninitialized reads, leaks |
| **Clang Static Analyzer / clang-tidy** | Static detection of many UB patterns |
| **`-Wall -Wextra -Wpedantic -Werror`** | Compiler warnings that flag many UB-prone constructs before you even run the program |
| **GCC/Clang `-ftrapv`** | Traps on signed integer overflow at runtime |

A recommended baseline for any serious C++ project:
```bash
g++ -std=c++20 -Wall -Wextra -Wpedantic -fsanitize=address,undefined -g -O1 main.cpp -o main
```

---

