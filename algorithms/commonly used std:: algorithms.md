# `<algorithm>` — Most Commonly Used

| API | Priority | Description |
|-----|----------|-------------|
| `std::sort(b, e)` | memorize | Unstable, O(n log n). Not on hot path. |
| `std::lower_bound(b, e, val)` | memorize | O(log n) search in sorted array. For finding the first element >= val. |
| `std::upper_bound(b, e, val)` | memorize | For finding the first element > val. |
| `std::binary_search(b, e, val)` | memorize | Returns bool. Not much different from `lower_bound != end` but more explicit. |
| `std::find(b, e, val)` | know | Linear. Can be cache-friendly for small arrays. |
| `std::find_if(b, e, pred)` | know | Linear search with condition. |
| `std::any_of` / `all_of` / `none_of` | know | Returns bool, short-circuits. |
| `std::fill(b, e, val)` | memorize | Fill array. Alternative to memset for non-trivial types. |
| `std::copy(b, e, out)` | memorize | Copy. As fast as memcpy for trivial types. |
| `std::move(b, e, out)` | know | Move with move semantics. |
| `std::transform(b, e, out, fn)` | know | Apply fn to each element, write to output. |
| `std::min` / `std::max` | memorize | Min/max of two values. Compiler generates cmov — branchless. |
| `std::clamp(val, lo, hi)` | memorize | Clamp val to [lo,hi] range. Useful for risk limit checks. |
| `std::swap(a, b)` | memorize | Swap two values. Move-aware. |
| `std::reverse(b, e)` | know | Reverse in place. |
| `std::unique(b, e)` | know | Remove consecutive duplicates. sort + unique = deduplicate. |
| `std::remove_if(b, e, pred)` | know | Use with erase-remove idiom. |
| `std::count_if(b, e, pred)` | know | Count elements satisfying condition. |
| `std::nth_element(b, nth, e)` | know | nth element in place, smaller elements on left. O(n) average. For median. |
| `std::partial_sort(b, mid, e)` | know | Sort first k elements, leave the rest untouched. |
