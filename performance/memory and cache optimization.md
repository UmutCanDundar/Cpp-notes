# Memory and Cache Optimizations

| Problem | Example | Solution |
|---------|---------|----------|
| Cache miss — random access | `node->next->next->data` pointer chase | Contiguous memory: vector, array. Index-based instead of linked list. |
| False sharing | Two threads writing different variables on the same cache line | Use `alignas(64)` to put each variable on a separate cache line. |
| AoS vs SoA | `struct Point{float x,y,z}` array — reading x values wastes cache | SoA: separate `float xs[], ys[], zs[]` — SIMD-friendly, cache-friendly. |
| Unnecessary copy | Passing large struct by value to a function | Pass by const ref or pointer. Use move semantics. |
| Hot/cold data mixed | Frequently and rarely accessed fields side by side in struct | Move frequently accessed fields to the beginning of the struct or a separate struct. |
| Reallocation | Vector keeps growing with push_back | Use `reserve(n)` to pre-allocate. |
| Heap fragmentation | Frequent alloc/free of small objects | Object pool or arena allocator. |

```cpp
// False sharing example — BAD
struct Counters {
    std::atomic<int> a;  // thread 1 writes
    std::atomic<int> b;  // thread 2 writes — same cache line!
};

// GOOD — separate cache lines
struct Counters {
    alignas(64) std::atomic<int> a;
    alignas(64) std::atomic<int> b;
};
```
