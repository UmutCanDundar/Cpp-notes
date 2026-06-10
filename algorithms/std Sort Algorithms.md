| Algorithm | Usage | Big-O | Notes |
|-----------|-------|-------|-------|
| `std::sort` | `sort(v.begin(), v.end())` | O(n log n) | Unstable. Introsort — quicksort + heapsort hybrid. Default choice. |
| `std::stable_sort` | `stable_sort(v.begin(), v.end())` | O(n log² n) | Preserves order of equal elements. Slightly slower than sort — use only when order matters. |
| `std::partial_sort` | `partial_sort(v.begin(), v.begin()+k, v.end())` | O(n log k) | Sort only first k elements. Faster than full sort when k << n. |
| `std::nth_element` | `nth_element(v.begin(), v.begin()+n, v.end())` | O(n) avg | nth element correct, left smaller right larger — not sorted. Use for median or top-k without needing order. |
| `std::reverse` | `reverse(v.begin(), v.end())` | O(n) | In-place. Simple swap from both ends. |
| `std::rotate` | `rotate(v.begin(), v.begin()+k, v.end())` | O(n) | Rotate left by k. Useful for sliding window or circular buffer ops. |
| `std::shuffle` | `shuffle(v.begin(), v.end(), rng)` | O(n) | Fisher-Yates. Pass `std::mt19937` as rng. |
| `std::next_permutation` | `next_permutation(v.begin(), v.end())` | O(n) | Returns false when wraps around. Sort first to iterate all permutations. O(n! * n) total. |
| `std::merge` | `merge(a.begin(),a.end(), b.begin(),b.end(), out)` | O(n+m) | Both inputs must be sorted. Writes to separate output — allocates. |
| `std::inplace_merge` | `inplace_merge(v.begin(), v.begin()+k, v.end())` | O(n log n) | Merge two sorted halves of same array. No extra allocation but slower than merge. |
| `std::partition` | `partition(v.begin(), v.end(), pred)` | O(n) | Matching elements go to front. Unstable — does not preserve relative order. Returns iterator to second group. |
| `std::stable_partition` | `stable_partition(v.begin(), v.end(), pred)` | O(n log n) | Same as partition but preserves order. Slower — use only when order matters. |
| `std::unique` | `unique(v.begin(), v.end())` | O(n) | Removes consecutive duplicates only. Must sort first for full dedup. Returns new end — call erase after. |


