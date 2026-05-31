# Transform and Modify Algorithms

| Algorithm | Usage | Note |
|-----------|-------|------|
| `std::transform` | `transform(v.begin(), v.end(), out, fn)` | Apply fn to each element. |
| `std::for_each` | `for_each(v.begin(), v.end(), fn)` | For side effects, no return. |
| `std::copy` | `copy(v.begin(), v.end(), out)` | Copy. |
| `std::copy_if` | `copy_if(v.begin(), v.end(), out, pred)` | Copy elements matching condition. |
| `std::copy_n` | `copy_n(v.begin(), n, out)` | Copy n elements. |
| `std::move` (algo) | `std::move(v.begin(), v.end(), out)` | Move with move semantics. |
| `std::fill` | `fill(v.begin(), v.end(), val)` | Set all to val. |
| `std::fill_n` | `fill_n(v.begin(), n, val)` | Set first n elements to val. |
| `std::generate` | `generate(v.begin(), v.end(), fn)` | Fill with fn(). |
| `std::replace` | `replace(v.begin(), v.end(), old, nw)` | Replace old with nw. |
| `std::replace_if` | `replace_if(v.begin(), v.end(), pred, nw)` | Replace elements matching condition. |
| `std::remove` | `remove(v.begin(), v.end(), val)` | Returns new end, use with erase. |
| `std::remove_if` | `remove_if(v.begin(), v.end(), pred)` | Erase-remove idiom. |
| `std::swap_ranges` | `swap_ranges(a.begin(), a.end(), b.begin())` | Swap two ranges. |
