# C++ Standard Library Algorithms (`<algorithm>`)

### Notation
* `b`, `e`: Begin / End iterators
* `d_b`, `d_e`: Destination Begin / Destination End iterators
* `p`: Predicate function
* `f`, `op`: Unary/Binary function or operation
* `val`: Value
* `It`, `OutIt`: Return Iterator types
* `N`: Distance between iterators (`std::distance(b, e)`)

---

## 1. Non-Modifying Sequence Operations

| Return Type | Algorithm Signature | Big O Complexity | Description |
| :--- | :--- | :--- | :--- |
| `bool` | `all_of(b, e, p)` | $O(N)$ | Returns true if predicate `p` is true for all elements. |
| `bool` | `any_of(b, e, p)` | $O(N)$ | Returns true if predicate `p` is true for at least one element. |
| `bool` | `none_of(b, e, p)` | $O(N)$ | Returns true if predicate `p` is false for all elements. |
| `Func` | `for_each(b, e, f)` | $O(N)$ | Applies function `f` to every element in the range. |
| `It` | `for_each_n(b, n, f)` | $O(n)$ | Applies function `f` to the first `n` elements. |
| `ptrdiff_t` | `count(b, e, val)` | $O(N)$ | Counts elements equal to `val`. |
| `ptrdiff_t` | `count_if(b, e, p)` | $O(N)$ | Counts elements for which predicate `p` is true. |
| `pair` | `mismatch(b1, e1, b2)` | $O(N)$ | Finds the first position where two ranges differ. |
| `It` | `find(b, e, val)` | $O(N)$ | Finds the first element equal to `val`. |
| `It` | `find_if(b, e, p)` | $O(N)$ | Finds the first element for which predicate `p` is true. |
| `It` | `find_if_not(b, e, p)` | $O(N)$ | Finds the first element for which predicate `p` is false. |
| `It` | `find_end(b1, e1, b2, e2)` | $O(N \cdot M)$ | Finds the last occurrence of a sequence in a range. |
| `It` | `find_first_of(b1, e1, b2, e2)` | $O(N \cdot M)$ | Finds the first element matching any element in second range. |
| `It` | `adjacent_find(b, e)` | $O(N)$ | Finds the first two adjacent equal elements. |
| `It` | `search(b1, e1, b2, e2)` | $O(N \cdot M)$ | Searches for the first occurrence of a sequence. |
| `It` | `search_n(b, e, count, val)` | $O(N)$ | Searches for `count` consecutive copies of `val`. |

---

## 2. Modifying Sequence Operations

