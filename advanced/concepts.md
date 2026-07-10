# Concepts (C++20)

Concepts let you specify readable, checkable requirements on template parameters. They replace cryptic SFINAE errors with clear compile-time messages and self-documenting code.

## Constraints

A constraint is a predicate evaluated at compile time that a template argument must satisfy.

```cpp
template <typename T> requires std::is_integral_v<T>
T add(T a, T b) { return a + b; }
```

## Requires Clauses

A `requires` clause attaches constraints to a template, restricting which types can be used.

```cpp
template <typename T>
requires std::integral<T>
T doubleIt(T x) { return x * 2; }
```

## Requires Expressions

A `requires { ... }` expression checks whether a set of expressions/operations is valid for given types, producing a boolean.

```cpp
template <typename T>
concept HasSize = requires(T t) { t.size(); };
```

## Type Requirements

Inside a `requires` expression, `typename T::value_type;` checks that a nested type exists.

```cpp
template <typename T>
concept HasValueType = requires { typename T::value_type; };
```

## Compound Requirements

Check both that an expression is valid *and* that its result satisfies another constraint (optionally `noexcept`).

```cpp
template <typename T>
concept Addable = requires(T a, T b) {
    { a + b } -> std::same_as<T>;
};
```

## Nested Requirements

A `requires` clause can be nested inside a `requires` expression to add extra boolean conditions.

```cpp
template <typename T>
concept SmallType = requires {
    requires sizeof(T) <= 8;
};
```

## Concepts

A named concept bundles constraints into a reusable, readable name usable in templates.

```cpp
template <typename T>
concept Numeric = std::is_arithmetic_v<T>;

template <Numeric T>
T square(T x) { return x * x; }
```

## Type Constraints and auto

Concepts can constrain `auto` parameters and variables directly, giving a lightweight generic syntax.

```cpp
void print(Numeric auto x) { std::cout << x; }
```

## Standard Concepts

The standard library provides many ready-made concepts like `std::integral`, `std::floating_point`, `std::copyable`, `std::invocable`.

```cpp
template <std::integral T>
T increment(T x) { return x + 1; }
```
