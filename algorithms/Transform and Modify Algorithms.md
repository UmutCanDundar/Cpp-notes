# Transform and Modify Algorithms

| Algorithm | Usage | Note | Big-O |
|-----------|-------|------|-------|
| `std::transform` | `transform(v.begin(), v.end(), out, fn)` | Apply fn to each element. | O(n) |
| `std::for_each` | `for_each(v.begin(), v.end(), fn)` | For side effects, no return. | O(n) |
| `std::copy` | `copy(v.begin(), v.end(), out)` | Copy. | O(n) |
| `std::copy_if` | `copy_if(v.begin(), v.end(), out, pred)` | Copy elements matching condition. | O(n) |
| `std::copy_n` | `copy_n(v.begin(), n, out)` | Copy n elements. | O(n) |
| `std::move` (algo) | `std::move(v.begin(), v.end(), out)` | Move with move semantics. | O(n) |
| `std::fill` | `fill(v.begin(), v.end(), val)` | Set all to val. | O(n) |
| `std::fill_n` | `fill_n(v.begin(), n, val)` | Set first n elements to val. | O(n) |
| `std::generate` | `generate(v.begin(), v.end(), fn)` | Fill with fn(). | O(n) |
| `std::replace` | `replace(v.begin(), v.end(), old, nw)` | Replace old with nw. | O(n) |
| `std::replace_if` | `replace_if(v.begin(), v.end(), pred, nw)` | Replace elements matching condition. | O(n) |
| `std::remove` | `remove(v.begin(), v.end(), val)` | Returns new end, use with erase. | O(n) |
| `std::remove_if` | `remove_if(v.begin(), v.end(), pred)` | Erase-remove idiom. | O(n) |
| `std::swap_ranges` | `swap_ranges(a.begin(), a.end(), b.begin())` | Swap two ranges. | O(n) |