| Return Type | Algorithm Signature | Big O Complexity | Description |
| :--- | :--- | :--- | :--- |
| `OutIt` | `copy(b, e, d_b)` | $O(N)$ | Copies a range of elements to a new location. |
| `OutIt` | `copy_n(b, n, d_b)` | $O(n)$ | Copies `n` elements to a new location. |
| `OutIt` | `copy_if(b, e, d_b, p)` | $O(N)$ | Copies elements for which predicate `p` is true. |
| `BidirIt` | `copy_backward(b, e, d_e)` | $O(N)$ | Copies a range starting from the end. |
| `OutIt` | `move(b, e, d_b)` | $O(N)$ | Moves a range of elements to a new location. |
| `BidirIt` | `move_backward(b, e, d_e)` | $O(N)$ | Moves a range starting from the end. |
| `void` | `fill(b, e, val)` | $O(N)$ | Assigns `val` to all elements in a range. |
| `OutIt` | `fill_n(b, n, val)` | $O(n)$ | Assigns `val` to `n` elements. |
| `OutIt` | `transform(b, e, d_b, op)` | $O(N)$ | Applies unary operation and stores the result. |
| `void` | `generate(b, e, g)` | $O(N)$ | Assigns the result of generator `g` to a range. |
| `OutIt` | `generate_n(b, n, g)` | $O(n)$ | Assigns the result of generator `g` to `n` elements. |
| `It` | `remove(b, e, val)` | $O(N)$ | Logically removes all elements equal to `val`. |
| `It` | `remove_if(b, e, p)` | $O(N)$ | Logically removes all elements matching predicate `p`. |
| `OutIt` | `remove_copy(b, e, d_b, val)` | $O(N)$ | Copies range omitting elements equal to `val`. |
| `OutIt` | `remove_copy_if(b, e, d_b, p)` | $O(N)$ | Copies range omitting elements matching `p`. |
| `void` | `replace(b, e, old_v, new_v)` | $O(N)$ | Replaces all `old_v` with `new_v`. |
| `void` | `replace_if(b, e, p, new_v)` | $O(N)$ | Replaces values matching `p` with `new_v`. |
| `OutIt` | `replace_copy(b, e, d_b, old_v, new_v)` | $O(N)$ | Copies range, replacing `old_v` with `new_v`. |
| `OutIt` | `replace_copy_if(b, e, d_b, p, new_v)` | $O(N)$ | Copies range, replacing matching values with `new_v`. |
| `void` | `swap(a, b)` | $O(1)$ | Swaps the values of two objects. |
| `OutIt` | `swap_ranges(b1, e1, b2)` | $O(N)$ | Swaps elements between two equal-sized ranges. |
| `void` | `iter_swap(a, b)` | $O(1)$ | Swaps the elements pointed to by two iterators. |
| `void` | `reverse(b, e)` | $O(N)$ | Reverses the order of elements in a range. |
| `OutIt` | `reverse_copy(b, e, d_b)` | $O(N)$ | Copies a range in reverse order. |
| `It` | `rotate(b, mid, e)` | $O(N)$ | Rotates elements left around a pivot point (`mid`). |
| `OutIt` | `rotate_copy(b, mid, e, d_b)` | $O(N)$ | Copies a rotated sequence to a destination. |
| `It` | `shift_left(b, e, n)` | $O(N)$ | Shifts elements left by `n` positions (C++20). |
| `It` | `shift_right(b, e, n)` | $O(N)$ | Shifts elements right by `n` positions (C++20). |
| `It` | `unique(b, e)` | $O(N)$ | Removes consecutive duplicate elements. |
| `OutIt` | `unique_copy(b, e, d_b)` | $O(N)$ | Copies range, omitting consecutive duplicates. |

---

## 3. Partitioning Operations

| Return Type | Algorithm Signature | Big O Complexity | Description |
| :--- | :--- | :--- | :--- |
| `bool` | `is_partitioned(b, e, p)` | $O(N)$ | Checks if a range is partitioned by predicate `p`. |
| `It` | `partition(b, e, p)` | $O(N)$ | Divides range into matching and non-matching elements. |
| `It` | `stable_partition(b, e, p)` | $O(N \log N)$ | Partitions range while preserving original relative order. |
| `pair` | `partition_copy(b, e, d_true, d_false, p)` | $O(N)$ | Copies elements into two separate destinations based on `p`. |
| `It` | `partition_point(b, e, p)` | $O(\log N)$ | Finds the boundary of a partitioned range. |

---

## 4. Sorting Operations

| Return Type | Algorithm Signature | Big O Complexity | Description |
| :--- | :--- | :--- | :--- |
| `bool` | `is_sorted(b, e)` | $O(N)$ | Checks if the elements are sorted in ascending order. |
| `It` | `is_sorted_until(b, e)` | $O(N)$ | Finds the first unsorted element. |
| `void` | `sort(b, e)` | $O(N \log N)$ | Sorts a range in ascending order. |
| `void` | `stable_sort(b, e)` | $O(N \log^2 N)$ | Sorts range while preserving equal element order. |
| `void` | `partial_sort(b, mid, e)` | $O(N \log M)$ | Sorts the elements up to `mid` ($M = \text{distance}(b, mid)$). |
| `It` | `partial_sort_copy(b, e, d_b, d_e)` | $O(N \log M)$ | Sorts elements during a copy operation. |
| `void` | `nth_element(b, nth, e)` | $O(N)$ | Partitions elements around the `nth` element. |

---

## 5. Binary Search Operations (Sorted Ranges)

