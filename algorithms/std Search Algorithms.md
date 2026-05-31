# Search and Find Algorithms

| Algorithm | Usage | Note |
|-----------|-------|------|
| `std::find` | `find(v.begin(), v.end(), val)` | Returns iterator, `end()` if not found. |
| `std::find_if` | `find_if(v.begin(), v.end(), pred)` | First element matching condition. |
| `std::find_if_not` | `find_if_not(v.begin(), v.end(), pred)` | First element NOT matching condition. |
| `std::count` | `count(v.begin(), v.end(), val)` | Count of equal elements. |
| `std::count_if` | `count_if(v.begin(), v.end(), pred)` | Count of elements matching condition. |
| `std::any_of` | `any_of(v.begin(), v.end(), pred)` | Is at least one element true? |
| `std::all_of` | `all_of(v.begin(), v.end(), pred)` | Are all elements true? |
| `std::none_of` | `none_of(v.begin(), v.end(), pred)` | Are none of the elements true? |
| `std::binary_search` | `binary_search(v.begin(), v.end(), val)` | Returns bool for sorted array. |
| `std::lower_bound` | `lower_bound(v.begin(), v.end(), val)` | First position >= val. |
| `std::upper_bound` | `upper_bound(v.begin(), v.end(), val)` | First position > val. |
| `std::equal_range` | `equal_range(v.begin(), v.end(), val)` | Returns `pair{lower, upper}`. |
| `std::search` | `search(v.begin(), v.end(), sub.begin(), sub.end())` | Search for subrange. |
| `std::min_element` | `min_element(v.begin(), v.end())` | Iterator to smallest element. |
| `std::max_element` | `max_element(v.begin(), v.end())` | Iterator to largest element. |
| `std::minmax_element` | `minmax_element(v.begin(), v.end())` | Returns `pair{min_it, max_it}`. |
