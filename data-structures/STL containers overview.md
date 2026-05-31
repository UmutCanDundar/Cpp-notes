# Containers — `<array>` `<vector>` `<deque>` `<list>` `<map>` `<unordered_map>` `<queue>` `<stack>`

| Container | Priority | Access / Insert | HFT Note |
|-----------|----------|-----------------|-----------|
| `std::array<T,N>` | memorize | O(1) / none | On stack, zero overhead. Prefer over vector for fixed-size arrays. |
| `std::vector<T>` | memorize | O(1) / O(1) amortized | Cache-friendly. Use `reserve()` to prevent realloc. Most-used container. |
| `std::deque<T>` | careful | O(1) at both ends | Not cache-friendly — split into chunks. Vector is usually better. |
| `std::list<T>` | avoid | O(n) access | Each node is a heap allocation. Cache miss. Almost never preferred. |
| `std::map<K,V>` | careful | O(log n) | Red-black tree. Each node on heap. Cache miss. unordered_map is usually faster. |
| `std::unordered_map<K,V>` | know | O(1) avg | Hash collision + rehash = unpredictable. Custom hash + open addressing is better. |
| `std::set<T>` | careful | O(log n) | Like map. Sorted vector + binary_search is often faster. |
| `std::unordered_set<T>` | know | O(1) avg | Same notes as unordered_map. |
| `std::queue<T>` | avoid | O(1) push/pop | Built on deque. Write a lock-free ring buffer instead. |
| `std::priority_queue<T>` | know | O(log n) push/pop | vector + heap. Consider a custom heap for order books. |
| `std::stack<T>` | know | O(1) | Built on deque. Using vector as a stack is better. |
| `std::span<T>` | memorize | O(1) | C++20. Non-owning view. No allocation. Prefer for passing buffers around. |
