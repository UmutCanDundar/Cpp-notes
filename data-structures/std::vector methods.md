# `std::vector` Methods

| Method | Usage | Note |
|--------|-------|------|
| `push_back` | `v.push_back(x)` | Append to end. |
| `emplace_back` | `v.emplace_back(args...)` | Construct in place, prefer this. |
| `pop_back` | `v.pop_back()` | Remove from end. |
| `insert` | `v.insert(it, x)` | Insert at iterator position. |
| `erase` | `v.erase(it)` / `v.erase(a, b)` | Remove single element or range. |
| `reserve` | `v.reserve(n)` | Prevent reallocation, pre-allocate. |
| `resize` | `v.resize(n)` | Change size, new elements are default-initialized. |
| `shrink_to_fit` | `v.shrink_to_fit()` | Reduce capacity to match size. |
| `clear` | `v.clear()` | Remove elements but capacity remains. |
| `size` / `capacity` | `v.size()` / `v.capacity()` | Element count / allocated space. |
| `front` / `back` | `v.front()` / `v.back()` | Reference to first / last element. |
| `data` | `v.data()` | Raw pointer. |
| `assign` | `v.assign(n, val)` | Replace content. |
