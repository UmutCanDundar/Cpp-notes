# `<iterator>`

| API | Priority | Description |
|-----|----------|-------------|
| `std::back_inserter(container)` | memorize | Adapter that calls `.push_back()` on assignment — lets algorithms like `std::copy` append to a container. |
| `std::front_inserter(container)` | know | Same idea but calls `.push_front()`. |
| `std::inserter(container, it)` | know | Inserts at an arbitrary position via `.insert()`. |
| `std::distance(first, last)` | memorize | Number of elements between two iterators. O(1) for random-access, O(n) otherwise. |
| `std::advance(it, n)` | memorize | Moves an iterator forward (or backward) by n, using the fastest method available for the iterator category. |
| `std::next(it, n)` / `std::prev(it, n)` | memorize | Non-mutating versions of advance — return a new iterator without changing `it`. |
| `std::begin(c)` / `std::end(c)` | know | Free-function form, works uniformly on containers and C arrays. |
| `std::size(c)` | know | Free-function form of `.size()`, also works on C arrays. |
| `iterator_traits<It>` | know | Exposes an iterator's `value_type`, `difference_type`, `iterator_category` — used in generic template code. |
| `std::istream_iterator<T>` / `std::ostream_iterator<T>` | know | Adapt a stream into an iterator range, e.g. for reading whitespace-separated values with algorithms. |
| Iterator categories (input/forward/bidirectional/random_access) | know | Determine which algorithms/operations (like `+n` or `--it`) are valid and how efficient they are. |
