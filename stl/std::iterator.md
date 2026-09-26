# `<iterator>`

### Notation
* `c`: a container (or C array). `it`, `first`, `last`: iterators. `n`: a signed distance/count.
* `back_insert_iterator`, `front_insert_iterator`, `insert_iterator` are the types returned by the three inserter adapters below — not typically named/constructed directly by users, so they're listed under Methods, not Constructors.

---

### Methods

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| `back_insert_iterator` | back_inserter | `std::back_inserter(c)` | O(1) | Adapter that calls `.push_back()` on assignment — lets algorithms like `std::copy` append into a container. |
| `front_insert_iterator` | front_inserter | `std::front_inserter(c)` | O(1) | Same idea, but calls `.push_front()` (only valid for containers that support it, e.g. `list`, `deque`). |
| `insert_iterator` | inserter | `std::inserter(c, it)` | O(1) | Inserts at an arbitrary position via `.insert(it, ...)`, advancing `it` after each insertion. |
| `difference_type` | distance | `std::distance(first, last)` | O(1) random-access, O(n) otherwise | Number of elements between two iterators. |
| `void` | advance | `std::advance(it, n)` | O(1) random-access, O(n) otherwise | Moves an iterator forward (or backward, if `n < 0` and the iterator is at least bidirectional) by `n`, using the fastest method available for the iterator category. |
| `It` | next / prev | `std::next(it, n = 1)`<br>`std::prev(it, n = 1)` | O(1) random-access, O(n) otherwise | Non-mutating versions of `advance` — return a new iterator without changing `it`. |
| `It` | begin / end | `std::begin(c)` / `std::end(c)` | O(1) | Free-function form; works uniformly on containers and C arrays (unlike `c.begin()`, which arrays don't have). |
| `size_t` | size | `std::size(c)` | O(1) | Free-function form of `.size()`; also works on C arrays. |
| `bool` | empty | `std::empty(c)` | O(1) | Free-function form of `.empty()`; also works on C arrays (always `false`) and `initializer_list`. |
| `T*` | data | `std::data(c)` | O(1) | Free-function form of `.data()`; also works on C arrays. |
| `ptrdiff_t` | ssize (C++20) | `std::ssize(c)` | O(1) | Like `size(c)`, but returns a **signed** type — avoids signed/unsigned comparison warnings against `int` loop counters. |
| *(traits struct)* | iterator_traits\<It\> | `iterator_traits<It>::value_type` (etc.) | O(1) (compile-time) | Exposes an iterator's `value_type`, `difference_type`, `pointer`, `reference`, `iterator_category` — used in generic template code. |
| *(adapter class)* | istream_iterator\<T\> / ostream_iterator\<T\> | `istream_iterator<T>(cin)`<br>`ostream_iterator<T>(cout, " ")` | O(1) per element | Adapts a stream into an iterator range, e.g. for reading whitespace-separated values directly with algorithms like `std::copy`. |
| *(adapter class)* | istreambuf_iterator / ostreambuf_iterator | `istreambuf_iterator<char>(cin)` | O(1) per char | Lower-level than `istream_iterator` — iterates raw characters from a stream buffer, bypassing formatted extraction (`>>`). |
| *(adapter class)* | reverse_iterator | `make_reverse_iterator(it)` | O(1) | Wraps a bidirectional+ iterator to walk it backwards; `rbegin()`/`rend()` return this type. |
| *(adapter class)* | move_iterator / make_move_iterator | `make_move_iterator(it)` | O(1) | Wraps an iterator so that dereferencing yields an rvalue reference — makes an algorithm like `std::copy` move elements instead of copying them. |

---

### Iterator Categories

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| — | Input / Output | *(tag types, not called directly)* | N/A | Single-pass read-only (`InputIterator`) / write-only (`OutputIterator`) traversal — e.g. `istream_iterator`. |
| — | Forward | *(tag type)* | N/A | Multi-pass, read (and possibly write), only `++`. E.g. `forward_list::iterator`. |
| — | Bidirectional | *(tag type)* | N/A | Adds `--` (can walk backward too). E.g. `list::iterator`, `map::iterator`. |
| — | Random Access | *(tag type)* | N/A | Adds `+n`/`-n`/`[]`/`<`/`>` in O(1) — arbitrary jumps. E.g. `vector::iterator`, raw pointers. |
| — | Contiguous (C++20) | *(tag type)* | N/A | Strengthens Random Access with the guarantee that elements are laid out contiguously in memory (so `&*(it + n) == &*it + n`). E.g. `vector`, `array`, `string`. |

> A category determines which operations (like `it + n` or `--it`) are valid, and how efficient `distance`/`advance` will be for that iterator.
