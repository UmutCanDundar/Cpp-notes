# Algorithm-Level Optimizations

| Problem | Bad | Good |
|---------|-----|------|
| Existence check, repeated search | `find(v.begin(),v.end(),x)` every time O(n) | `unordered_set` → O(1) average |
| Search in sorted array | Linear scan O(n) | `binary_search` / `lower_bound` O(log n) |
| Frequent insert/delete in middle | `vector::insert` O(n) | `list` or index swap trick |
| String concatenation in loop | `s = s + x` creates a copy every step | `s.reserve()` + `s += x` or `ostringstream` |
| Repeated computation | Same result computed on every call | Memoize, cache — map or array |
| O(n²) double loop | Every pair combination | Sort first, then two-pointer O(n log n) |
| Frequent min/max query | Linear scan every time | `priority_queue` or `std::set` with O(log n) |
