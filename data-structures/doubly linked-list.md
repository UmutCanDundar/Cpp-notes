# DOUBLY LINKED-LIST – class

Consists of nodes having value and pointing next and prev nodes
Not contiguous in memory
Direct access to elements is not provided
Append, Prepend, Remove First, Remove Last – O(1)
Insert, Remove, Lookup by Index, Lookup by Value – O(n)
Space Complexity – O(n)

## Corresponds to (STL) std::list:

Direct access to elements is not provided.
Iterators invalidate when the corresponding element is deleted.
Insertion and deletion – O(1)
l.front(), l.back(), l.push_front(obj), l.push_back(obj), l.pop_front(), l.pop_back(), l.emplace_front(obj), l.emplace_back(obj), l.insert(it, obj), l.erase(it), l.size(), l.resize(int), … 
https://en.cppreference.com/w/cpp/container/list
