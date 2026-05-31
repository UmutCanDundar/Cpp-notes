# OTHER STL CONTAINERS

## std::deque:

Not contiguous in memory
Direct elements access
All iterators are available and may invalidate
Append, Prepend, Remove Last, Remove First – O(1)
Insert, Remove – O(n)
Space Complexity – O(n)
d.front(), d.back(), d.push_back(obj), d.pop_back(), d.push_front(obj), d.pop_front(), d.emplace_front, d.emplace_back(obj), d.size(), …
https://en.cppreference.com/w/cpp/container/deque

## std::set:

Ordered by key (operator <)
No concept of front and back.
No duplicates.
Iterators invalidate when the corresponding element is deleted.
s.insert(obj), s.erase(obj or it), s.clear(), s.find(obj), s.empty(), s.count(obj), …
https://en.cppreference.com/w/cpp/container/set
(std::unordered_set, std::multiset, std::unordered_multiset)

## std::map:

Elements stored as (key, value) pairs
Ordered by key
No duplicates
No concept of front and back.
Direct element access by using key
Iterators invalidate when the corresponding element is deleted.
Most operations are very efficient.
m.insert(key), m.erase(key or it), m.clear(), m.find(key), m.empty(), m.count(key),
m.at(key), m[key], m.size() …
https://en.cppreference.com/w/cpp/container/map
(std::unordered_map, std::multimap, std::unordered_multimap)
