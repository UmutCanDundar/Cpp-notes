# Generic Programming

Generic programming lets you write code that works with many types, using templates. It's central to the STL and enables reusable, type-safe, high-performance libraries.

## Template Terminology

Key terms: template parameter (placeholder), template argument (actual type/value), instantiation (generated code for specific arguments), specialization (custom implementation for certain arguments).

## Function Templates

A function template generates a function for each set of type arguments it's called with.

```cpp
template <typename T>
T max_val(T a, T b) { return a > b ? a : b; }
```

## Class Templates

A class template generates a class for each combination of type arguments used.

```cpp
template <typename T>
class Box { public: T value; };
Box<int> b;
```

## Variable Templates

A variable template defines a family of variables parameterized by type.

```cpp
template <typename T>
constexpr T pi = T(3.1415926535);
double d = pi<double>;
```

## Alias Templates

An alias template gives a name to a family of types, often used to simplify complex template types.

```cpp
template <typename T>
using Vec = std::vector<T, MyAllocator<T>>;
```

## Member Templates

A member function or nested class of a class can itself be a template, independent of the enclosing class's parameters.

```cpp
class Container {
    template <typename T> void add(T item) { /* ... */ }
};
```

## Template Parameters

### Type Parameters

Standard `typename T` or `class T` parameters represent a type.

```cpp
template <typename T> void f(T x) {}
```

### Non-Type Parameters

A template parameter can be a compile-time constant value, not just a type.

```cpp
template <int N> struct Array { int data[N]; };
Array<10> a;
```

### auto Non-Type Parameters

C++17 allows `auto` for non-type template parameters, deducing their type automatically.

```cpp
template <auto N> struct Array { int data[N]; };
Array<10> a; // N deduced as int
```

### Template Template Parameters

A template parameter can itself be a template, letting you parameterize over container types.

```cpp
template <template <typename> class Container, typename T>
class Wrapper { Container<T> data; };
```

## Function Templates & Overloading

Function templates participate in overload resolution alongside regular functions and other templates; the compiler picks the best match.

```cpp
void f(int) {}
template <typename T> void f(T) {}
f(5); // calls non-template f(int) - preferred when equally good
```

## Explicit Specialization

You can provide a completely custom implementation for a specific set of template arguments.

```cpp
template <typename T> struct Printer { void print(T v) { std::cout << v; } };
template <> struct Printer<bool> { void print(bool v) { std::cout << (v ? "true" : "false"); } };
```

## Partial Specialization

For class templates, you can specialize for a subset of possible arguments (e.g., all pointer types).

```cpp
template <typename T> struct Info { static constexpr bool isPointer = false; };
template <typename T> struct Info<T*> { static constexpr bool isPointer = true; };
```

## Template Instantiation

Instantiation is when the compiler generates actual code for a template with specific arguments, either implicitly (on use) or explicitly.

### Explicit Instantiation

You can force instantiation of a template for a specific type, useful for reducing compile times or controlling binary size.

```cpp
template class std::vector<int>; // explicit instantiation
```

## Template Arguments

Arguments passed to instantiate a template — can be types, values, or other templates.

## Template Argument Deduction

The compiler infers template arguments from function call arguments, so you often don't need to specify them explicitly.

```cpp
template <typename T> void f(T x) {}
f(42); // T deduced as int
```

## Dependent & Non-Dependent Names, Templates & Friendship

Names that depend on a template parameter need `typename`/`template` disambiguation. Friendship in templates can be granted to specific instantiations or all of them.

```cpp
template <typename T>
struct A {
    typename T::type value; // dependent name needs 'typename'
};
```

## CTAD (Class Template Argument Deduction)

Since C++17, constructors can deduce class template arguments, so you often don't need to specify `<...>` explicitly.

```cpp
std::pair p(1, 2.0); // deduces std::pair<int, double>
```

## Meta Functions and Standard Type Traits Library

Type traits are compile-time metafunctions that inspect or transform types, used heavily for SFINAE and constraints.

```cpp
static_assert(std::is_integral_v<int>);
using T = std::remove_const_t<const int>; // int
```

## static_assert Declarations

Performs a compile-time check, causing a compile error with a message if the condition is false.

```cpp
static_assert(sizeof(int) == 4, "int must be 4 bytes");
```

## std::type_identity

A trivial trait that returns the same type unchanged, useful to block template argument deduction for a parameter.

```cpp
template <typename T>
void f(std::type_identity_t<T> x, T y) {} // x doesn't participate in deduction
```

## Tag Dispatch

Uses overloading on empty "tag" types to select different implementations at compile time, an alternative to `if constexpr`.

```cpp
void impl(std::true_type) { /* fast path */ }
void impl(std::false_type) { /* slow path */ }
```

## if constexpr (static if)

Compiles only the branch matching a compile-time condition, discarding the other — enables simpler generic code without full tag dispatch.

```cpp
template <typename T>
void print(T v) {
    if constexpr (std::is_pointer_v<T>) std::cout << *v;
    else std::cout << v;
}
```

## Variadic Templates

Allow a template to accept an arbitrary number of arguments of arbitrary types.

```cpp
template <typename... Args>
void log(Args... args) { (std::cout << ... << args); }
```

### sizeof... Operator

Returns the number of arguments in a parameter pack.

```cpp
template <typename... Args>
constexpr size_t count() { return sizeof...(Args); }
```

### Parameter Packs

A parameter pack represents zero or more template or function parameters, expanded with `...`.

```cpp
template <typename... Ts> void f(Ts... args) {}
```

### Pack Expansion Patterns

The `...` syntax expands a pack in various contexts: function calls, initializer lists, base class lists.

```cpp
template <typename... Bases>
struct Derived : Bases... {}; // expands into multiple base classes
```

### Fold Expressions / Unary Fold / Binary Fold

Fold expressions (C++17) apply an operator across a whole parameter pack concisely.

```cpp
template <typename... Args>
auto sum(Args... args) { return (args + ...); } // unary right fold
```

## SFINAE / std::enable_if / std::void_t

SFINAE ("Substitution Failure Is Not An Error") lets invalid template substitutions be silently removed from overload resolution instead of causing errors, enabling conditional compilation of templates.

```cpp
template <typename T, typename = std::enable_if_t<std::is_integral_v<T>>>
void onlyInts(T x) {}
```

## std::declval

Produces a "fake" reference to a type for use in unevaluated contexts (like `decltype`), without needing an actual object.

```cpp
template <typename T>
using ResultOf = decltype(std::declval<T>().doWork());
```

## C++20/23 Additions

C++20 added concepts, `if consteval`, and abbreviated function templates (`auto` params). C++23 refined constraints and added more standard concepts.

```cpp
void f(auto x) {} // abbreviated function template, C++20
```
