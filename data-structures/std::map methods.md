# `std::map` / `std::unordered_map` Methods

| Method | Usage | Note |
|--------|-------|------|
| `insert` | `m.insert({key, val})` | Does not insert if key already exists. |
| `insert_or_assign` (C++17) | `m.insert_or_assign(key, val)` | Overwrites if key exists. |
| `emplace` | `m.emplace(key, val)` | Construct in place. |
| `try_emplace` (C++17) | `m.try_emplace(key, args...)` | Do not touch if key exists. Prefer this. |
| `operator[]` | `m[key] = val` | Default-constructs if key does not exist — be careful. |
| `at` | `m.at(key)` | Throws exception if key not found. |
| `find` | `m.find(key)` | Returns iterator, `end()` if not found. |
| `contains` (C++20) | `m.contains(key)` | Returns bool. |
| `count` | `m.count(key)` | 0 or 1 in map. |
| `erase` | `m.erase(key)` / `m.erase(it)` | By key or iterator. |
| `extract` (C++17) | `m.extract(key)` | Extract node, key can be changed. |
| `merge` (C++17) | `m.merge(other)` | Move nodes from other. |
| `lower_bound` / `upper_bound` | `m.lower_bound(key)` | Only `std::map`, ordered. |
