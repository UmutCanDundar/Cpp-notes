# `std::vector` Methods

| Method | Usage | Note |
|--------|-------|------|
| `push_back` | `v.push_back(x)` | Append to end. |
| `emplace_back` | `v.emplace_back(args...)` | Construct in place, prefer this. |
| `pop_back` | `v.pop_back()` | Remove from end. |
| `insert` | `v.insert(it, x)` | Insert at iterator position. |
| `erase` | `v.erase(it)` / `v.erase(a, b)` | Remove single element or range. |
| `reserve` | `v.reserve(n)` | Prevent reallocation, pre-allocate. |
| `resize` | `v.resize(n)` | Change size, new elements are default-initialized. |
| `shrink_to_fit` | `v.shrink_to_fit()` | Reduce capacity to match size. |
| `clear` | `v.clear()` | Remove elements but capacity remains. |
| `size` / `capacity` | `v.size()` / `v.capacity()` | Element count / allocated space. |
| `front` / `back` | `v.front()` / `v.back()` | Reference to first / last element. |
| `data` | `v.data()` | Raw pointer. |
| `assign` | `v.assign(n, val)` | Replace content. |


# `std::vector` Initialization

## Standalone

```cpp
// 1. default — empty
std::vector<int> v;

// 2. size only — default initialized (0 for int)
std::vector<int> v(10);             // {0,0,0,0,0,0,0,0,0,0}

// 3. size + value
std::vector<int> v(10, 42);         // {42,42,42,42,42,42,42,42,42,42}

// 4. initializer list
std::vector<int> v = {1, 2, 3, 4, 5};
std::vector<int> v {1, 2, 3, 4, 5};  // same thing

// 5. copy constructor
std::vector<int> v2 = v1;
std::vector<int> v2(v1);            // same thing

// 6. move constructor
std::vector<int> v2 = std::move(v1);  // v1 is empty after this

// 7. iterator range
std::vector<int> v(arr, arr + n);
std::vector<int> v(other.begin(), other.end());

// 8. assign — resize + fill after construction
std::vector<int> v;
v.assign(10, 42);                   // fill with value
v.assign({1, 2, 3, 4, 5});         // from initializer list

// 9. reserve + push_back — when final size is known
std::vector<int> v;
v.reserve(10);                      // allocate, no elements yet
v.push_back(1);
```

---

## Inside a Class

```cpp
class Foo {
    // ✅ C++11 default member initializer — use = or {}
    std::vector<int> v = {1, 2, 3};
    std::vector<int> v{1, 2, 3};                       // same thing
    std::vector<int> v = std::vector<int>(10, 0);       // size + value

    // ❌ constructor syntax — NOT allowed as member initializer
    std::vector<int> v(10, 0);   // compile error — looks like a function declaration
    std::vector<int> v(10);      // compile error — most vexing parse

    // ✅ constructor initialization list — correct place for (size, value) form
    Foo() : v(10, 0) {}
    Foo() : v{1, 2, 3} {}
};
```

### Why `v(10, 0)` fails as a member initializer

```cpp
std::vector<int> v(10, 0);
// compiler reads this as:
// a function named v, taking two ints, returning std::vector<int>
// — most vexing parse
```

Use `= {}` or `{}` for in-class initialization.
Use the constructor initialization list (`: v(10, 0)`) for size/value forms.

---

## Quick Reference

| Syntax | Where | Notes |
|--------|-------|-------|
| `v(n)` | standalone, init list | n default-initialized elements |
| `v(n, val)` | standalone, init list | n copies of val |
| `v{1,2,3}` | standalone, in-class, init list | initializer list |
| `v = {1,2,3}` | standalone, in-class | same as above |
| `v = std::move(other)` | standalone, in-class | no allocation |
| `v(it, it)` | standalone, init list | range copy |
| `v(n, val)` in-class | ❌ | use `= vector<int>(n, val)` |
