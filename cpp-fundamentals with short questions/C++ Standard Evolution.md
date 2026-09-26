# C++ Standard Evolution: C++98 → C++26

---

## C++98 / C++03 — the foundation

| Feature | Description | Example |
|---------|-------------|---------|
| Templates | Generic, type-parameterized functions/classes. | `template<class T> T max(T a, T b);` |
| STL containers | `vector`, `list`, `map`, `set`, `string`, etc. | `std::vector<int> v = {1,2,3};` (init-list syntax is actually C++11 — in C++98 you'd `push_back` each element) |
| STL algorithms | `sort`, `find`, `for_each`, etc., operating via iterators. | `std::sort(v.begin(), v.end());` |
| Exceptions | `try`/`catch`/`throw` for error handling. | `try { throw std::runtime_error("x"); } catch (...) {}` |
| RTTI | `typeid` and `dynamic_cast` for runtime type info. | `if (dynamic_cast<Derived*>(base_ptr)) {...}` |
| Namespaces | Grouping to avoid name collisions. | `namespace ns { void f(); }` |
| Casts | `static_cast`, `dynamic_cast`, `const_cast`, `reinterpret_cast`. | `static_cast<int>(3.9);` |
| Function objects | Callable class instances (functors), used with algorithms. | `struct Adder { int operator()(int x) const { return x+1; } };` |
| `auto_ptr` | Early (flawed) single-ownership smart pointer — copying "steals" ownership. Deprecated in C++11, removed in C++17. | `std::auto_ptr<int> p(new int(5));` |
| `bool`, `mutable`, `explicit` | Core keywords added over C (or refined) for the object model. | `explicit Foo(int x);` |
| Virtual functions & polymorphism | Runtime dispatch through a base class pointer/reference. | `virtual void speak() { std::cout << "..."; }` |
| Operator overloading | Give operators custom meaning for user types. | `Point operator+(const Point& o) const;` |
| References & default arguments | Aliases for existing objects; parameters with fallback values. | `void f(int& x, int y = 10);` |
| Multiple inheritance | A class can inherit from more than one base class. | `class C : public A, public B {};` |
| `iostream` | `cin`/`cout`/`cerr` stream-based I/O, replacing C's `printf`/`scanf`. | `std::cout << "hi" << std::endl;` |

---

## C++11 — the big one

| Feature | Description | Example |
|---------|-------------|---------|
| `auto` | Type deduced from the initializer. | `auto x = 5;` |
| `decltype` | Yields the compile-time type of an expression. | `decltype(x) y = 10;` |
| Range-based `for` | Iterates a container without explicit iterators. | `for (int x : v) { ... }` |
| Lambda expressions | Inline anonymous function objects. | `auto f = [](int x){ return x+1; };` |
| Move semantics / rvalue refs | `&&` references + move constructors avoid unnecessary copies. | `std::vector<int> b = std::move(a);` |
| Smart pointers | `unique_ptr`, `shared_ptr`, `weak_ptr` — RAII-managed ownership. | `auto p = std::make_shared<Foo>();` |
| `nullptr` | Type-safe null pointer literal, replaces `NULL`/`0`. | `int* p = nullptr;` |
| `enum class` | Scoped, strongly-typed enums (no implicit int conversion). | `enum class Color { Red, Green };` |
| `static_assert` | Compile-time assertion. | `static_assert(sizeof(int)==4, "bad platform");` |
| Variadic templates | Templates taking any number of type arguments. | `template<class... Args> void f(Args... args);` |
| Uniform initialization | `{}`-braces for initializing anything. | `int arr[] = {1,2,3};` |
| `constexpr` | Compile-time evaluable functions/values. | `constexpr int square(int x){ return x*x; }` |
| Threading library | `<thread>`, `<mutex>`, `<atomic>`, `<future>`. | `std::thread t([]{ /*...*/ });` |
| `unordered_map`/`unordered_set` | Hash-table-based associative containers. | `std::unordered_map<std::string,int> m;` |
| `tuple`, `array`, `chrono`, `regex` | New standard library components. | `std::tuple<int,std::string> t(1,"a");` |
| `override` / `final` | Explicit virtual-function-override intent, prevents typos. | `void f() override;` |
| Delegating constructors | One constructor calling another. | `Foo() : Foo(0) {}` |
| `= default` / `= delete` | Explicitly request/suppress a compiler-generated special member. | `Foo(const Foo&) = delete;` |
| `<type_traits>` | Compile-time type introspection (`is_integral`, etc.). | `std::is_integral<int>::value` |
| `noexcept` | Declares a function won't throw — enables optimizations, calls `terminate` if it does anyway. | `void f() noexcept;` |
| User-defined literals | Custom suffixes for literal values. | `operator""_km(long double d);  // 5.0_km` |
| Raw string literals | No-escape string literals, great for regex/paths. | `R"(C:\path\no\escapes)"` |
| Trailing return type | Return type written after the parameter list. | `auto f(int x) -> int { return x; }` |
| `std::function` | Type-erased wrapper for any callable. | `std::function<int(int)> f = [](int x){return x;};` |
| Attributes `[[...]]` | Standardized syntax for compiler hints (`[[noreturn]]`, etc.). | `[[noreturn]] void die();` |
| `long long`, `alignas`/`alignof` | 64-bit integer type; explicit control over object alignment. | `alignas(16) int x;` |

---

## C++14 — the small refinement

| Feature | Description | Example |
|---------|-------------|---------|
| Generic lambdas | Lambda parameters can be `auto`. | `auto f = [](auto x){ return x+1; };` |
| Lambda init-capture | Capture-by-move / compute-and-capture into a new name. | `auto f = [x = std::move(y)]{ return x; };` |
| Function return type deduction | `auto` as a function's return type, deduced from `return`. | `auto add(int a, int b) { return a+b; }` |
| Relaxed `constexpr` | Loops/branches allowed inside `constexpr` functions (not just one `return`). | `constexpr int fact(int n){ int r=1; for(int i=1;i<=n;i++) r*=i; return r; }` |
| `std::make_unique` | Factory function for `unique_ptr` (symmetry with `make_shared`, which existed since C++11). | `auto p = std::make_unique<Foo>();` |
| Variable templates | Templated compile-time constants. | `template<class T> constexpr T pi = T(3.1415926535);` |
| Binary literals / digit separators | `0b`-prefixed literals, `'` as a readability separator. | `int x = 0b1010'1010;` |
| `[[deprecated]]` | Standard attribute marking an API as obsolete — triggers a compiler warning on use. | `[[deprecated("use g() instead")]] void f();` |
| `std::exchange` | Assigns a new value, returns the old one — handy in move constructors. | `old_ptr = std::exchange(ptr, nullptr);` |

---

## C++17 — big library additions

| Feature | Description | Example |
|---------|-------------|---------|
| Structured bindings | Unpack a tuple/pair/struct into named variables in one line. | `auto [a, b] = std::make_pair(1, 2);` |
| `if`/`switch` with init-statement | Scope a variable to just the condition. | `if (auto it = m.find(k); it != m.end()) {...}` |
| `std::optional` | A value that may or may not be present, no heap allocation. | `std::optional<int> maybe = std::nullopt;` |
| `std::variant` | Type-safe tagged union. | `std::variant<int, std::string> v = "x";` |
| `std::string_view` | Non-owning view of string data, zero allocation. | `void f(std::string_view s);` |
| `std::filesystem` | Cross-platform path/file operations. | `std::filesystem::exists("a.txt");` |
| `constexpr if` | Compile-time branch selection inside a template. | `if constexpr (std::is_integral_v<T>) {...}` |
| Class template argument deduction (CTAD) | Template arguments inferred from constructor arguments. | `std::pair p(1, "a");` (no `<int, const char*>` needed) |
| Fold expressions | Collapse a parameter pack with a binary operator in one expression. | `template<class... Args> auto sum(Args... a){ return (a + ...); }` |
| `[[nodiscard]]`, `[[maybe_unused]]`, `[[fallthrough]]` | Standard attributes for compiler diagnostics. | `[[nodiscard]] int compute();` |
| Parallel algorithms | `<execution>` policies (`par`, `par_unseq`) for STL algorithms. | `std::sort(std::execution::par, v.begin(), v.end());` |
| Nested namespaces | `namespace A::B { ... }` shorthand. | `namespace app::net { void connect(); }` |
| `std::any` | Type-safe container for a single value of any type. | `std::any a = 5; a = std::string("x");` |
| `std::byte` | A distinct type for raw byte data — not `char`, not `int`, no arithmetic implied. | `std::byte b{0xFF};` |
| Inline variables | `inline` on a variable, allowing definition in a header without ODR violations. | `inline int counter = 0;` |
| Guaranteed copy elision | Returning a prvalue by value is guaranteed not to copy/move at all (not just optimized away). | `Widget make() { return Widget(); } // no move ctor call needed` |
| `std::invoke` / `std::apply` | Uniformly call any callable · call a function with a tuple's elements unpacked as arguments. | `std::invoke(fn, args...);` |

---

## C++20 — the second "big one"

| Feature | Description | Example |
|---------|-------------|---------|
| Concepts | Named, checkable constraints on template parameters. | `template<std::integral T> T add(T a, T b);` |
| Ranges | Composable, lazy view-based algorithms over sequences. | `for (int x : v \| std::views::filter([](int x){return x%2==0;})) {...}` |
| Coroutines | Native `co_await`/`co_yield`/`co_return` for suspendable functions. | `generator<int> range(int n){ for(int i=0;i<n;i++) co_yield i; }` |
| Modules | `import`/`export` as an alternative to header files. | `export module math; export int add(int,int);` (consumer: `import math;`) |
| Spaceship operator | `<=>` auto-generates all six comparison operators. | `auto operator<=>(const Point&) const = default;` |
| `std::format` | Type-safe, `{}`-based string formatting (Python-style). | `std::format("{} + {} = {}", 1, 2, 3);` |
| `consteval` / `constinit` | Force compile-time-only evaluation · guarantee constant (no dynamic) initialization. | `consteval int square(int x){ return x*x; }` |
| `std::span` | Non-owning view over a contiguous array/buffer. | `void f(std::span<int> data);` |
| `std::jthread` | `std::thread` that auto-joins and supports cooperative cancellation. | `std::jthread t([](std::stop_token st){...});` |
| Designated initializers | Name struct members explicitly when initializing (C-style). | `Point p{.x = 1, .y = 2};` |
| Template lambdas | Explicit template parameter list on a lambda. | `auto f = []<class T>(T x){ return x; };` |
| Abbreviated function templates | `auto` parameters directly in a plain function (not just lambdas). | `void f(auto x) { ... }` |
| `char8_t` | Distinct type for UTF-8 code units. | `char8_t c = u8'A';` |
| `using enum` | Brings an `enum class`'s enumerators into scope, avoiding repeated qualification. | `using enum Color; auto c = Red;` |
| `[[likely]]` / `[[unlikely]]` | Branch-prediction hints for the compiler. | `if (x) [[likely]] { ... }` |
| `std::source_location` | Captures the calling file/line/function without macros. | `void log(std::source_location loc = std::source_location::current());` |
| `<bit>` header | `popcount`, `has_single_bit`, `bit_cast`, and other bit-manipulation utilities. | `std::popcount(0b1011u); // 3` |
| `std::to_array` | Converts a C array to a `std::array`, deducing size and type. | `auto a = std::to_array({1,2,3});` |
| Class-type non-type template parameters | Literal class objects usable directly as template arguments. | `template<Point P> void f();` |
| Parenthesized aggregate init | Aggregates can be initialized with `()`, not just `{}`. | `struct P { int x, y; }; P p(1, 2);` |

---

## C++23 — refinement + a few big items

| Feature | Description | Example |
|---------|-------------|---------|
| `std::expected` | Return-value alternative to exceptions — holds a value or an error. | `std::expected<int, std::string> parse(std::string_view);` |
| `std::print` / `std::println` | Formats and writes directly to a stream in one call. | `std::println("x = {}", x);` |
| Deducing `this` (explicit object parameter) | Write the implicit `this` as a named, deducible first parameter. | `auto get(this auto&& self) { return self.value; }` |
| `if consteval` | Branch based on whether the current evaluation is compile-time. | `if consteval { /* compile-time path */ } else { /* runtime path */ }` |
| Multidimensional `operator[]` | Subscript operator can take multiple indices directly. | `mat[i, j] = 5;` |
| `std::mdspan` | Non-owning multidimensional array view. | `std::mdspan<int, std::extents<size_t,3,3>> m(data);` |
| `std::flat_map` / `std::flat_set` | Sorted-vector-backed associative containers — cache-friendlier than tree-based `map`/`set`. | `std::flat_map<int,std::string> m;` |
| `auto(x)` / `auto{x}` | Explicit decay-copy of an expression. | `auto y = auto(x);` |
| `std::stacktrace` | Capture and print the current call stack. | `std::cout << std::stacktrace::current();` |
| `std::move_only_function` | `std::function`-like wrapper for move-only callables. | `std::move_only_function<void()> f = [p = std::move(ptr)]{};` |
| `std::to_underlying` | Converts a scoped enum to its underlying integer type. | `std::to_underlying(Color::Red);` |
| `size_t` literal suffix `uz` | Literal directly of type `size_t`. | `auto n = 5uz;` |
| `import std;` | Import the entire standard library as a single module, instead of `#include`-ing headers. | `import std; std::println("hi");` |
| `std::generator` | Coroutine-based lazy sequence type from the ranges library. | `std::generator<int> range(int n){ for(int i=0;i<n;i++) co_yield i; }` |
| `ranges::to` | Converts any range into a concrete container. | `auto v = r \| std::ranges::to<std::vector>();` |
| `static operator()` / `static operator[]` | Call/subscript operators that don't need a `this` (no captured state). | `struct Adder { static int operator()(int a, int b) { return a+b; } };` |
| `std::unreachable()` | Marks a code path as impossible, letting the compiler optimize accordingly (UB if actually reached). | `default: std::unreachable();` |
| `<stdfloat>` | Fixed-width floating-point types (`float16_t`, `float128_t`, etc.). | `std::float32_t f = 1.5f32;` |

---

## C++26 — finalized 28 March 2026

| Feature | Description | Example |
|---------|-------------|---------|
| Static reflection | Compile-time introspection of types/members via the `^^` ("reflect") operator and `[: :]` splice syntax. | `constexpr auto r = ^^int; using T = [: r :]; // T is int` |
| Contracts | Preconditions/postconditions/assertions checked at runtime, declared in the function signature. | `void f(int x) pre(x > 0) { ... }` |
| `std::execution` (senders/receivers) | Standardized composable model for asynchronous/parallel work (based on P2300), unifying thread pools, futures, and executors. | `auto work = sender \| then([]{ return 42; });` |
| Parallel range algorithms | Ranges-library algorithms gain execution-policy overloads. | `std::ranges::sort(std::execution::par, v);` |
| Safety-by-default | Reading an uninitialized variable is no longer undefined behavior (deterministic value instead). | `int x; // now well-defined, not UB, if read before assignment` |
| `#embed` | Preprocessor directive to include a binary file's contents directly as data. | `const unsigned char img[] = { #embed "logo.png" };` |
| Pack indexing | Directly index into a template parameter pack without recursion. | `template<class... Ts> using First = Ts...[0];` |
| Trivial relocatability | Lets the compiler `memcpy`-relocate objects instead of move-constructing + destroying, when safe. | `class [[trivially_relocatable]] Buffer { ... };` |
| Concurrent queues | First concurrent (thread-safe) container added to the standard library. | `std::concurrent_queue<int> q;` |
| SIMD parallelism | `std::simd` — portable vectorized data-parallel types (re-added after being cut from C++20). | `std::simd<float> v = {1.0f, 2.0f, 3.0f, 4.0f};` |
| Async scopes & parallel scheduler | RAII-style structured lifetime management for async work, plus a standard thread-pool scheduler. | `co_await async_scope.spawn(do_work());` |
| Hardening / bounds-checked access | Optional standard library hardening mode adds bounds checks and null-pointer validation for safety. | *(compiler/library flag, not new syntax)* |

