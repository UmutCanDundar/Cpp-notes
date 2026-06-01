# ARRAY

Each element can be accessed directly 

Contiguous in memory

The name of the array is the location of the first element (index 0)

No bounds checking

Fixed Size (No insertion or deletion after declaration)

Lookup by index, Lookup by value – O(1)

Space Complexity O(n)

## Corresponds to (STL) std::array:

Contiguous in memory

Direct elements access

All iterators are available and do not invalidate

arr.front(), arr.back(), arr.size(), arr.at(index), arr{index], arr.data(),arr.empty(), …

https://en.cppreference.com/w/cpp/container/array 

##Dynamic Array (STL) std::vector:

Dynamic Size

Contiguous in memory

Direct elements access

All iterators are available and may invalidate

Append, Remove Last, Lookup by Index – O(1)

Prepend, Remove First, Insert, Remove, Lookup by value – O(n)

Space Complexity – O(n)

v.front(), v.back(), v.push_back(obj), v.pop_back(), v.emplace_back(obj), v.size(), v.capacity(), v.empty(), …

https://en.cppreference.com/w/cpp/container/vector
