# Constants & Constant Expressions

Compile-time constants let the compiler compute values before the program runs, improving performance and enabling things like array sizes and template arguments. They also help catch errors early.

## constexpr Variables

A `constexpr` variable's value must be computable at compile time and is implicitly `const`.

```cpp
constexpr int size = 10;
int arr[size]; // valid: size known at compile time
```

## constexpr Functions

A `constexpr` function *can* run at compile time if given constant arguments; otherwise it runs at runtime like a normal function.

```cpp
constexpr int square(int x) { return x * x; }
constexpr int r = square(5); // computed at compile time
```

## constexpr Constructors

A `constexpr` constructor allows objects of a class to be created as compile-time constants, useful for literal types.

```cpp
struct Point {
    constexpr Point(int x, int y) : x(x), y(y) {}
    int x, y;
};
constexpr Point p(1, 2);
```

## Literal Types

A literal type is a type simple enough to be used in constant expressions (e.g., has a `constexpr` constructor, trivial destructor). Only literal types can be `constexpr` variables.

```cpp
struct Simple { constexpr Simple(int v) : v(v) {} int v; }; // literal type
```

## consteval & Immediate Functions

`consteval` functions ("immediate functions") *must* be evaluated at compile time; calling them at runtime is a compile error.

```cpp
consteval int square(int x) { return x * x; }
constexpr int r = square(4); // OK, forced compile-time
```

## constinit

`constinit` guarantees a variable is initialized at compile time (no runtime initialization order issues), but the variable itself may still be mutable afterward.

```cpp
constinit int counter = 0; // initialized at compile time, can change later
```

## constexpr Virtual Functions

Since C++20, virtual functions can be `constexpr`, allowing polymorphic code to run at compile time when the dynamic type is known.

```cpp
struct Base { constexpr virtual int f() const { return 1; } };
struct Derived : Base { constexpr int f() const override { return 2; } };
```

## constexpr Lambda Functions

Lambdas can be `constexpr` (implicitly since C++17 if they qualify), letting them be evaluated at compile time.

```cpp
constexpr auto square = [](int x) { return x * x; };
constexpr int r = square(6);
```

## User Defined Literals

Custom suffixes let you create literals with clear semantics, like `10_km`, improving readability and type safety.

```cpp
constexpr long double operator"" _km(long double km) { return km * 1000; }
auto distance = 5.0_km; // 5000.0
```

## C++20/23 Additions

C++20 added `consteval`, `constinit`, constexpr virtual functions, and more relaxed constexpr rules (dynamic allocation, try/catch). C++23 further relaxed constexpr for things like `std::unique_ptr` usage.

```cpp
constexpr int compute() {
    int* p = new int(5); // allowed in constexpr since C++20
    int v = *p;
    delete p;
    return v;
}
```
