# Lambda Expressions (C++11/14/17/20/23)

Lambdas let you define small, anonymous, inline functions. They are heavily used with STL algorithms, callbacks, and functional-style code, replacing verbose function objects.

## Lambda Expressions and Type Deductions

A lambda's parameter and return types can be deduced with `auto`, making it behave like a generic function.

```cpp
auto add = [](auto a, auto b) { return a + b; }; // generic lambda (C++14)
```

## Lambda Expressions & constexpr

Lambdas can be evaluated at compile time if their body qualifies as a constant expression.

```cpp
constexpr auto square = [](int x) { return x * x; };
static_assert(square(3) == 9);
```

## Generalized Lambda Expressions

C++14 introduced generic (polymorphic) lambdas using `auto` parameters, so one lambda can work with multiple types.

```cpp
auto print = [](const auto& x) { std::cout << x; };
print(5); print("hi");
```

## Lambda Init Capture

Init capture (C++14) lets you initialize a new variable in the capture clause, useful for moving objects into a lambda.

```cpp
auto ptr = std::make_unique<int>(5);
auto lambda = [p = std::move(ptr)] { return *p; };
```

## Lambda Expressions & Perfect Forwarding

Lambdas can accept forwarding references to preserve argument value category when forwarding to another call.

```cpp
auto forwarder = [](auto&&... args) {
    target(std::forward<decltype(args)>(args)...);
};
```

## Pack Expansions in Lambda Expressions

Lambdas can capture and expand parameter packs, useful for wrapping variadic function calls.

```cpp
template <typename... Args>
auto make(Args&&... args) {
    return [args...] { return std::make_tuple(args...); };
}
```

## Lambda Expressions & STL Algorithms

Lambdas are the standard way to supply predicates/comparators to STL algorithms, replacing verbose functors.

```cpp
std::vector<int> v{5,3,1,4};
std::sort(v.begin(), v.end(), [](int a, int b) { return a > b; });
```

## Recursive Lambda

Lambdas can't easily call themselves by name, but you can use `std::function`, a Y-combinator style trick, or (C++23) the `this` deducing parameter.

```cpp
std::function<int(int)> fact = [&fact](int n) {
    return n <= 1 ? 1 : n * fact(n - 1);
};
```

## Lambda Expressions in Member Functions

Lambdas inside member functions can capture `this` (or `*this` by value in C++17) to access the enclosing object's members.

```cpp
struct Widget {
    int value = 5;
    auto getter() { return [*this] { return value; }; } // captures a copy
};
```

## Lambda Expressions for Functional Programming

Lambdas enable composing functions, currying, and passing behavior as data, key techniques in functional-style C++.

```cpp
auto compose = [](auto f, auto g) { return [=](auto x) { return f(g(x)); }; };
```

## Lambda Expressions in C++20/23

C++20 added template parameters on lambdas, new capture forms, and use in unevaluated contexts; C++23 added deducing `this`.

## Template Parameter Lists on Lambdas (C++20)

You can explicitly specify template parameters on a lambda for more control than `auto` gives.

```cpp
auto f = []<typename T>(std::vector<T> v) { return v.size(); };
```

## New Lambda Captures (C++20)

C++20 allows capturing structured bindings and pack expansion in init-capture.

```cpp
auto [a, b] = std::pair{1, 2};
auto l = [a, b] { return a + b; }; // capturing structured bindings
```

## Lambda Expressions in Unevaluated Context (C++20)

Lambdas can now appear in unevaluated contexts such as `decltype` and template arguments.

```cpp
using Comparator = decltype([](int a, int b) { return a < b; });
```

## Lambda Init Capture Pack Expansions (C++20)

You can now expand a parameter pack directly in an init-capture.

```cpp
template <typename... Args>
auto wrap(Args... args) {
    return [...args = std::move(args)] { /* use args */ };
}
```

## Default Constructible and Assignable Stateless Lambdas (C++20)

Lambdas with no captures are now default-constructible and assignable, so they can be used as template parameters like `std::less<>`.

```cpp
auto cmp = [](int a, int b) { return a < b; };
decltype(cmp) cmp2; // default constructed, C++20
```

## Multiple Lambda

Refers to composing or chaining several lambdas together, e.g., an "overload set" combining multiple lambdas (see variant overloader idiom).

```cpp
template <typename... Ts> struct overload : Ts... { using Ts::operator()...; };
template <typename... Ts> overload(Ts...) -> overload<Ts...>;
```

## IIFE (Immediately Invoked Function Expression)

Calling a lambda immediately after defining it, useful for initializing a `const` variable with complex logic.

```cpp
const int result = []{ int x = 1; x += 2; return x; }();
```

## Lambda Call Once

A pattern to ensure a lambda's logic runs only once, often via `std::call_once` or a static flag.

```cpp
std::call_once(flag, [] { std::cout << "init\n"; });
```

## Type Distinction Through Lambda

Since each lambda has a unique, unnamed type, lambdas can be used to create distinct "tag" types for overload resolution or type-based dispatch.

```cpp
auto tagA = [] {};
auto tagB = [] {};
static_assert(!std::is_same_v<decltype(tagA), decltype(tagB)>);
```
