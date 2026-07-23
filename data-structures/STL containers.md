# C++ STL Containers — Comprehensive Reference

> Memory layout, method tables, use cases, and iterator invalidation rules for every STL container.

---

## Table of Contents

1. [General Comparison Table](#general-comparison-table)
2. [Sequence Containers](#sequence-containers)
   - [std::vector](#stdvector)
   - [std::array](#stdarray)
   - [std::deque](#stddeque)
   - [std::list](#stdlist)
   - [std::forward_list](#stdforward_list)
3. [Associative Containers](#associative-containers)
   - [std::set / std::multiset](#stdset--stdmultiset)
   - [std::map / std::multimap](#stdmap--stdmultimap)
4. [Unordered Associative Containers](#unordered-associative-containers)
   - [std::unordered_set / std::unordered_multiset](#stdunordered_set--stdunordered_multiset)
   - [std::unordered_map / std::unordered_multimap](#stdunordered_map--stdunordered_multimap)
5. [Container Adapters](#container-adapters)
   - [std::stack](#stdstack)
   - [std::queue](#stdqueue)
   - [std::priority_queue](#stdpriority_queue)
6. [Iterator Invalidation — Summary Table](#iterator-invalidation--summary-table)
7. [Which Container to Choose — Decision Tree](#which-container-to-choose)

---

## General Comparison Table

| Container | Internal Structure | Ordering | Random Access | Insert (avg) | Erase (avg) | Search | Memory |
|---|---|---|---|---|---|---|---|
| `vector` | Contiguous array | Insertion order | O(1) | Back O(1)*, middle O(n) | O(n) | O(n) | Contiguous |
| `array` | Fixed-size array | Insertion order | O(1) | — (fixed size) | — | O(n) | Contiguous (stack) |
| `deque` | Chunks (segmented array) | Insertion order | O(1) | Front/back O(1) | Front/back O(1) | O(n) | Segmented blocks |
| `list` | Doubly linked list | Insertion order | O(n) | O(1) (with known position) | O(1) | O(n) | Scattered (nodes) |
| `forward_list` | Singly linked list | Insertion order | O(n) | O(1) | O(1) | O(n) | Scattered (nodes) |
| `set`/`multiset` | Red-Black Tree | Sorted by key | — | O(log n) | O(log n) | O(log n) | Scattered (nodes) |
| `map`/`multimap` | Red-Black Tree | Sorted by key | — | O(log n) | O(log n) | O(log n) | Scattered (nodes) |
| `unordered_set` | Hash table | Unordered | — | O(1) avg / O(n) worst | O(1) avg | O(1) avg | Buckets + nodes |
| `unordered_map` | Hash table | Unordered | — | O(1) avg / O(n) worst | O(1) avg | O(1) avg | Buckets + nodes |
| `stack` | Adapter (over deque) | LIFO | — | O(1) | O(1) | — | Depends on underlying container |
| `queue` | Adapter (over deque) | FIFO | — | O(1) | O(1) | — | Depends on underlying container |
| `priority_queue` | Adapter (vector + heap) | Heap order | — | O(log n) | O(log n) (top) | — | Contiguous |

\* Amortized O(1) — O(n) when reallocation is required

---

## Sequence Containers

### std::vector

#### Memory Layout
```
A CONTIGUOUS block of memory on the heap.

[vector object on the stack]
 ┌────────────┬────────────┬────────────┐
 │ T* _begin  │ T* _end    │ T* _cap_end│   (typical libstdc++ implementation)
 └─────┬──────┴─────┬──────┴─────┬──────┘
       │            │            │
       ▼            ▼            ▼
   [heap] e0 | e1 | e2 | e3 | (spare capacity)
       └── size() = _end - _begin
       └── capacity() = _cap_end - _begin

Growth: when capacity is exhausted, a new block is usually allocated at 2x
(some implementations use 1.5x), ALL elements are copied/moved,
and the old block is freed.
```

#### Members Held on the Stack (typical)
| Member (implementation-defined) | Description |
|---|---|
| `T* _M_start` | Address of the first element |
| `T* _M_finish` | Address just past the last element (size boundary) |
| `T* _M_end_of_storage` | End of the allocated block (capacity boundary) |

Total is usually **24 bytes** (3 pointers on a 64-bit system).

#### Method Table

| Method | Return Type | Parameters | Complexity | Description |
|---|---|---|---|---|
| `push_back(const T&)` / `push_back(T&&)` | `void` | element | Amortized O(1) | Appends to the back |
| `emplace_back(Args&&...)` | `T&` (C++17+) | constructor arguments | Amortized O(1) | Constructs in-place at the back |
| `pop_back()` | `void` | — | O(1) | Removes the last element |
| `insert(iterator pos, const T&)` | `iterator` | position, value | O(n) | Inserts at the given position |
| `emplace(iterator pos, Args&&...)` | `iterator` | position, args | O(n) | In-place insertion |
| `erase(iterator pos)` | `iterator` | position | O(n) | Erases, returns iterator to next element |
| `erase(iterator first, iterator last)` | `iterator` | range | O(n) | Erases a range |
| `clear()` | `void` | — | O(n) | Removes all elements (capacity unchanged) |
| `at(size_type)` | `T&` / `const T&` | index | O(1) | Bounds-checked access (throws `out_of_range`) |
| `operator[](size_type)` | `T&` / `const T&` | index | O(1) | Unchecked access |
| `front()` | `T&` / `const T&` | — | O(1) | First element |
| `back()` | `T&` / `const T&` | — | O(1) | Last element |
| `data()` | `T*` / `const T*` | — | O(1) | Raw pointer to the underlying array |
| `size()` | `size_type` | — | O(1) | Number of elements |
| `capacity()` | `size_type` | — | O(1) | Allocated capacity |
| `empty()` | `bool` | — | O(1) | Is it empty? |
| `reserve(size_type)` | `void` | new capacity | O(n) (if reallocated) | Pre-allocates capacity |
| `shrink_to_fit()` | `void` | — | O(n) | Requests to release excess capacity |
| `resize(size_type)` / `resize(size_type, T)` | `void` | new size, (optional) fill value | O(n) | Changes the size |
| `assign(size_type, const T&)` | `void` | count, value | O(n) | Reassigns the contents |
| `swap(vector&)` | `void` | another vector | O(1) | Swaps contents |
| `begin()` / `end()` | `iterator` | — | O(1) | Iterators |
| `rbegin()` / `rend()` | `reverse_iterator` | — | O(1) | Reverse iterators |
| `cbegin()` / `cend()` | `const_iterator` | — | O(1) | Const iterators |

#### Use Cases
✅ **Advantageous:**
- Random access via index is needed
- Cache-friendly operations are needed (contiguous memory → CPU-cache friendly)
- Operations are mostly at the back
- Default container choice when nothing else stands out (**use vector**)

❌ **Disadvantageous:**
- Frequent insert/erase at the front or middle (O(n) shifting cost)
- Pointer/iterator stability across reallocation is critical

#### Iterator Invalidation
| Operation | Effect |
|---|---|
| `push_back` / `emplace_back` (if reallocation occurs) | **All** iterators, pointers, references are invalidated |
| `push_back` (no reallocation) | Only `end()` iterator is invalidated, others remain valid |
| `insert` | All iterators/references after the insertion point are invalidated (all of them if reallocation occurs) |
| `erase` | All iterators/references after the erased point are invalidated |
| `reserve` (if capacity grows) | All iterators/pointers/references are invalidated |
| `clear()` | All iterators/references are invalidated (except `begin()==end()`) |

---

### std::array

#### Memory Layout
```
A FIXED-size, contiguous array, typically on the STACK (or wherever its owner lives).
Uses NO dynamic memory (heap) — it's essentially a wrapped C-style array.

 ┌────┬────┬────┬────┬────┐
 │ e0 │ e1 │ e2 │ e3 │ e4 │   <- std::array<T,5>
 └────┴────┴────┴────┴────┘
 Size is fixed at compile time (template parameter N).
```

#### Members Held on the Stack
| Member | Description |
|---|---|
| `T _M_elems[N]` | A raw C-array of N elements, embedded directly |

No extra pointers/metadata — just `sizeof(T) * N`.

#### Method Table

| Method | Return Type | Parameters | Complexity | Description |
|---|---|---|---|---|
| `at(size_type)` | `T&` | index | O(1) | Bounds-checked access |
| `operator[](size_type)` | `T&` | index | O(1) | Unchecked access |
| `front()` / `back()` | `T&` | — | O(1) | First/last element |
| `data()` | `T*` | — | O(1) | Raw pointer |
| `size()` | `size_type` | — | O(1) | N (fixed) |
| `max_size()` | `size_type` | — | O(1) | N (same as size) |
| `empty()` | `bool` | — | O(1) | Is N==0? |
| `fill(const T&)` | `void` | value | O(n) | Fills all elements |
| `swap(array&)` | `void` | another array | O(n) | Element-wise content swap |
| `begin()` / `end()` | `iterator` | — | O(1) | Iterators |

#### Use Cases
✅ **Advantageous:**
- Size is known at compile time and won't change
- Heap allocation overhead is undesirable (embedded/performance-critical code)
- Usage in a `constexpr` context is needed

❌ **Disadvantageous:**
- Size needs to change at runtime (use `vector` instead)

#### Iterator Invalidation
- No insert/erase since the size is fixed.
- References only remain valid across `swap()` in terms of content, not address (swap is element-wise, not address-based).

---

### std::deque

#### Memory Layout
```
"Double-Ended Queue" — a SEGMENTED (chunked) memory layout.
NOT one contiguous block; a "map" of pointers to fixed-size chunks is maintained.

 ┌───────────────────────────┐
 │   map (chunk pointers)    │  <- T** map_ptr (array of chunk addresses)
 └──┬─────┬─────┬─────┬──────┘
    ▼     ▼     ▼     ▼
 [chunk0][chunk1][chunk2][chunk3]   <- each is a fixed-size (e.g. 512 bytes) array of T
    e0-e7  e8-e15 e16-e23 e24-e31

Push front/back: a new chunk is added to the map (existing chunks are NOT moved).
Random access to a middle element is O(1) but with a larger constant factor than vector
(2 levels of indirection: map -> chunk -> element).
```

#### Members Held on the Stack (typical)
| Member | Description |
|---|---|
| `T** _M_map` | Pointer to the array of chunk pointers |
| `size_t _M_map_size` | Number of chunks in the map |
| iterator `_M_start`, `_M_finish` | "Deque iterators" pointing to the first and last elements (chunk ptr + element ptr + chunk boundaries) |

#### Method Table

| Method | Return Type | Parameters | Complexity | Description |
|---|---|---|---|---|
| `push_back(const T&)` | `void` | element | Amortized O(1) | Appends to the back |
| `push_front(const T&)` | `void` | element | Amortized O(1) | Prepends to the front |
| `emplace_back(Args&&...)` | `T&` | args | O(1) | Constructs in-place at the back |
| `emplace_front(Args&&...)` | `T&` | args | O(1) | Constructs in-place at the front |
| `pop_back()` | `void` | — | O(1) | Removes from the back |
| `pop_front()` | `void` | — | O(1) | Removes from the front |
| `insert(iterator, const T&)` | `iterator` | position, value | O(n) | Inserts in the middle (faster near the ends) |
| `erase(iterator)` | `iterator` | position | O(n) | Erases |
| `at(size_type)` | `T&` | index | O(1) | Bounds-checked access |
| `operator[](size_type)` | `T&` | index | O(1) | Access via 2-level indirection |
| `front()` / `back()` | `T&` | — | O(1) | End elements |
| `size()` / `empty()` | `size_type` / `bool` | — | O(1) | — |
| `clear()` | `void` | — | O(n) | Removes all elements |
| `resize(size_type)` | `void` | new size | O(n) | Changes the size |
| `shrink_to_fit()` | `void` | — | O(n) | Requests to reduce capacity |
| `begin()` / `end()` | `iterator` | — | O(1) | Iterators |

#### Use Cases
✅ **Advantageous:**
- Frequent insert/erase at both the front and back is needed
- Random access combined with flexible growth is needed
- E.g. **sliding window** algorithms, **BFS** queue structures

❌ **Disadvantageous:**
- Pure sequential-read performance isn't as cache-friendly as vector (chunk jumps)
- Middle insert/erase is still O(n)

#### Iterator Invalidation
| Operation | Effect |
|---|---|
| `push_back` / `push_front` | All iterators are invalidated; **references and pointers remain valid** (unlike vector!) |
| `pop_back` | Iterator/ref/pointer to the removed back element is invalidated, others remain valid |
| `pop_front` | Iterator/ref/pointer to the removed front element is invalidated, others remain valid |
| `insert` (except at the ends) | All iterators/references/pointers are invalidated |
| `erase` (except at the ends) | All iterators/references/pointers are invalidated |

---

### std::list

#### Memory Layout
```
A DOUBLY LINKED LIST. Elements are SCATTERED across memory,
each one a separate heap allocation (node).

 ┌──────┐    ┌──────┐    ┌──────┐    ┌──────┐
 │ prev │◄───┤ prev │◄───┤ prev │◄───┤ prev │
 │ data │    │ data │    │ data │    │ data │
 │ next │───►│ next │───►│ next │───►│ next │
 └──────┘    └──────┘    └──────┘    └──────┘
  node0        node1        node2        node3
  (at different heap addresses, NOT contiguous)
```

#### Members Held on the Stack
| Member | Description |
|---|---|
| `_List_node_base _M_node` | Sentinel/head node (prev/next pointers) — circular list |
| `size_t _M_size` | Element count (kept since C++11 for O(1) `size()`) |

Each node additionally holds `T data`, `node* prev`, `node* next` internally (on the heap).

#### Method Table

| Method | Return Type | Parameters | Complexity | Description |
|---|---|---|---|---|
| `push_back(const T&)` | `void` | element | O(1) | Appends to the back |
| `push_front(const T&)` | `void` | element | O(1) | Prepends to the front |
| `emplace_back(Args&&...)` / `emplace_front` | `T&` | args | O(1) | In-place insertion |
| `pop_back()` / `pop_front()` | `void` | — | O(1) | Removes from the ends |
| `insert(iterator, const T&)` | `iterator` | position, value | O(1) | Inserts before the given node |
| `erase(iterator)` | `iterator` | position | O(1) | Erases a node |
| `front()` / `back()` | `T&` | — | O(1) | End elements |
| `size()` / `empty()` | `size_type` / `bool` | — | O(1) | — |
| `clear()` | `void` | — | O(n) | Removes all nodes |
| `remove(const T&)` | `void` | value | O(n) | Removes all elements equal to value |
| `remove_if(Predicate)` | `void` | predicate | O(n) | Removes elements matching a condition |
| `unique()` | `void` | (optional) predicate | O(n) | Removes consecutive duplicates |
| `sort()` | `void` | (optional) comparator | O(n log n) | Merge sort (adapted for linked lists) |
| `merge(list&)` | `void` | another sorted list | O(n) | Merges two sorted lists |
| `splice(iterator, list&)` | `void` | position, another list | O(1) | **Moves** nodes (no copying) |
| `reverse()` | `void` | — | O(n) | Reverses the list |
| `begin()` / `end()` | `iterator` (bidirectional) | — | O(1) | Iterators |

#### Use Cases
✅ **Advantageous:**
- Frequent insertion/deletion (especially in the middle, given an iterator) O(1)
- `splice` for moving nodes is needed (e.g. LRU cache implementation)
- Iterator/pointer/reference stability is critical (insert/erase don't affect other elements)

❌ **Disadvantageous:**
- Random access is needed (O(n), no indexing)
- Cache-unfriendly (scattered nodes, pointer chasing → performance loss)
- Every node carries an extra 2-pointer overhead

#### Iterator Invalidation
| Operation | Effect |
|---|---|
| `insert` | **No** iterators/references/pointers are invalidated |
| `erase` | Only the **erased element's** iterator/reference/pointer is invalidated, others remain valid |
| `splice` | No invalidation, only "ownership" changes |
| `sort` / `reverse` / `unique` | Iterators/pointers/references remain valid (node addresses stay fixed even if elements reorder) |

---

### std::forward_list

#### Memory Layout
```
A SINGLY linked list. Unlike list, it only has a "next" pointer, no "prev".
This reduces the per-node memory overhead (saves one pointer).

 ┌──────┐    ┌──────┐    ┌──────┐
 │ data │───►│ data │───►│ data │───► nullptr
 │ next │    │ next │    │ next │
 └──────┘    └──────┘    └──────┘
```

#### Members Held on the Stack
| Member | Description |
|---|---|
| `_Fwd_list_node_base _M_head` | Pointer to the first node (next only) |

**No `size()`** (a deliberate design choice — to keep overhead minimal, not even an extra counter was added for an O(1) size).

#### Method Table

| Method | Return Type | Parameters | Complexity | Description |
|---|---|---|---|---|
| `push_front(const T&)` | `void` | element | O(1) | Prepends to the front |
| `emplace_front(Args&&...)` | `T&` | args | O(1) | Constructs in-place at the front |
| `pop_front()` | `void` | — | O(1) | Removes from the front |
| `insert_after(iterator, const T&)` | `iterator` | position, value | O(1) | Inserts **after** the given node |
| `emplace_after(iterator, Args&&...)` | `iterator` | position, args | O(1) | In-place insertion after |
| `erase_after(iterator)` | `iterator` | position | O(1) | Erases the node after the given one |
| `front()` | `T&` | — | O(1) | First element (**no `back()`**!) |
| `empty()` | `bool` | — | O(1) | — |
| `before_begin()` | `iterator` | — | O(1) | Virtual position before `begin()` (needed to erase the head element) |
| `remove(const T&)` / `remove_if` | `void` | value/predicate | O(n) | Removes matching elements |
| `unique()` | `void` | — | O(n) | Removes consecutive duplicates |
| `sort()` | `void` | (optional) comparator | O(n log n) | Sorts |
| `merge(forward_list&)` | `void` | another list | O(n) | Merges |
| `splice_after(iterator, forward_list&)` | `void` | position, list | O(1) | Moves nodes |
| `reverse()` | `void` | — | O(n) | Reverses |
| `begin()` / `end()` | `iterator` (forward only) | — | O(1) | Iterators |

#### Use Cases
✅ **Advantageous:**
- Memory overhead is critical and one-way traversal suffices
- Very large numbers of small elements (per-node pointer savings add up meaningfully)

❌ **Disadvantageous:**
- Backward traversal, `size()`, or `back()` is needed
- In most practical scenarios, `list` or `vector` is preferred

#### Iterator Invalidation
- Same logic as `list`: only the erased element's iterator is invalidated, insertion doesn't affect any iterator.
- **Note:** Since `erase_after` is used, you need to keep the iterator to the node **before** the one you want to erase.

---

## Associative Containers

### std::set / std::multiset

#### Memory Layout
```
A RED-BLACK TREE (self-balancing binary search tree).
Each node is a separate heap allocation, kept sorted by key
(in-order traversal = sorted output).

              [8,black]
             /          \
       [3,red]          [10,red]
       /     \                \
   [1,blk]  [6,blk]         [14,blk]

Each node: key + color(red/black) + parent/left/right pointers.
set: key only (value == key)
multiset: multiple entries with the same key allowed (equal keys chained together)
```

#### Members Held on the Stack
| Member | Description |
|---|---|
| `_Rb_tree_node_base _M_header` | Sentinel node (root, leftmost=begin, for rightmost) |
| `size_t _M_node_count` | Element count |
| `Compare _M_key_compare` | Comparison function (usually `std::less<T>`, empty class optimized away) |

#### Method Table

| Method | Return Type | Parameters | Complexity | Description |
|---|---|---|---|---|
| `insert(const T&)` | `pair<iterator,bool>` (set) / `iterator` (multiset) | value | O(log n) | Inserts; in `set` a duplicate is not added, `bool` reports success |
| `emplace(Args&&...)` | `pair<iterator,bool>` | args | O(log n) | In-place insertion |
| `erase(iterator)` | `iterator` | position | O(log n) amortized | Erases |
| `erase(const T&)` | `size_type` | value | O(log n + k) | Erases all elements equal to value, returns count erased |
| `find(const T&)` | `iterator` | value | O(log n) | Finds, or `end()` if not found |
| `count(const T&)` | `size_type` | value | O(log n + k) | Number of matching elements (0/1 in set) |
| `contains(const T&)` (C++20) | `bool` | value | O(log n) | Is it present? |
| `lower_bound(const T&)` | `iterator` | value | O(log n) | First element ≥ value |
| `upper_bound(const T&)` | `iterator` | value | O(log n) | First element > value |
| `equal_range(const T&)` | `pair<iterator,iterator>` | value | O(log n) | Range [lower_bound, upper_bound) |
| `size()` / `empty()` | `size_type` / `bool` | — | O(1) | — |
| `clear()` | `void` | — | O(n) | Removes all elements |
| `begin()` / `end()` | `iterator` (bidirectional) | — | O(1) | Points to smallest / one past largest |
| `rbegin()` / `rend()` | `reverse_iterator` | — | O(1) | Reverse iterators |
| `key_comp()` | `Compare` | — | O(1) | Returns the comparison function |

#### Use Cases
✅ **Advantageous:**
- Elements need to stay continuously **sorted**
- Range queries (`lower_bound`/`upper_bound`) are needed
- A duplicate-free, sorted collection is needed (`set`)
- Sorted frequency counting is needed (`multiset`)

❌ **Disadvantageous:**
- Ordering isn't needed → `unordered_set` is faster (O(1) vs O(log n))
- Random access (index) is needed

#### Iterator Invalidation
| Operation | Effect |
|---|---|
| `insert` | **No** iterators are invalidated |
| `erase` | Only the **erased element's** iterator is invalidated, others remain valid |

---

### std::map / std::multimap

#### Memory Layout
```
Uses the SAME Red-Black Tree structure as set; the difference is that
each node's value_type is std::pair<const Key, T> (key + mapped value together).

              [{8:"h"}]
             /         \
      [{3:"c"}]      [{10:"j"}]

map: at most one pair per key
multimap: multiple pairs with the same key allowed
```

#### Members Held on the Stack
Identical to `set` (Red-Black tree header + node_count + comparator), except the node holds a `pair<const Key,T>`.

#### Method Table

| Method | Return Type | Parameters | Complexity | Description |
|---|---|---|---|---|
| `insert({key,value})` | `pair<iterator,bool>` (map) | key-value pair | O(log n) | Inserts; does nothing if key exists |
| `insert_or_assign(key, value)` (C++17) | `pair<iterator,bool>` | key, value | O(log n) | Updates if present, inserts otherwise |
| `emplace(Args&&...)` | `pair<iterator,bool>` | args | O(log n) | In-place insertion |
| `try_emplace(key, Args&&...)` (C++17) | `pair<iterator,bool>` | key, args | O(log n) | Constructs in-place only if key is absent (avoids unnecessary copies) |
| `operator[](const Key&)` | `T&` | key | O(log n) | **Inserts a default-constructed value if the key doesn't exist!** |
| `at(const Key&)` | `T&` | key | O(log n) | Throws `out_of_range` if missing (does not insert) |
| `erase(iterator)` / `erase(const Key&)` | `iterator` / `size_type` | position/key | O(log n) | Erases |
| `find(const Key&)` | `iterator` | key | O(log n) | Finds, or `end()` if not found |
| `count(const Key&)` | `size_type` | key | O(log n + k) | 0/1 in map, k in multimap |
| `contains(const Key&)` (C++20) | `bool` | key | O(log n) | Is it present? |
| `lower_bound` / `upper_bound` / `equal_range` | `iterator` / `pair<iterator,iterator>` | key | O(log n) | Same logic as set |
| `size()` / `empty()` / `clear()` | — | — | O(1)/O(1)/O(n) | — |
| `begin()` / `end()` | `iterator` | — | O(1) | Traverses in sorted key order |

#### Use Cases
✅ **Advantageous:**
- Key-value association + **sorted** key ordering is needed
- Range-based key queries (`lower_bound` for "first key greater than X")
- Ordered dictionary / config-like usage

❌ **Disadvantageous:**
- If order doesn't matter, `unordered_map` performs better
- Watch out for accidental insertion via `operator[]`

#### Iterator Invalidation
| Operation | Effect |
|---|---|
| `insert` | **No** iterators are invalidated |
| `erase` | Only the erased element's iterator is invalidated |

---

## Unordered Associative Containers

### std::unordered_set / std::unordered_multiset

#### Memory Layout
```
A HASH TABLE (bucket array + chaining/linked nodes).

 bucket[0] -> node(key=17) -> node(key=33) -> nullptr
 bucket[1] -> nullptr
 bucket[2] -> node(key=9)  -> nullptr
 bucket[3] -> node(key=5)  -> node(key=21) -> nullptr
 ...

hash(key) % bucket_count -> determines which bucket the element falls into.
Collisions are resolved via chaining within the same bucket.
Once load_factor = size() / bucket_count() crosses a threshold (max_load_factor),
the table is REHASHED (bucket count increases, all elements redistributed).
```

#### Members Held on the Stack
| Member | Description |
|---|---|
| `Bucket* _M_buckets` | Pointer to the bucket array (on the heap) |
| `size_t _M_bucket_count` | Number of buckets |
| `size_t _M_element_count` | Number of elements |
| `float _M_max_load_factor` | Rehash threshold (default 1.0) |
| `Hash _M_hash` | Hash function |
| `KeyEqual _M_equal` | Equality function |

#### Method Table

| Method | Return Type | Parameters | Complexity | Description |
|---|---|---|---|---|
| `insert(const T&)` | `pair<iterator,bool>` | value | O(1) avg / O(n) worst | Inserts |
| `emplace(Args&&...)` | `pair<iterator,bool>` | args | O(1) avg | In-place insertion |
| `erase(iterator)` / `erase(const T&)` | `iterator` / `size_type` | — | O(1) avg | Erases |
| `find(const T&)` | `iterator` | value | O(1) avg / O(n) worst | Finds |
| `count(const T&)` | `size_type` | value | O(1) avg | Number of matches |
| `contains(const T&)` (C++20) | `bool` | value | O(1) avg | Is it present? |
| `bucket_count()` | `size_type` | — | O(1) | Number of buckets |
| `bucket_size(size_type)` | `size_type` | bucket index | O(bucket size) | Number of elements in that bucket |
| `bucket(const T&)` | `size_type` | value | O(1) | The bucket index for a value |
| `load_factor()` | `float` | — | O(1) | size/bucket_count |
| `max_load_factor()` / `max_load_factor(float)` | `float` / `void` | (optional) new value | O(1) | Read/set the rehash threshold |
| `rehash(size_type)` | `void` | min bucket count | O(n) | Changes the bucket count |
| `reserve(size_type)` | `void` | element count | O(n) | Pre-allocates enough buckets |
| `size()` / `empty()` / `clear()` | — | — | O(1)/O(1)/O(n) | — |
| `begin()` / `end()` | `iterator` (forward only) | — | O(1) | Unordered traversal |

#### Use Cases
✅ **Advantageous:**
- Fast lookup/insert/erase is needed and **order doesn't matter**
- Frequency counting, "is it present" checks, cache/lookup tables

❌ **Disadvantageous:**
- Sorted traversal is needed (use `set` instead)
- A poor hash function risks worst-case O(n)
- Memory overhead is generally higher than `set` (bucket array + chaining)

#### Iterator Invalidation
| Operation | Effect |
|---|---|
| `insert` (no rehash triggered) | Iterators remain valid, but **no guarantees** are given about reference/pointer ordering |
| `insert` (rehash triggered) | **All** iterators are invalidated; references/pointers **remain valid** |
| `erase` | Only the erased element's iterator is invalidated |
| `rehash` / `reserve` | All iterators are invalidated, references/pointers remain valid |

---

### std::unordered_map / std::unordered_multimap

#### Memory Layout
Identical hash table structure to `unordered_set`; nodes hold `pair<const Key,T>`.

#### Members Held on the Stack
Same as `unordered_set` (bucket array pointer, bucket_count, element_count, max_load_factor, hash, key_equal).

#### Method Table

| Method | Return Type | Parameters | Complexity | Description |
|---|---|---|---|---|
| `insert({key,value})` | `pair<iterator,bool>` | pair | O(1) avg | Inserts |
| `insert_or_assign(key,value)` (C++17) | `pair<iterator,bool>` | key, value | O(1) avg | Updates if present |
| `emplace(Args&&...)` | `pair<iterator,bool>` | args | O(1) avg | In-place insertion |
| `try_emplace(key, Args&&...)` (C++17) | `pair<iterator,bool>` | key, args | O(1) avg | Constructs in-place if key absent |
| `operator[](const Key&)` | `T&` | key | O(1) avg | Inserts a default if missing |
| `at(const Key&)` | `T&` | key | O(1) avg | Throws if missing |
| `erase(iterator)` / `erase(const Key&)` | — | — | O(1) avg | Erases |
| `find(const Key&)` | `iterator` | key | O(1) avg | Finds |
| `count(const Key&)` | `size_type` | key | O(1) avg | 0/1 in map |
| `contains(const Key&)` (C++20) | `bool` | key | O(1) avg | Is it present? |
| `bucket_count()` / `load_factor()` / `rehash()` / `reserve()` | — | — | — | Same as `unordered_set` |
| `size()` / `empty()` / `clear()` | — | — | O(1)/O(1)/O(n) | — |

#### Use Cases
✅ **Advantageous:** Fast key-value lookup, cache implementations, frequency/counter tables
❌ **Disadvantageous:** If sorted key traversal is needed, prefer `map`

#### Iterator Invalidation
Same rules as `unordered_set`, verbatim.

---

## Container Adapters

> **Note:** Adapters don't have their own memory layout; they wrap another container (with a default choice) and expose a restricted interface. **They provide no iterators.**

### std::stack

#### Memory Layout
```
Default underlying container: std::deque<T> (vector or list can also be used)

std::stack<int> s;                       // internally uses deque<int>
std::stack<int, std::vector<int>> s2;    // internally uses vector<int>

 [underlying container: deque/vector/list]
        top() ────► the most recently added element (LIFO)
```

#### Members Held on the Stack
| Member | Description |
|---|---|
| `Container c` | The wrapped underlying container itself (protected member) |

#### Method Table

| Method | Return Type | Parameters | Complexity | Description |
|---|---|---|---|---|
| `push(const T&)` | `void` | element | O(1) | Pushes onto the top |
| `emplace(Args&&...)` | `void` / `T&` (C++17+) | args | O(1) | Constructs in-place on top |
| `pop()` | `void` | — | O(1) | Removes the top element (**does not return a value!**) |
| `top()` | `T&` / `const T&` | — | O(1) | Accesses the top element (doesn't remove it) |
| `empty()` | `bool` | — | O(1) | Is it empty? |
| `size()` | `size_type` | — | O(1) | Number of elements |
| `swap(stack&)` | `void` | another stack | O(1) | Swaps contents |

⚠️ **No iterators.** Traversal is not supported.

#### Use Cases
✅ **Advantageous:** Bracket matching, DFS, undo/redo mechanisms, expression evaluation, backtracking algorithms
❌ **Disadvantageous:** Not suitable if access/traversal of middle elements is needed

#### Iterator Invalidation
Since there are no iterators, this concept doesn't apply; however, a reference obtained from `top()` is subject to the underlying container's invalidation rules after `pop()`/`push()` (e.g. realloc risk if vector-based).

---

### std::queue

#### Memory Layout
```
Default underlying container: std::deque<T> (list can also be used; vector CANNOT be
used in the strict sense since pop_front is needed... actually queue doesn't use
push_front, only push_back/pop_front, so vector could theoretically work but
pop_front would be O(n), hence deque is preferred)

 front ─► [e0][e1][e2][e3] ◄─ back
          pop_front()   push_back()
```

#### Members Held on the Stack
| Member | Description |
|---|---|
| `Container c` | The wrapped underlying container (protected member) |

#### Method Table

| Method | Return Type | Parameters | Complexity | Description |
|---|---|---|---|---|
| `push(const T&)` | `void` | element | O(1) | Adds to the back |
| `emplace(Args&&...)` | `void` / `T&` (C++17+) | args | O(1) | Constructs in-place at the back |
| `pop()` | `void` | — | O(1) | Removes the front element |
| `front()` | `T&` / `const T&` | — | O(1) | Front element |
| `back()` | `T&` / `const T&` | — | O(1) | Back element |
| `empty()` | `bool` | — | O(1) | Is it empty? |
| `size()` | `size_type` | — | O(1) | Number of elements |
| `swap(queue&)` | `void` | another queue | O(1) | Swaps contents |

⚠️ **No iterators.**

#### Use Cases
✅ **Advantageous:** BFS, task scheduling, producer-consumer buffers, print/job queues
❌ **Disadvantageous:** Not suitable if random access or priority-based processing is needed

#### Iterator Invalidation
No iterators; subject to the underlying container's rules.

---

### std::priority_queue

#### Memory Layout
```
Default underlying container: std::vector<T> + BINARY HEAP algorithm
(heapify, push_heap, pop_heap)

Max-heap example (default: largest element at top()):
              90
            /    \
          70      80
         /  \    /
        20  60  50

Stored as a flat array inside the vector:
[90, 70, 80, 20, 60, 50]
 children of index i: 2i+1, 2i+2  (array-based complete binary tree)
```

#### Members Held on the Stack
| Member | Description |
|---|---|
| `Container c` | Underlying container (usually vector) |
| `Compare comp` | Comparison function (default `std::less` → max-heap) |

#### Method Table

| Method | Return Type | Parameters | Complexity | Description |
|---|---|---|---|---|
| `push(const T&)` | `void` | element | O(log n) | Inserts, rebalances the heap (`push_heap`) |
| `emplace(Args&&...)` | `void` | args | O(log n) | In-place insertion |
| `pop()` | `void` | — | O(log n) | Removes the highest-priority element (`pop_heap`) |
| `top()` | `const T&` | — | O(1) | Accesses the highest-priority element |
| `empty()` | `bool` | — | O(1) | Is it empty? |
| `size()` | `size_type` | — | O(1) | Number of elements |
| `swap(priority_queue&)` | `void` | another priority_queue | O(1) | Swaps contents |

⚠️ **No iterators.** No access to elements other than via `top()`.

#### Use Cases
✅ **Advantageous:**
- Algorithms requiring "always take the smallest/largest" such as Dijkstra, Prim's
- Finding the top-K elements
- Event simulation (time-based priority)

❌ **Disadvantageous:**
- Full sorted traversal is needed (`set` is more suitable)
- For a min-heap, you must supply the `std::greater<T>` comparator:
  ```cpp
  std::priority_queue<int, std::vector<int>, std::greater<int>> minHeap;
  ```

#### Iterator Invalidation
No iterators; the `top()` reference should be considered invalidated after `pop()`/`push()` (the heap gets rearranged).

---

## Iterator Invalidation — Summary Table

| Container | Insert Effect | Erase Effect | Trigger for Realloc/Rehash |
|---|---|---|---|
| `vector` | If reallocated, **all** invalidated; otherwise only after the insertion point | Everything after the erased point is invalidated | `push_back`, `insert`, `reserve` (if capacity insufficient) |
| `array` | — (fixed size) | — (fixed size) | None |
| `deque` | All iterators invalidated (except insert at the ends); refs/pointers preserved for end-insertion | All iterators in the affected chunk plus the erased one are invalidated | Any insert in the middle/front/back is potentially risky |
| `list` / `forward_list` | **None** invalidated | Only the erased element's iterator is invalidated | None (node-based, addresses stay fixed) |
| `set` / `multiset` / `map` / `multimap` | **None** invalidated | Only the erased element's iterator is invalidated | None (tree node addresses stay fixed) |
| `unordered_*` | If rehashed, **all iterators** invalidated (references/pointers preserved) | Only the erased element's iterator is invalidated | `insert`, `rehash`, `reserve` (if load_factor exceeded) |
| `stack`/`queue`/`priority_queue` | No iterator concept | No iterator concept | Depends on underlying container |

**Golden rule:** In node-based containers (list, set, map, unordered_*), **inserting** is safe; **erasing** only affects the erased element. In array-based containers (vector, deque), both inserting and erasing can cause widespread invalidation.

---

## Which Container to Choose

```
Is random access (by index) required?
│
├── YES
│   │
│   ├── Is the size fixed (known at compile time)?
│   │   ├── YES → std::array
│   │   └── NO
│   │       ├── Only inserting/erasing at the back → std::vector  (default choice)
│   │       └── Frequent insert/erase at front AND back → std::deque
│   │
│   └── (as above)
│
└── NO (only key/value or sorted traversal matters)
    │
    ├── Is it a key-value relationship, or just values?
    │   │
    │   ├── VALUES ONLY (no key)
    │   │   ├── Does order matter?
    │   │   │   ├── YES (sorted) → std::set / std::multiset
    │   │   │   └── NO (speed priority) → std::unordered_set / unordered_multiset
    │   │   └── Frequent middle insert/erase with iterator in hand → std::list
    │   │
    │   └── KEY-VALUE
    │       ├── Does order matter?
    │       │   ├── YES → std::map / std::multimap
    │       │   └── NO (speed priority) → std::unordered_map / unordered_multimap
    │
    └── Special access pattern?
        ├── LIFO (last in, first out) → std::stack
        ├── FIFO (first in, first out) → std::queue
        └── Always need the highest/lowest priority element → std::priority_queue
```

### Quick Decision Criteria

| Need | Preference |
|---|---|
| General-purpose, "not sure what to use" | `vector` |
| Fixed size, should live on the stack | `array` |
| Fast insert/erase at both ends | `deque` |
| Frequent middle insert/erase, iterator stability | `list` |
| Minimal memory overhead, one-way traversal is enough | `forward_list` |
| Sorted, deduplicated data | `set` |
| Sorted, key-value | `map` |
| Fastest lookup, order doesn't matter | `unordered_map` / `unordered_set` |
| LIFO behavior | `stack` |
| FIFO behavior | `queue` |
| Priority-based processing (heap) | `priority_queue` |

---

## Bonus: Complexity Cheat Sheet (Big-O)

| Container | Access | Search | Insert (front) | Insert (back) | Insert (middle) | Erase (front) | Erase (back) | Erase (middle) |
|---|---|---|---|---|---|---|---|---|
| `vector` | O(1) | O(n) | O(n) | O(1)* | O(n) | O(n) | O(1) | O(n) |
| `array` | O(1) | O(n) | — | — | — | — | — | — |
| `deque` | O(1) | O(n) | O(1) | O(1) | O(n) | O(1) | O(1) | O(n) |
| `list` | O(n) | O(n) | O(1) | O(1) | O(1)** | O(1) | O(1) | O(1)** |
| `forward_list` | O(n) | O(n) | O(1) | O(n)*** | O(1)** | O(1) | O(n)*** | O(1)** |
| `set`/`map` | — | O(log n) | O(log n) | O(log n) | O(log n) | O(log n) | O(log n) | O(log n) |
| `unordered_set`/`map` | — | O(1) avg | O(1) avg | O(1) avg | O(1) avg | O(1) avg | O(1) avg | O(1) avg |

\* Amortized  \*\* Given a position (iterator) in hand  \*\*\* No `back()`; reaching the end takes O(n)

---

*This document is based on the C++11/14/17/20 standards; implementation details (libstdc++, libc++, MSVC STL) may differ slightly, but complexity guarantees are fixed by the standard.*
