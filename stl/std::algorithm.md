# C++ Standard Library Algorithms (`<algorithm>`)

### Notation
* `b`, `e`: Begin / End iterators
* `d_b`, `d_e`: Destination Begin / Destination End iterators
* `p`: Predicate function
* `f`, `op`: Unary/Binary function or operation
* `val`: Value
* `It`, `OutIt`: Return Iterator types
* `N`: Distance between iterators (`std::distance(b, e)`)
* Iterator category annotations in parentheses next to each parameter show the **minimum** iterator type required: `InIt` < `FwdIt` < `BidirIt` < `RandIt` (each higher category also satisfies the requirements of the ones before it). `OutIt` is a separate, write-only category used for destination parameters.

---

## 1. Non-Modifying Sequence Operations

| Return Type | Algorithm Signature | Big O Complexity | Description |
| :--- | :--- | :--- | :--- |
| `bool` | `all_of(b(InIt), e, p)` | $O(N)$ | Returns true if predicate `p` is true for all elements. |
| `bool` | `any_of(b(InIt), e, p)` | $O(N)$ | Returns true if predicate `p` is true for at least one element. |
| `bool` | `none_of(b(InIt), e, p)` | $O(N)$ | Returns true if predicate `p` is false for all elements. |
| `Func` | `for_each(b(InIt), e, f)` | $O(N)$ | Applies function `f` to every element in the range. |
| `It` | `for_each_n(b(InIt), n, f)` | $O(n)$ | Applies function `f` to the first `n` elements. |
| `ptrdiff_t` | `count(b(InIt), e, val)` | $O(N)$ | Counts elements equal to `val`. |
| `ptrdiff_t` | `count_if(b(InIt), e, p)` | $O(N)$ | Counts elements for which predicate `p` is true. |
| `pair` | `mismatch(b1(InIt), e1, b2(InIt))` | $O(N)$ | Finds the first position where two ranges differ. |
| `bool` | `equal(b1(InIt), e1, b2(InIt))` | $O(N)$ | Checks if two ranges are element-wise equal. |
| `It` | `find(b(InIt), e, val)` | $O(N)$ | Finds the first element equal to `val`. |
| `It` | `find_if(b(InIt), e, p)` | $O(N)$ | Finds the first element for which predicate `p` is true. |
| `It` | `find_if_not(b(InIt), e, p)` | $O(N)$ | Finds the first element for which predicate `p` is false. |
| `It` | `find_end(b1(FwdIt), e1, b2(FwdIt), e2)` | $O(N \cdot M)$ | Finds the last occurrence of a sequence in a range. |
| `It` | `find_first_of(b1(InIt), e1, b2(FwdIt), e2)` | $O(N \cdot M)$ | Finds the first element matching any element in second range. |
| `It` | `adjacent_find(b(FwdIt), e)` | $O(N)$ | Finds the first two adjacent equal elements. |
| `It` | `search(b1(FwdIt), e1, b2(FwdIt), e2)` | $O(N \cdot M)$ | Searches for the first occurrence of a sequence. |
| `It` | `search_n(b(FwdIt), e, count, val)` | $O(N)$ | Searches for `count` consecutive copies of `val`. |

---

## 2. Modifying Sequence Operations

