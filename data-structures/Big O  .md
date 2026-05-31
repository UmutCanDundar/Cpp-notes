# Big O — Complete Reference

---

## 1. Basic Concepts

| Notation | Name | Description | Example |
|----------|------|-------------|---------|
| O(1) | Constant Time | Does not depend on the size of the data set. | Accessing an array element by index |
| O(log n) | Logarithmic Time | Splits the data in each step (divide and conquer). | Binary search |
| O(n) | Linear Time | Directly proportional to the data set size. | Looping through an array |
| O(n log n) | Linearithmic Time | Splits and sorts or searches data. | Merge sort, Quick sort |
| O(n²) | Polynomial Time | Nested loops for each power of n. | Bubble sort |
| O(2ⁿ) | Exponential Time | Doubles with each addition to input. | Recursive Fibonacci |
| O(n!) | Factorial Time | One operation for every permutation of input. | Travelling salesman (brute force) |

---

## 2. Notation Types

| Symbol | Name | Meaning |
|--------|------|---------|
| Ω (Omega) | Lower Bound | **Best-case** scenario. The fastest an algorithm can run. |
| Θ (Theta) | Tight Bound | **Average-case**. What to generally expect. |
| O (Big O) | Upper Bound | **Worst-case** scenario. The slowest an algorithm can run. |

---

## 3. Useful Rules

| Rule | Explanation |
|------|-------------|
| Drop non-dominant terms | In O(n² + n), focus on O(n²) — it dominates for large n. |
| Drop constants | O(2n) simplifies to O(n). |
| Consider all cases | Always analyze best, average, and worst case. |
| Nested loops multiply | Two nested O(n) loops → O(n²). |
| Sequential steps add | O(n) step followed by O(n²) step → O(n²) total. |

---

## 4. Common Data Structure Operations

| Data Structure | Avg Access | Avg Search | Avg Insert | Avg Delete | Worst Access | Worst Search | Worst Insert | Worst Delete | Space |
|----------------|-----------|-----------|-----------|-----------|-------------|-------------|-------------|-------------|-------|
| Array | 🟢 O(1) | 🟡 O(n) | 🟡 O(n) | 🟡 O(n) | O(1) | O(n) | O(n) | O(n) | 🟡 O(n) |
| Stack | 🟡 O(n) | 🟡 O(n) | 🟢 O(1) | 🟢 O(1) | O(n) | O(n) | O(1) | O(1) | 🟡 O(n) |
| Queue | 🟡 O(n) | 🟡 O(n) | 🟢 O(1) | 🟢 O(1) | O(n) | O(n) | O(1) | O(1) | 🟡 O(n) |
| Singly-Linked List | 🟡 O(n) | 🟡 O(n) | 🟢 O(1) | 🟢 O(1) | O(n) | O(n) | O(1) | O(1) | 🟡 O(n) |
| Doubly-Linked List | 🟡 O(n) | 🟡 O(n) | 🟢 O(1) | 🟢 O(1) | O(n) | O(n) | O(1) | O(1) | 🟡 O(n) |
| Skip List | 🟢 O(log n) | 🟢 O(log n) | 🟢 O(log n) | 🟢 O(log n) | O(n) | O(n) | O(n) | O(n) | 🔴 O(n log n) |
| Hash Table | N/A | 🟢 O(1) | 🟢 O(1) | 🟢 O(1) | N/A | O(n) | O(n) | O(n) | 🟡 O(n) |
| Binary Search Tree | 🟢 O(log n) | 🟢 O(log n) | 🟢 O(log n) | 🟢 O(log n) | O(n) | O(n) | O(n) | O(n) | 🟡 O(n) |
| Cartesian Tree | N/A | 🟢 O(log n) | 🟢 O(log n) | 🟢 O(log n) | N/A | O(n) | O(n) | O(n) | 🟡 O(n) |
| B-Tree | 🟢 O(log n) | 🟢 O(log n) | 🟢 O(log n) | 🟢 O(log n) | O(log n) | O(log n) | O(log n) | O(log n) | 🟡 O(n) |
| Red-Black Tree | 🟢 O(log n) | 🟢 O(log n) | 🟢 O(log n) | 🟢 O(log n) | O(log n) | O(log n) | O(log n) | O(log n) | 🟡 O(n) |
| Splay Tree | N/A | 🟢 O(log n) | 🟢 O(log n) | 🟢 O(log n) | N/A | O(log n) | O(log n) | O(log n) | 🟡 O(n) |
| AVL Tree | 🟢 O(log n) | 🟢 O(log n) | 🟢 O(log n) | 🟢 O(log n) | O(log n) | O(log n) | O(log n) | O(log n) | 🟡 O(n) |
| KD Tree | 🟢 O(log n) | 🟢 O(log n) | 🟢 O(log n) | 🟢 O(log n) | O(n) | O(n) | O(n) | O(n) | 🟡 O(n) |

