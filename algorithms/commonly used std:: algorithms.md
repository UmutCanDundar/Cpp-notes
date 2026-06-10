# `<algorithm>` — Most Commonly Used

| API | Priority | Description | Big-O |
|-----|----------|-------------|-------|
| `std::sort(b, e)` | memorize | Unstable, O(n log n). Not on hot path. | O(n log n) |
| `std::lower_bound(b, e, val)` | memorize | O(log n) search in sorted array. For finding the first element >= val. | O(log n) |
| `std::upper_bound(b, e, val)` | memorize | For finding the first element > val. | O(log n) |
| `std::binary_search(b, e, val)` | memorize | Returns bool. Not much different from `lower_bound != end` but more explicit. | O(log n) |
| `std::find(b, e, val)` | know | Linear. Can be cache-friendly for small arrays. | O(n) |
| `std::find_if(b, e, pred)` | know | Linear search with condition. | O(n) |
| `std::any_of` / `all_of` / `none_of` | know | Returns bool, short-circuits. Use with std::ranges. | O(n) |
| `std::fill(b, e, val)` | memorize | Fill array. Alternative to memset for non-trivial types. | O(n) |
| `std::copy(b, e, out)` | memorize | Copy. As fast as memcpy for trivial types. | O(n) |
| `std::move(b, e, out)` | know | Move with move semantics. | O(n) |
| `std::transform(b, e, out, fn)` | know | Apply fn to each element, write to output. | O(n) |
| `std::min` / `std::max` | memorize | Min/max of two values. Compiler generates cmov — branchless. | O(1) |
| `std::clamp(val, lo, hi)` | memorize | Clamp val to [lo,hi] range. Useful for risk limit checks. | O(1) |
| `std::swap(a, b)` | memorize | Swap two values. Move-aware. | O(1) |
| `std::reverse(b, e)` | know | Reverse in place. | O(n) |
| `std::unique(b, e)` | know | Remove consecutive duplicates. sort + unique = deduplicate. | O(n) |
| `std::remove_if(b, e, pred)` | know | Use with erase-remove idiom. | O(n) |
| `std::count_if(b, e, pred)` | know | Count elements satisfying condition. | O(n) |
| `std::nth_element(b, nth, e)` | know | nth element in place, smaller elements on left. For median. | O(n) avg |
| `std::partial_sort(b, mid, e)` | know | Sort first k elements, leave the rest untouched. | O(n log k) |

