# std::ranges (C++20)

Ranges let you operate on whole sequences (not just iterator pairs), compose operations lazily, and write more readable, less error-prone algorithm pipelines.

## Basics

A range is anything with `begin()`/`end()`. `std::ranges` algorithms take the range directly instead of two iterators.

```cpp
std::vector<int> v{5,3,1,4};
std::ranges::sort(v); // no need for v.begin(), v.end()
```

## Range Access

Free functions like `std::ranges::begin`, `std::ranges::end`, `std::ranges::size` uniformly access range boundaries and size, working with C arrays and custom types too.

```cpp
int arr[5]{1,2,3,4,5};
auto s = std::ranges::size(arr); // 5
```

## Range Primitives

Core building blocks like `std::ranges::iterator_t`, `std::ranges::range_value_t` extract associated types from a range.

```cpp
using ValueType = std::ranges::range_value_t<std::vector<int>>; // int
```

## Range Concepts

Concepts like `std::ranges::input_range`, `std::ranges::random_access_range` classify ranges by their capabilities, used to constrain algorithms.

```cpp
template <std::ranges::random_access_range R>
void fastSort(R&& r) { std::ranges::sort(r); }
```

## Range Factories

Functions that create ranges from scratch, without an underlying container, e.g. `std::views::iota` for number sequences.

```cpp
for (int i : std::views::iota(1, 5)) std::cout << i; // 1234
```

## Range Adaptors

Lazy, composable views that transform a range, chained with `|`, without creating intermediate containers.

```cpp
auto result = v | std::views::filter([](int x) { return x % 2 == 0; })
                | std::views::transform([](int x) { return x * x; });
```