| Return Type | Algorithm Signature | Big O Complexity | Description |
| :--- | :--- | :--- | :--- |
| `OutIt` | `copy(b(InIt), e, d_b(OutIt))` | $O(N)$ | Copies a range of elements to a new location. |
| `OutIt` | `copy_n(b(InIt), n, d_b(OutIt))` | $O(n)$ | Copies `n` elements to a new location. |
| `OutIt` | `copy_if(b(InIt), e, d_b(OutIt), p)` | $O(N)$ | Copies elements for which predicate `p` is true. |
| `BidirIt` | `copy_backward(b(BidirIt), e, d_e(BidirIt))` | $O(N)$ | Copies a range starting from the end. |
| `OutIt` | `move(b(InIt), e, d_b(OutIt))` | $O(N)$ | Moves a range of elements to a new location. |
| `BidirIt` | `move_backward(b(BidirIt), e, d_e(BidirIt))` | $O(N)$ | Moves a range starting from the end. |
| `void` | `fill(b(FwdIt), e, val)` | $O(N)$ | Assigns `val` to all elements in a range. |
| `OutIt` | `fill_n(b(OutIt), n, val)` | $O(n)$ | Assigns `val` to `n` elements. |
| `OutIt` | `transform(b(InIt), e, d_b(OutIt), op)` | $O(N)$ | Applies unary operation and stores the result. |
| `void` | `generate(b(FwdIt), e, g)` | $O(N)$ | Assigns the result of generator `g` to a range. |
| `OutIt` | `generate_n(b(OutIt), n, g)` | $O(n)$ | Assigns the result of generator `g` to `n` elements. |
| `It` | `remove(b(FwdIt), e, val)` | $O(N)$ | Logically removes all elements equal to `val`. |
| `It` | `remove_if(b(FwdIt), e, p)` | $O(N)$ | Logically removes all elements matching predicate `p`. |
| `OutIt` | `remove_copy(b(InIt), e, d_b(OutIt), val)` | $O(N)$ | Copies range omitting elements equal to `val`. |
| `OutIt` | `remove_copy_if(b(InIt), e, d_b(OutIt), p)` | $O(N)$ | Copies range omitting elements matching `p`. |
| `void` | `replace(b(FwdIt), e, old_v, new_v)` | $O(N)$ | Replaces all `old_v` with `new_v`. |
| `void` | `replace_if(b(FwdIt), e, p, new_v)` | $O(N)$ | Replaces values matching `p` with `new_v`. |
| `OutIt` | `replace_copy(b(InIt), e, d_b(OutIt), old_v, new_v)` | $O(N)$ | Copies range, replacing `old_v` with `new_v`. |
| `OutIt` | `replace_copy_if(b(InIt), e, d_b(OutIt), p, new_v)` | $O(N)$ | Copies range, replacing matching values with `new_v`. |
| `void` | `swap(a, b)` | $O(1)$ | Swaps the values of two objects. |
| `OutIt` | `swap_ranges(b1(FwdIt), e1, b2(FwdIt))` | $O(N)$ | Swaps elements between two equal-sized ranges. |
| `void` | `iter_swap(a(FwdIt), b(FwdIt))` | $O(1)$ | Swaps the elements pointed to by two iterators. |
| `void` | `reverse(b(BidirIt), e)` | $O(N)$ | Reverses the order of elements in a range. |
| `OutIt` | `reverse_copy(b(BidirIt), e, d_b(OutIt))` | $O(N)$ | Copies a range in reverse order. |
| `It` | `rotate(b(FwdIt), mid, e)` | $O(N)$ | Rotates elements left around a pivot point (`mid`). |
| `OutIt` | `rotate_copy(b(FwdIt), mid, e, d_b(OutIt))` | $O(N)$ | Copies a rotated sequence to a destination. |
| `It` | `shift_left(b(FwdIt), e, n)` | $O(N)$ | Shifts elements left by `n` positions (C++20). |
| `It` | `shift_right(b(FwdIt), e, n)` | $O(N)$ | Shifts elements right by `n` positions (C++20). |
| `It` | `unique(b(FwdIt), e)` | $O(N)$ | Removes consecutive duplicate elements. |
| `OutIt` | `unique_copy(b(InIt), e, d_b(OutIt))` | $O(N)$ | Copies range, omitting consecutive duplicates. |

---

## 3. Partitioning Operations

| Return Type | Algorithm Signature | Big O Complexity | Description |
| :--- | :--- | :--- | :--- |
| `bool` | `is_partitioned(b(InIt), e, p)` | $O(N)$ | Checks if a range is partitioned by predicate `p`. |
| `It` | `partition(b(FwdIt), e, p)` | $O(N)$ | Divides range into matching and non-matching elements. |
| `BidirIt` | `stable_partition(b(BidirIt), e, p)` | $O(N \log N)$ | Partitions range while preserving original relative order. |
| `pair` | `partition_copy(b(InIt), e, d_true(OutIt), d_false(OutIt), p)` | $O(N)$ | Copies elements into two separate destinations based on `p`. |
| `It` | `partition_point(b(FwdIt), e, p)` | $O(\log N)$ | Finds the boundary of a partitioned range. |

---

## 4. Sorting Operations

| Return Type | Algorithm Signature | Big O Complexity | Description |
| :--- | :--- | :--- | :--- |
| `bool` | `is_sorted(b(FwdIt), e)` | $O(N)$ | Checks if the elements are sorted in ascending order. |
| `It` | `is_sorted_until(b(FwdIt), e)` | $O(N)$ | Finds the first unsorted element. |
| `void` | `sort(b(RandIt), e)` | $O(N \log N)$ | Sorts a range in ascending order. |
| `void` | `stable_sort(b(RandIt), e)` | $O(N \log^2 N)$ | Sorts range while preserving equal element order. |
| `void` | `partial_sort(b(RandIt), mid, e)` | $O(N \log M)$ | Sorts the elements up to `mid` ($M = \text{distance}(b, mid)$). |
| `It` | `partial_sort_copy(b(InIt), e, d_b(RandIt), d_e)` | $O(N \log M)$ | Sorts elements during a copy operation. |
| `void` | `nth_element(b(RandIt), nth, e)` | $O(N)$ | Partitions elements around the `nth` element. |

---

## 5. Binary Search Operations (Sorted Ranges)

| Return Type | Algorithm Signature | Big O Complexity | Description |
| :--- | :--- | :--- | :--- |
| `It` | `lower_bound(b(FwdIt), e, val)` | $O(\log N)$ | Finds the first element not less than `val` (>= `val`). |
| `It` | `upper_bound(b(FwdIt), e, val)` | $O(\log N)$ | Finds the first element greater than `val` (> `val`). |
| `bool` | `binary_search(b(FwdIt), e, val)` | $O(\log N)$ | Checks if `val` exists in a sorted range. |
| `pair` | `equal_range(b(FwdIt), e, val)` | $O(\log N)$ | Returns lower and upper bounds as a `std::pair`. |

