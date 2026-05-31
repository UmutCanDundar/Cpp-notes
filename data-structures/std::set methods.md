# `std::set` / `std::unordered_set` Methods

| Method | Usage | Note |
|--------|-------|------|
| `insert` | `s.insert(val)` | Returns `pair{it, bool}`. |
| `emplace` | `s.emplace(args...)` | Construct in place. |
| `erase` | `s.erase(val)` / `s.erase(it)` | Returns count of erased elements. |
| `find` | `s.find(val)` | Iterator, `end()` if not found. |
| `contains` (C++20) | `s.contains(val)` | Returns bool. |
| `count` | `s.count(val)` | Can be >1 in multiset. |
| `lower_bound` | `s.lower_bound(val)` | First iterator >= val. |
| `upper_bound` | `s.upper_bound(val)` | First iterator > val. |
| `equal_range` | `s.equal_range(val)` | Find range in multiset. |
| `merge` (C++17) | `s.merge(other)` | Move from other. |
| `extract` (C++17) | `s.extract(it)` | Take element, can re-insert. |
