# Type Deduction (C++11/14/17/20/23)

Type deduction lets the compiler figure out types automatically instead of you writing them explicitly. It reduces verbosity, avoids mistakes with complex template types, and is the foundation for generic code.

## auto Type Deduction

`auto` deduces a variable's type from its initializer, following the same rules as template argument deduction.

```cpp
auto x = 5;          // int
auto y = 5.0;         // double
auto v = std::vector<int>{1,2,3}; // std::vector<int>
```

## decltype Specifier

`decltype(expr)` yields the exact declared type of an expression, including references and const, without evaluating it.

```cpp
int x = 0;
decltype(x) y = 10;       // int
decltype((x)) z = x;      // int& (parentheses matter!)
```

## decltype(auto)

Combines `auto`'s deduction from an initializer with `decltype`'s exact-type rules, preserving references and const-ness.

```cpp
int x = 5;
int& getRef() { return x; }
decltype(auto) r = getRef(); // int&, not int
```

## Trailing Return Type

Lets you declare a function's return type after the parameter list, useful when the return type depends on the parameters.

```cpp
template <typename T, typename U>
auto add(T t, U u) -> decltype(t + u) { return t + u; }
```

## auto Return Type Deduction in Lambda Expressions

Lambdas can deduce their return type automatically from the `return` statement, just like `auto` functions.

```cpp
auto add = [](int a, int b) { return a + b; }; // return type deduced as int
```

## Type Deductions in Function & Class Templates

Function templates deduce their type parameters from the arguments passed. Class templates (since C++17) can deduce from constructor arguments too.

```cpp
template <typename T> void f(T x) {}
f(42); // T = int, deduced automatically
```

## Deduction Guides

Deduction guides tell the compiler how to deduce class template parameters from constructor arguments, used with Class Template Argument Deduction (CTAD).

```cpp
template <typename T>
struct Box { Box(T t); };
Box(const char*) -> Box<std::string>; // custom deduction guide

Box b("hi"); // deduces Box<std::string> instead of Box<const char*>
```
