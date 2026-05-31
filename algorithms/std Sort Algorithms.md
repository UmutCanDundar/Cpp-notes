# Sort and Reorder Algorithms

| Algorithm | Usage | Note |
|-----------|-------|------|
| `std::sort` | `sort(v.begin(), v.end())` | O(n log n), unstable. |
| `std::stable_sort` | `stable_sort(v.begin(), v.end())` | Preserves order of equal elements. |
| `std::partial_sort` | `partial_sort(v.begin(), v.begin()+k, v.end())` | Sort only the first k elements. |
| `std::nth_element` | `nth_element(v.begin(), v.begin()+n, v.end())` | nth element in place, smaller elements on left. |
| `std::reverse` | `reverse(v.begin(), v.end())` | Reverse in place. |
| `std::rotate` | `rotate(v.begin(), v.begin()+k, v.end())` | Rotate left by k. |
| `std::shuffle` | `shuffle(v.begin(), v.end(), rng)` | Randomly shuffle. |
| `std::next_permutation` | `next_permutation(v.begin(), v.end())` | Next permutation, returns bool. |
| `std::merge` | `merge(a.begin(),a.end(), b.begin(),b.end(), out)` | Merge two sorted arrays. |
| `std::inplace_merge` | `inplace_merge(v.begin(), v.begin()+k, v.end())` | Merge two parts of a single array. |
| `std::partition` | `partition(v.begin(), v.end(), pred)` | Elements satisfying condition go to front. |
| `std::stable_partition` | `stable_partition(v.begin(), v.end(), pred)` | Partition while preserving order. |
| `std::unique` | `unique(v.begin(), v.end())` | Removes consecutive duplicates, returns new end. |