| Return Type | Algorithm Signature | Big O Complexity | Description |
| :--- | :--- | :--- | :--- |
| `It` | `lower_bound(b, e, val)` | $O(\log N)$ | Finds the first element not less than `val` (>= `val`). |
| `It` | `upper_bound(b, e, val)` | $O(\log N)$ | Finds the first element greater than `val` (> `val`). |
| `bool` | `binary_search(b, e, val)` | $O(\log N)$ | Checks if `val` exists in a sorted range. |
| `pair` | `equal_range(b, e, val)` | $O(\log N)$ | Returns lower and upper bounds as a `std::pair`. |

---

## 6. Set Operations (Sorted Ranges)

| Return Type | Algorithm Signature | Big O Complexity | Description |
| :--- | :--- | :--- | :--- |
| `bool` | `includes(b1, e1, b2, e2)` | $O(N + M)$ | Checks if one sorted range contains another. |
| `OutIt` | `set_union(b1, e1, b2, e2, d_b)` | $O(N + M)$ | Computes the union of two sorted ranges. |
| `OutIt` | `set_intersection(b1, e1, b2, e2, d_b)` | $O(N + M)$ | Computes the intersection of two sorted ranges. |
| `OutIt` | `set_difference(b1, e1, b2, e2, d_b)` | $O(N + M)$ | Computes the difference of two sorted ranges. |
| `OutIt` | `set_symmetric_difference(b1, e1, b2, e2, d_b)` | $O(N + M)$ | Computes the symmetric difference of two ranges. |

---

## 7. Heap Operations

| Return Type | Algorithm Signature | Big O Complexity | Description |
| :--- | :--- | :--- | :--- |
| `bool` | `is_heap(b, e)` | $O(N)$ | Checks if the range is a max-heap. |
| `It` | `is_heap_until(b, e)` | $O(N)$ | Finds the first element violating the heap property. |
| `void` | `make_heap(b, e)` | $O(N)$ | Rearranges a range into a max-heap. |
| `void` | `push_heap(b, e)` | $O(\log N)$ | Adds the last element into the max-heap structure. |
| `void` | `pop_heap(b, e)` | $O(\log N)$ | Swaps the max element to the end and restores the heap. |
| `void` | `sort_heap(b, e)` | $O(N \log N)$ | Converts a max-heap into a sorted range. |

---

## 8. Minimum/Maximum Operations

| Return Type | Algorithm Signature | Big O Complexity | Description |
| :--- | :--- | :--- | :--- |
| `T` | `min(a, b)` | $O(1)$ | Returns the smaller value. |
| `T` | `max(a, b)` | $O(1)$ | Returns the larger value. |
| `pair` | `minmax(a, b)` | $O(1)$ | Returns both the smaller and larger values as a pair. |
| `It` | `min_element(b, e)` | $O(N)$ | Finds the smallest element in a range. |
| `It` | `max_element(b, e)` | $O(N)$ | Finds the largest element in a range. |
| `pair` | `minmax_element(b, e)` | $O(N)$ | Finds both smallest and largest elements. |
| `T` | `clamp(val, lo, hi)` | $O(1)$ | Clamps `val` between `lo` and `hi`. |

---

## 9. Permutation Operations

| Return Type | Algorithm Signature | Big O Complexity | Description |
| :--- | :--- | :--- | :--- |
| `bool` | `is_permutation(b1, e1, b2)` | $O(N^2)$ | Checks if a range is a permutation of another. |
| `bool` | `next_permutation(b, e)` | $O(N)$ | Generates the next lexicographical permutation. |
| `bool` | `prev_permutation(b, e)` | $O(N)$ | Generates the previous lexicographical permutation. |

---

## 10. Comparison Operations

| Return Type | Algorithm Signature | Big O Complexity | Description |
| :--- | :--- | :--- | :--- |
| `bool` | `lexicographical_compare(b1, e1, b2, e2)` | $O(\min(N, M))$ | Checks if range1 is lexicographically less than range2. |
| `auto` | `lexicographical_compare_three_way(b1, e1, b2, e2)` | $O(\min(N, M))$ | Compares two ranges using the three-way comparison operator (C++20). |