> Note: `lower_bound`/`upper_bound`/`binary_search`/`equal_range` only need `FwdIt` for correctness (comparisons stay $O(\log N)$), but with anything less than `RandIt` the *traversal* between comparisons becomes $O(N)$ per step, so overall walltime is only truly $O(\log N)$ when given a `RandIt` (e.g. `vector`, `deque`).

---

## 6. Set & Merge Operations (Sorted Ranges)

| Return Type | Algorithm Signature | Big O Complexity | Description |
| :--- | :--- | :--- | :--- |
| `bool` | `includes(b1(InIt), e1, b2(InIt), e2)` | $O(N + M)$ | Checks if one sorted range contains another. |
| `OutIt` | `set_union(b1(InIt), e1, b2(InIt), e2, d_b(OutIt))` | $O(N + M)$ | Computes the union of two sorted ranges. |
| `OutIt` | `set_intersection(b1(InIt), e1, b2(InIt), e2, d_b(OutIt))` | $O(N + M)$ | Computes the intersection of two sorted ranges. |
| `OutIt` | `set_difference(b1(InIt), e1, b2(InIt), e2, d_b(OutIt))` | $O(N + M)$ | Computes the difference of two sorted ranges. |
| `OutIt` | `set_symmetric_difference(b1(InIt), e1, b2(InIt), e2, d_b(OutIt))` | $O(N + M)$ | Computes the symmetric difference of two ranges. |
| `OutIt` | `merge(b1(InIt), e1, b2(InIt), e2, d_b(OutIt))` | $O(N + M)$ | Merges two sorted ranges into one sorted destination range. |
| `void` | `inplace_merge(b(BidirIt), mid, e)` | $O(N \log N)$ (or $O(N)$ with enough extra memory) | Merges two consecutive sorted sub-ranges `[b, mid)` and `[mid, e)` in place. |

---

## 7. Heap Operations

| Return Type | Algorithm Signature | Big O Complexity | Description |
| :--- | :--- | :--- | :--- |
| `bool` | `is_heap(b(RandIt), e)` | $O(N)$ | Checks if the range is a max-heap. |
| `It` | `is_heap_until(b(RandIt), e)` | $O(N)$ | Finds the first element violating the heap property. |
| `void` | `make_heap(b(RandIt), e)` | $O(N)$ | Rearranges a range into a max-heap. |
| `void` | `push_heap(b(RandIt), e)` | $O(\log N)$ | Adds the last element into the max-heap structure. |
| `void` | `pop_heap(b(RandIt), e)` | $O(\log N)$ | Swaps the max element to the end and restores the heap. |
| `void` | `sort_heap(b(RandIt), e)` | $O(N \log N)$ | Converts a max-heap into a sorted range. |

---

## 8. Minimum/Maximum Operations

| Return Type | Algorithm Signature | Big O Complexity | Description |
| :--- | :--- | :--- | :--- |
| `T` | `min(a, b)` | $O(1)$ | Returns the smaller value. |
| `T` | `max(a, b)` | $O(1)$ | Returns the larger value. |
| `pair` | `minmax(a, b)` | $O(1)$ | Returns both the smaller and larger values as a pair. |
| `It` | `min_element(b(FwdIt), e)` | $O(N)$ | Finds the smallest element in a range. |
| `It` | `max_element(b(FwdIt), e)` | $O(N)$ | Finds the largest element in a range. |
| `pair` | `minmax_element(b(FwdIt), e)` | $O(N)$ | Finds both smallest and largest elements. |
| `T` | `clamp(val, lo, hi)` | $O(1)$ | Clamps `val` between `lo` and `hi`. |

---

## 9. Permutation Operations

| Return Type | Algorithm Signature | Big O Complexity | Description |
| :--- | :--- | :--- | :--- |
| `bool` | `is_permutation(b1(FwdIt), e1, b2(FwdIt))` | $O(N^2)$ | Checks if a range is a permutation of another. |
| `bool` | `next_permutation(b(BidirIt), e)` | $O(N)$ | Generates the next lexicographical permutation. |
| `bool` | `prev_permutation(b(BidirIt), e)` | $O(N)$ | Generates the previous lexicographical permutation. |

---

## 10. Comparison Operations

| Return Type | Algorithm Signature | Big O Complexity | Description |
| :--- | :--- | :--- | :--- |
| `bool` | `lexicographical_compare(b1(InIt), e1, b2(InIt), e2)` | $O(\min(N, M))$ | Checks if range1 is lexicographically less than range2. |
| `auto` | `lexicographical_compare_three_way(b1(InIt), e1, b2(InIt), e2)` | $O(\min(N, M))$ | Compares two ranges using the three-way comparison operator (C++20). |

---

## 11. Random Sampling Operations (C++17)

| Return Type | Algorithm Signature | Big O Complexity | Description |
| :--- | :--- | :--- | :--- |
| `SampleIt` | `sample(b(FwdIt), e, d_b(OutIt), n, gen)` | $O(N)$ | Randomly selects `min(n, N)` elements from `[b, e)` without replacement and writes them to `d_b`, using `gen` as the random number generator. |
