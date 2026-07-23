# C++ Iterators — Comprehensive Reference

---

## Table of Contents

** Iterators**
1. [Ranges](#1-ranges)
2. [Iterator Categories](#2-iterator-categories)
3. [Container begin()/end() Member Functions](#3-container-beginend-member-functions)
4. [std::begin / std::end](#4-stdbegin--stdend)
5. [Functions That Operate on Iterators](#5-functions-that-operate-on-iterators)
6. [Iterator Adaptors](#6-iterator-adaptors)
7. [Iterator Traits](#7-iterator-traits)

---

# Iterators

## 1. Ranges

A **range** is, conceptually, anything that has a `begin()` and an `end()` — i.e. anything you can iterate over with a range-based `for` loop. Traditionally in the STL a "range" is expressed as a **pair of iterators** `[first, last)`, where `last` is one-past-the-end (a half-open interval).

```cpp
std::vector<int> v = {1, 2, 3, 4, 5};

// A "range" here is expressed as the pair (v.begin(), v.end())
std::sort(v.begin(), v.end());                 // sorts the whole range
std::sort(v.begin(), v.begin() + 3);            // sorts only the first 3 elements — still a valid range
```

### The `[first, last)` Convention (Half-Open Range)

```cpp
// [first, last) means: includes first, excludes last
// This is why an empty range is expressed as first == last:
std::vector<int> empty_range_check = {1,2,3};
auto first = empty_range_check.begin();
auto last  = empty_range_check.begin();   // first == last -> empty range, zero elements
```

### The Ranges Library (C++20)

C++20 introduced `std::ranges`, allowing a *single object* (with `begin()`/`end()`) to be passed directly to algorithms, instead of a pair of iterators — plus composable, lazy "views."

```cpp
#include <ranges>
#include <algorithm>

std::vector<int> v = {5, 3, 1, 4, 2};

// C++20 ranges-based sort — pass the container itself, no need for .begin()/.end()
std::ranges::sort(v);

// Composable views: lazy, pipeline-style transformations
auto result = v
    | std::views::filter([](int n) { return n % 2 == 0; })
    | std::views::transform([](int n) { return n * 10; });

for (int x : result) {
    std::cout << x << " ";   // prints transformed even numbers only
}
```

Ranges are a large topic in their own right (views, adaptors, projections), but the core idea for this reference is: **a range is anything iterable — classically a pair of iterators, and since C++20, optionally a single range object.**

---

## 2. Iterator Categories

Iterators are classified into a **hierarchy of categories** based on which operations they support. Each category refines (adds capabilities to) the one before it.

```
Input Iterator ──┐
                  ├──► Forward Iterator ──► Bidirectional Iterator ──► Random Access Iterator
Output Iterator ──┘
```

### Input Iterator

- **Can:** read values (`*it`), move forward once (`++it`), compare for equality (`==`, `!=`)
- **Cannot:** write, move backward, be re-read after advancing (single-pass only)
- **Example containers/sources:** `std::istream_iterator`

```cpp
#include <iterator>
#include <sstream>

std::istringstream iss("1 2 3 4 5");
std::istream_iterator<int> it(iss), end;

while (it != end) {
    std::cout << *it << " ";  // read-only access
    ++it;                       // can only move forward, single pass
}
```

### Output Iterator

- **Can:** write values (`*it = value`), move forward once (`++it`)
- **Cannot:** read the value back, move backward
- **Example:** `std::ostream_iterator`, `std::back_insert_iterator`

```cpp
#include <iterator>
#include <vector>
#include <algorithm>

std::vector<int> src = {1, 2, 3};
std::ostream_iterator<int> out(std::cout, " ");
std::copy(src.begin(), src.end(), out);  // writes each element, single pass forward
```

### Forward Iterator

- **Can:** everything Input Iterator can, PLUS: multi-pass guaranteed (you can re-read/re-traverse the same range multiple times), can be default-constructed
- **Cannot:** move backward
- **Example containers:** `std::forward_list`, `std::unordered_map`/`unordered_set`

```cpp
std::forward_list<int> fl = {1, 2, 3};
auto it1 = fl.begin();
auto it2 = it1;          // can copy and reuse the same starting point (multi-pass)
++it1;
std::cout << *it1 << " " << *it2 << "\n";   // 2 1 -- both iterators are independently valid
```

### Bidirectional Iterator

- **Can:** everything Forward Iterator can, PLUS: move backward (`--it`)
- **Example containers:** `std::list`, `std::set`, `std::map`

```cpp
std::list<int> l = {1, 2, 3, 4, 5};
auto it = l.end();
--it;                       // move backward — only possible with bidirectional+
std::cout << *it << "\n";  // 5
--it;
std::cout << *it << "\n";  // 4
```

### Random Access Iterator

- **Can:** everything Bidirectional Iterator can, PLUS: jump by arbitrary offsets in O(1) (`it + n`, `it[n]`), compare with `<`, `>`, `<=`, `>=`, subtract two iterators to get a distance
- **Example containers:** `std::vector`, `std::deque`, `std::array`, raw pointers (`T*`)

```cpp
std::vector<int> v = {10, 20, 30, 40, 50};
auto it = v.begin();
it += 3;                          // O(1) jump — only random access iterators can do this
std::cout << *it << "\n";        // 40
std::cout << it[1] << "\n";      // 50 (offset access)
std::cout << (v.end() - v.begin()) << "\n";  // 5 (distance via subtraction)
```

### C++20 Addition: Contiguous Iterator

C++20 adds a further refinement: **Contiguous Iterator**, guaranteeing the underlying elements are laid out contiguously in memory (so `&*(it + n) == &*it + n`). `vector`, `array`, `string`, and raw pointers satisfy this; `deque` does **not** (chunked memory).

### Summary Table

| Category | Read | Write | ++ | -- | + n (O(1)) | Multi-pass | Example |
|---|---|---|---|---|---|---|---|
| Input | ✅ | ❌ | ✅ | ❌ | ❌ | ❌ | `istream_iterator` |
| Output | ❌ | ✅ | ✅ | ❌ | ❌ | ❌ | `ostream_iterator` |
| Forward | ✅ | (✅ if mutable) | ✅ | ❌ | ❌ | ✅ | `forward_list` |
| Bidirectional | ✅ | (✅ if mutable) | ✅ | ✅ | ❌ | ✅ | `list`, `map`, `set` |
| Random Access | ✅ | (✅ if mutable) | ✅ | ✅ | ✅ | ✅ | `vector`, `deque`, `T*` |
| Contiguous (C++20) | ✅ | (✅ if mutable) | ✅ | ✅ | ✅ | ✅ | `vector`, `array`, `string` |

---

## 3. Container begin()/end() Member Functions

Every STL container provides `begin()`/`end()` member functions that return iterators to the first element and to "one past the last" element, respectively, defining the half-open range `[begin(), end())`.

```cpp
std::vector<int> v = {10, 20, 30};

auto it = v.begin();     // iterator to the first element (10)
auto e  = v.end();        // iterator to one-past-the-last element (NOT dereferenceable)

for (auto i = v.begin(); i != v.end(); ++i) {
    std::cout << *i << " ";   // 10 20 30
}
```

### The Const / Reverse / cbegin Family

| Function | Returns | Notes |
|---|---|---|
| `begin()` / `end()` | `iterator` (or `const_iterator` if container is const) | Mutable access (if container isn't const) |
| `cbegin()` / `cend()` (C++11) | `const_iterator` | Always read-only, even on a non-const container |
| `rbegin()` / `rend()` | `reverse_iterator` | Iterates from last element to first |
| `crbegin()` / `crend()` (C++11) | `const_reverse_iterator` | Read-only reverse iteration |

```cpp
std::vector<int> v = {1, 2, 3};

for (auto it = v.cbegin(); it != v.cend(); ++it) {
    // *it = 100;   // ERROR: const_iterator, cannot modify
    std::cout << *it << " ";  // 1 2 3
}

for (auto it = v.rbegin(); it != v.rend(); ++it) {
    std::cout << *it << " ";  // 3 2 1 — reverse order
}
```

### Why a Member Function Instead of a Free Function?

Because `begin()`/`end()` are member functions, each container implements them according to its own internal structure (pointer arithmetic for `vector`, node traversal for `list`/`map`, bucket iteration for `unordered_map`, etc.) while presenting the same uniform interface to generic algorithms.

---

## 4. std::begin / std::end

`std::begin()` and `std::end()` (C++11, `<iterator>`) are **free functions** that work uniformly for both:
1. Any container with member `begin()`/`end()` (they just call `container.begin()`/`container.end()`)
2. Plain **C-style arrays**, which have no member functions at all

```cpp
#include <iterator>

std::vector<int> v = {1, 2, 3};
auto it1 = std::begin(v);   // equivalent to v.begin()
auto it2 = std::end(v);      // equivalent to v.end()

int arr[5] = {10, 20, 30, 40, 50};
auto it3 = std::begin(arr);  // arr, i.e. &arr[0] — works even though arrays have no .begin()!
auto it4 = std::end(arr);     // arr + 5

for (auto it = std::begin(arr); it != std::end(arr); ++it) {
    std::cout << *it << " ";  // 10 20 30 40 50
}
```

### Why This Matters: Writing Generic Code

```cpp
template <typename Container>
void printAll(const Container& c) {
    // works whether c is a std::vector, std::list, or a raw C-array
    for (auto it = std::begin(c); it != std::end(c); ++it) {
        std::cout << *it << " ";
    }
}

int rawArr[3] = {1, 2, 3};
std::vector<int> vec = {4, 5, 6};

printAll(rawArr);  // works — std::begin/end handle raw arrays
printAll(vec);      // works — std::begin/end call vec.begin()/vec.end()
```

This is also exactly the mechanism the range-based `for` loop uses internally: `for (auto& x : container)` is roughly sugar for `for (auto it = std::begin(container); it != std::end(container); ++it) { auto& x = *it; ... }`.

---

## 5. Functions That Operate on Iterators

All defined in `<iterator>`.

### std::next

Returns an iterator advanced by `n` positions (default 1), **without modifying the original iterator**. Works efficiently in O(1) for random-access iterators, O(n) otherwise.

```cpp
std::vector<int> v = {1, 2, 3, 4, 5};
auto it = v.begin();

auto it2 = std::next(it);       // it2 points to 2; it itself is UNCHANGED (still points to 1)
auto it3 = std::next(it, 3);    // it3 points to 4 (advanced by 3)

std::cout << *it  << "\n";  // 1 (unchanged!)
std::cout << *it2 << "\n";  // 2
std::cout << *it3 << "\n";  // 4
```

### std::prev

Returns an iterator moved backward by `n` positions (default 1). Requires at least a **bidirectional** iterator.

```cpp
std::list<int> l = {1, 2, 3, 4, 5};
auto it = l.end();

auto lastElem = std::prev(it);        // one before end() -> points to 5
auto thirdFromEnd = std::prev(it, 3); // points to 3

std::cout << *lastElem << "\n";       // 5
std::cout << *thirdFromEnd << "\n";   // 3
```

### std::iter_swap

Swaps the values pointed to by two iterators (which can even be of different, but compatible, iterator types).

```cpp
std::vector<int> v = {1, 2, 3, 4};
std::iter_swap(v.begin(), v.begin() + 3);   // swaps v[0] and v[3]

for (int x : v) std::cout << x << " ";   // 4 2 3 1
```

### std::advance

**Modifies the iterator itself in place**, moving it forward (or backward, for bidirectional+) by `n`. Unlike `std::next`/`std::prev`, it has no return value.

```cpp
std::list<int> l = {10, 20, 30, 40, 50};
auto it = l.begin();

std::advance(it, 2);       // it is MODIFIED in place, now points to 30
std::cout << *it << "\n"; // 30

std::advance(it, -1);      // moves backward by 1 (bidirectional iterator) -> 20
std::cout << *it << "\n"; // 20
```

### std::distance

Computes the number of increments needed to get from `first` to `last`. O(1) for random-access iterators, O(n) otherwise (it literally walks the range for non-random-access iterators).

```cpp
std::vector<int> v = {1, 2, 3, 4, 5};
std::cout << std::distance(v.begin(), v.end()) << "\n";   // 5 -- O(1), computed via subtraction

std::list<int> l = {1, 2, 3, 4, 5};
std::cout << std::distance(l.begin(), l.end()) << "\n";   // 5 -- O(n), walks the list
```

### Summary Table

| Function | Effect | Return Value | Modifies Original? | Complexity |
|---|---|---|---|---|
| `std::next(it, n=1)` | Advance forward | New iterator | No | O(1) random-access, O(n) otherwise |
| `std::prev(it, n=1)` | Move backward | New iterator | No | O(1) random-access, O(n) otherwise |
| `std::advance(it, n)` | Advance/move by n | `void` | **Yes, in-place** | O(1) random-access, O(n) otherwise |
| `std::distance(first, last)` | Count steps between | `difference_type` | No | O(1) random-access, O(n) otherwise |
| `std::iter_swap(a, b)` | Swap pointed-to values | `void` | Modifies pointed-to *values*, not iterators | O(1) |

---

## 6. Iterator Adaptors

**Iterator adaptors** wrap an existing iterator (or stream, or container) to give it different behavior — e.g. reversing direction, moving instead of copying, or inserting instead of overwriting.

### Stream Iterators

Stream iterators adapt an I/O stream so it can be used with iterator-based STL algorithms.

#### istream_iterator

An **input iterator** that reads formatted values from an `std::istream` one at a time via `operator>>`.

```cpp
#include <iterator>
#include <sstream>
#include <algorithm>

std::istringstream iss("10 20 30 40 50");
std::istream_iterator<int> begin(iss);
std::istream_iterator<int> end;    // default-constructed = "end of stream" sentinel

std::vector<int> v(begin, end);    // reads all ints from the stream into the vector

for (int x : v) std::cout << x << " ";  // 10 20 30 40 50
```

#### ostream_iterator

An **output iterator** that writes each assigned value to an `std::ostream` via `operator<<`, optionally with a delimiter after each element.

```cpp
#include <iterator>
#include <vector>
#include <algorithm>

std::vector<int> v = {1, 2, 3, 4, 5};
std::ostream_iterator<int> out(std::cout, ", ");

std::copy(v.begin(), v.end(), out);   // prints: 1, 2, 3, 4, 5,
```

#### istreambuf_iterator

An **input iterator** that reads raw, unformatted **characters** directly from a stream buffer (`std::streambuf`), bypassing formatted extraction (`operator>>`) — much faster for reading raw text/binary data character-by-character (e.g. whitespace is NOT skipped, unlike `istream_iterator<char>`).

```cpp
#include <iterator>
#include <sstream>
#include <fstream>

std::ifstream file("data.txt");
std::string content((std::istreambuf_iterator<char>(file)),
                      std::istreambuf_iterator<char>());   // reads the ENTIRE file, including whitespace

std::cout << content;
```

#### ostreambuf_iterator

An **output iterator** that writes raw characters directly to a stream buffer, without the formatting overhead of `operator<<`.

```cpp
#include <iterator>
#include <string>

std::string s = "Hello, World!";
std::ostreambuf_iterator<char> out(std::cout);

std::copy(s.begin(), s.end(), out);   // writes each character directly, faster than ostream_iterator<char>
```

---

### Reverse Iterators

`std::reverse_iterator` wraps a bidirectional (or better) iterator so that incrementing it moves **backward** through the underlying sequence. `rbegin()`/`rend()` return this type.

```cpp
#include <iterator>
#include <vector>

std::vector<int> v = {1, 2, 3, 4, 5};

std::reverse_iterator<std::vector<int>::iterator> rit(v.end());
std::reverse_iterator<std::vector<int>::iterator> rend(v.begin());

while (rit != rend) {
    std::cout << *rit << " ";  // 5 4 3 2 1
    ++rit;                      // ++ on a reverse_iterator moves BACKWARD in the underlying sequence
}

// Usually obtained more conveniently via the container:
for (auto it = v.rbegin(); it != v.rend(); ++it) {
    std::cout << *it << " ";  // 5 4 3 2 1
}
```

**Important quirk:** a `reverse_iterator` internally stores the underlying iterator **one position ahead** of what it logically points to; `*rit` is implemented as `*(base() - 1)`. This matters when converting between `reverse_iterator` and the underlying `iterator` (e.g. for `erase()`).

Creating one manually with the helper:

```cpp
auto rit2 = std::make_reverse_iterator(v.end());  // C++14 — cleaner than the constructor above
```

---

### Move Iterators

`std::move_iterator` (C++11) wraps an iterator so that dereferencing it yields an **rvalue reference**, turning what would normally be a copy into a move — useful for transferring elements between containers without duplicating expensive resources.

```cpp
#include <iterator>
#include <vector>
#include <string>

std::vector<std::string> src = {"hello", "world", "foo"};
std::vector<std::string> dst;

// Without move iterators: std::copy would COPY each string (expensive)
// With move iterators: each string is MOVED, leaving src's strings empty
std::copy(std::make_move_iterator(src.begin()),
          std::make_move_iterator(src.end()),
          std::back_inserter(dst));

// After this: dst = {"hello", "world", "foo"}, src's strings are now in a moved-from (empty) state
```

`std::make_move_iterator` (C++11) is the standard way to construct one — cleaner than the raw constructor `std::move_iterator<Iter>(it)`.

A very common real use-case: moving instead of copying when the *source* container's contents won't be needed afterward:

```cpp
std::vector<std::string> extractAll(std::vector<std::string>&& source) {
    return std::vector<std::string>(std::make_move_iterator(source.begin()),
                                      std::make_move_iterator(source.end()));
}
```

---

### Insert Iterators

Insert iterators are **output iterators** that, instead of overwriting an existing element (like a normal container iterator would), **insert** a new element into the container each time they're assigned to — solving the classic problem of `std::copy` requiring pre-sized destination containers.

#### back_insert_iterator

Calls `push_back()` on the underlying container for every assignment. Requires the container to support `push_back` (e.g. `vector`, `deque`, `list`).

```cpp
#include <iterator>
#include <vector>
#include <algorithm>

std::vector<int> src = {1, 2, 3, 4, 5};
std::vector<int> dst;   // starts EMPTY -- no pre-sizing needed!

std::copy(src.begin(), src.end(), std::back_inserter(dst));
// std::back_inserter(dst) creates a back_insert_iterator<std::vector<int>>

for (int x : dst) std::cout << x << " ";  // 1 2 3 4 5
```

Without `back_inserter`, `std::copy` into an empty `dst` would be undefined behavior (writing past the end).

#### front_insert_iterator

Calls `push_front()` on the underlying container for every assignment. Requires `push_front` support (e.g. `deque`, `list` — **not** `vector`, which has no `push_front`).

```cpp
#include <iterator>
#include <deque>
#include <algorithm>

std::vector<int> src = {1, 2, 3};
std::deque<int> dst;

std::copy(src.begin(), src.end(), std::front_inserter(dst));

for (int x : dst) std::cout << x << " ";  // 3 2 1 -- note the REVERSED order!
// (because each new element is pushed to the front, so the last-copied element ends up first)
```

#### insert_iterator

Calls `insert()` at a specified position for every assignment, and **advances that position** after each insertion so subsequent elements are inserted right after the previous one (preserving relative order). Works with any container supporting `insert(iterator, value)`.

```cpp
#include <iterator>
#include <vector>
#include <algorithm>

std::vector<int> src = {10, 20, 30};
std::vector<int> dst = {1, 2, 3, 4, 5};

auto it = dst.begin() + 2;   // insert starting at position 2 (before the third element)
std::copy(src.begin(), src.end(), std::inserter(dst, it));

for (int x : dst) std::cout << x << " ";  // 1 2 10 20 30 3 4 5
```

### Summary Table

| Adaptor | Category | Underlying Requirement | Effect on Assignment |
|---|---|---|---|
| `back_insert_iterator` | Output | Container has `push_back` | Appends to the back |
| `front_insert_iterator` | Output | Container has `push_front` | Prepends to the front |
| `insert_iterator` | Output | Container has `insert(pos, val)` | Inserts at (and advances) a tracked position |
| `reverse_iterator` | Same as wrapped (≥ bidirectional) | Bidirectional+ iterator | Reverses traversal direction |
| `move_iterator` | Same as wrapped | Any iterator | Dereferencing yields an rvalue (enables moves) |
| `istream_iterator` | Input | `std::istream` | Reads formatted values via `>>` |
| `ostream_iterator` | Output | `std::ostream` | Writes values via `<<`, optional delimiter |
| `istreambuf_iterator` | Input | `std::streambuf` | Reads raw characters, unformatted |
| `ostreambuf_iterator` | Output | `std::streambuf` | Writes raw characters, unformatted |

---

## 7. Iterator Traits

`std::iterator_traits<Iterator>` (in `<iterator>`) is a **template that exposes a uniform, standardized set of associated types for any iterator** — allowing generic algorithms to query properties of an iterator (its category, the type it points to, etc.) without needing to know the iterator's concrete type in advance. This is the classic example of a **traits class** in C++.

### The Five Associated Types

```cpp
template <typename Iterator>
struct iterator_traits {
    using difference_type   = typename Iterator::difference_type;    // type for distance between iterators (usually std::ptrdiff_t)
    using value_type         = typename Iterator::value_type;          // the type of element pointed to
    using pointer             = typename Iterator::pointer;              // pointer-to-value_type
    using reference           = typename Iterator::reference;            // reference-to-value_type
    using iterator_category  = typename Iterator::iterator_category;    // tag identifying the category (input/forward/etc.)
};
```

### Why It's Needed: Uniform Access, Including for Raw Pointers

Raw pointers (`T*`) act as random-access iterators but have **no member typedefs** (`T*` has no `T*::value_type`). `iterator_traits` has a **partial specialization for pointer types** so that generic code can treat pointers uniformly with class-type iterators:

```cpp
// Partial specialization built into the standard library for raw pointers:
template <typename T>
struct iterator_traits<T*> {
    using difference_type   = std::ptrdiff_t;
    using value_type         = T;
    using pointer             = T*;
    using reference           = T&;
    using iterator_category  = std::random_access_iterator_tag;
};
```

This means generic algorithm code can be written once and work for both container iterators AND raw pointers:

```cpp
template <typename Iterator>
void printType() {
    using ValueType = typename std::iterator_traits<Iterator>::value_type;
    std::cout << typeid(ValueType).name() << "\n";
}

printType<std::vector<int>::iterator>();   // works: value_type = int
printType<int*>();                          // ALSO works, thanks to the pointer specialization
```

### Iterator Category Tags

These are empty "tag" structs used purely for compile-time dispatch (tag dispatching):

```cpp
struct input_iterator_tag {};
struct output_iterator_tag {};
struct forward_iterator_tag       : public input_iterator_tag {};
struct bidirectional_iterator_tag : public forward_iterator_tag {};
struct random_access_iterator_tag : public bidirectional_iterator_tag {};
// C++20 additionally: struct contiguous_iterator_tag : public random_access_iterator_tag {};
```

The inheritance hierarchy means a function accepting `forward_iterator_tag` will also accept a `random_access_iterator_tag` argument (it "is-a" forward iterator too), enabling clean tag-dispatch based overload resolution.

### Practical Example: Tag Dispatch — Implementing `std::advance` Efficiently

This is the canonical real-world example of why `iterator_traits` exists: `std::advance` behaves completely differently (O(1) vs O(n)) depending on the iterator category, and `iterator_traits` lets it pick the right implementation at **compile time**, with **zero runtime overhead**.

```cpp
#include <iterator>

// O(n) version — for input/forward/bidirectional iterators, must step one at a time
template <typename Iterator, typename Distance>
void advance_impl(Iterator& it, Distance n, std::input_iterator_tag) {
    while (n > 0) { ++it; --n; }   // works for input_iterator_tag and anything derived from it
}

// O(1) version — only enabled for random-access iterators, jumps directly
template <typename Iterator, typename Distance>
void advance_impl(Iterator& it, Distance n, std::random_access_iterator_tag) {
    it += n;   // random_access_iterator_tag is more specialized, so THIS overload wins for vector/deque/T*
}

// Dispatcher: picks the right overload based on the iterator's category, resolved at compile time
template <typename Iterator, typename Distance>
void myAdvance(Iterator& it, Distance n) {
    advance_impl(it, n, typename std::iterator_traits<Iterator>::iterator_category{});
}

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5};
    auto vit = v.begin();
    myAdvance(vit, 3);              // dispatches to the O(1) random_access_iterator_tag overload

    std::list<int> l = {1, 2, 3, 4, 5};
    auto lit = l.begin();
    myAdvance(lit, 3);               // dispatches to the O(n) input_iterator_tag overload (list is bidirectional, inherits from input)
}
```

### Checking an Iterator's Category at Compile Time

```cpp
template <typename Iterator>
constexpr bool isRandomAccess =
    std::is_same_v<typename std::iterator_traits<Iterator>::iterator_category,
                    std::random_access_iterator_tag>;

static_assert(isRandomAccess<std::vector<int>::iterator> == true);
static_assert(isRandomAccess<std::list<int>::iterator>   == false);
```

### C++20 Note: Concepts Largely Supersede Manual Tag Dispatch

C++20 introduces iterator **concepts** (`std::input_iterator`, `std::forward_iterator`, `std::bidirectional_iterator`, `std::random_access_iterator`, `std::contiguous_iterator` in `<iterator>`), which express the same category hierarchy but as constraints usable directly with `if constexpr`/`requires`, often making explicit `iterator_traits`-based tag dispatch unnecessary in new code:

```cpp
template <typename Iterator>
void myAdvance(Iterator& it, std::iter_difference_t<Iterator> n) {
    if constexpr (std::random_access_iterator<Iterator>) {
        it += n;
    } else {
        while (n > 0) { ++it; --n; }
    }
}
```
