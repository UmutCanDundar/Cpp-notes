# Spaceship Operator (C++20)

The spaceship operator `<=>` auto-generates all six relational operators from one definition. It simplifies writing comparison logic and reduces bugs from inconsistent operators.

## Comparison Categories

The result type of `<=>` describes what kind of ordering is possible: `strong_ordering`, `weak_ordering`, or `partial_ordering`.

```cpp
auto result = 3 <=> 5; // std::strong_ordering::less
```

## Strong/Weak/Partial Ordering

- **Strong ordering**: equal values are truly indistinguishable (e.g., integers).
- **Weak ordering**: equivalent but possibly distinguishable values (e.g., case-insensitive strings).
- **Partial ordering**: some values are unordered/incomparable (e.g., floating point with NaN).

```cpp
struct Point {
    int x, y;
    auto operator<=>(const Point&) const = default; // strong_ordering
};
```

## Partial Ordering

Used when comparisons can be "unordered," such as floating-point NaN comparisons.

```cpp
double a = 0.0/0.0; // NaN
auto r = a <=> 1.0; // std::partial_ordering::unordered
```

## Spaceship Operator in STL

Many STL types (like `std::tuple`, `std::pair`, `std::vector`) support `<=>` for automatic lexicographic comparison.

```cpp
std::pair<int,int> p1{1,2}, p2{1,3};
bool less = p1 < p2; // uses <=> internally
```
