# LINKED-LIST - class

Consists of nodes having value and pointing next node

Not contiguous in memory

Direct access to elements is not provided

Append, Prepend, Remove First – O(1)

Remove Last, Insert, Remove, Lookup by Index, Lookup by Value – O(n)

Space Complexity – O(n)

## Corresponds to (STL) std::forward_list:

Direct access to elements is not provided.

Reverse iterator is not available, iterators invalidate when the corresponding element is deleted.

Insertion and deletion – O(1)

fl.front(), fl.push_front(obj), fl.pop_front(), fl.emplace_front(obj), fl.insert_after(it, obj), fl.emplace_after(it, obj), fl.erase_after(it), fl.resize(int), … 

https://en.cppreference.com/w/cpp/container/forward_list