---

## 5. Array Sorting Algorithms

| Algorithm | Best | Average | Worst | Space (Worst) | Stable? |
|-----------|------|---------|-------|---------------|---------|
| Quicksort | 🟢 Ω(n log n) | Θ(n log n) | 🔴 O(n²) | 🟢 O(log n) | ❌ |
| Mergesort | 🟢 Ω(n log n) | Θ(n log n) | 🟡 O(n log n) | 🟡 O(n) | ✅ |
| Timsort | 🟢 Ω(n) | Θ(n log n) | 🟡 O(n log n) | 🟡 O(n) | ✅ |
| Heapsort | 🟢 Ω(n log n) | Θ(n log n) | 🟡 O(n log n) | 🟢 O(1) | ❌ |
| Bubble Sort | 🟢 Ω(n) | 🔴 Θ(n²) | 🔴 O(n²) | 🟢 O(1) | ✅ |
| Insertion Sort | 🟢 Ω(n) | 🔴 Θ(n²) | 🔴 O(n²) | 🟢 O(1) | ✅ |
| Selection Sort | 🔴 Ω(n²) | 🔴 Θ(n²) | 🔴 O(n²) | 🟢 O(1) | ❌ |
| Tree Sort | 🟢 Ω(n log n) | Θ(n log n) | 🔴 O(n²) | 🟡 O(n) | ✅ |
| Shell Sort | 🟢 Ω(n log n) | 🔴 Θ(n(log n)²) | 🔴 O(n(log n)²) | 🟢 O(1) | ❌ |
| Bucket Sort | 🟢 Ω(n+k) | Θ(n+k) | 🔴 O(n²) | 🟡 O(n) | ✅ |
| Radix Sort | 🟢 Ω(nk) | Θ(nk) | 🟡 O(nk) | 🔴 O(n+k) | ✅ |
| Counting Sort | 🟢 Ω(n+k) | Θ(n+k) | 🟡 O(n+k) | 🟢 O(k) | ✅ |
| Cubesort | 🟢 Ω(n) | Θ(n log n) | 🟡 O(n log n) | 🟡 O(n) | ✅ |

> 🟢 = Good &nbsp; 🟡 = Fair &nbsp; 🔴 = Bad

---

## 6. Linked List vs Vector (std::vector)

| Operation | Linked List | Vector | Notes |
|-----------|-------------|--------|-------|
| Append (end) | 🟢 O(1) | 🟢 O(1) amortized | Vector may trigger realloc occasionally |
| Remove Last | 🔴 O(n) | 🟢 O(1) | LL must traverse to find the second-to-last node |
| Prepend (front) | 🟢 O(1) | 🔴 O(n) | Vector must shift all elements |
| Remove First | 🟢 O(1) | 🔴 O(n) | Vector must shift all elements |
| Insert (middle) | 🟡 O(n) | 🟡 O(n) | LL: O(n) to find position; Vector: O(n) to shift |
| Remove (middle) | 🟡 O(n) | 🟡 O(n) | Same reason as insert |
| Lookup by Index | 🔴 O(n) | 🟢 O(1) | LL has no random access |
| Lookup by Value | 🔴 O(n) | 🔴 O(n) | Both need linear scan |
| Cache Friendliness | ❌ Poor | ✅ Excellent | LL nodes scattered in heap; vector is contiguous |
| Memory Overhead | 🔴 High | 🟢 Low | Each LL node stores a pointer (8 bytes extra) |

### When to Use Which

| Prefer Linked List | Prefer Vector |
|--------------------|---------------|
| Frequent prepend / remove first | Frequent random access by index |
| Frequent insert/delete at known pointer | Frequent append to end |
| Size changes dramatically | Cache performance matters (HFT) |
| Implementing queues/deques manually | SIMD / vectorization needed |

> **HFT rule:** Almost always prefer `std::vector`. Cache locality alone outweighs LL's O(1) insert advantage in practice.
