# C++ Traits Reference Guide (C++11 → C++26)

Traits are compile-time interfaces that extract metadata, standardize operations, and provide fallback implementations for generic components (such as allocators and iterators) without enforcing rigid inheritance hierarchies.

---

## 1. `<memory>` — `std::allocator_traits`

`std::allocator_traits<Alloc>` is the central interface for memory management in C++ containers. It wraps an allocator type `Alloc` and provides default implementations for optional allocator features.

### A. Member Types

| Member Type | Description & Fallback Mechanism |
| :--- | :--- |
| `allocator_type` | The wrapped allocator type `Alloc`. |
| `value_type` | `Alloc::value_type`. The object type allocated by the allocator. |
| `pointer` | `Alloc::pointer` if present; otherwise defaults to `value_type*`. |
| `const_pointer` | `Alloc::const_pointer` if present; otherwise defaults to `std::pointer_traits<pointer>::pointer_to_const<value_type>`. |
| `void_pointer` | `Alloc::void_pointer` if present; otherwise defaults to `std::pointer_traits<pointer>::rebind<void>`. |
| `const_void_pointer` | `Alloc::const_void_pointer` if present; otherwise defaults to `std::pointer_traits<pointer>::rebind<const void>`. |
| `difference_type` | `Alloc::difference_type` if present; otherwise defaults to `std::pointer_traits<pointer>::difference_type` (`std::ptrdiff_t`). |
| `size_type` | `Alloc::size_type` if present; otherwise defaults to `std::make_unsigned_t<difference_type>` (`std::size_t`). |
| `propagate_on_container_copy_assignment` | `Alloc::propagate_on_container_copy_assignment` if present; otherwise defaults to `std::false_type`. |
| `propagate_on_container_move_assignment` | `Alloc::propagate_on_container_move_assignment` if present; otherwise defaults to `std::false_type`. |
| `propagate_on_container_swap` | `Alloc::propagate_on_container_swap` if present; otherwise defaults to `std::false_type`. |
| `is_always_equal` | `Alloc::is_always_equal` if present; otherwise defaults to `std::is_empty<Alloc>::type`. |
| `rebind_alloc<U>` | `Alloc::rebind<U>::other` if present; otherwise defaults to `Alloc<U, Args...>` if `Alloc` is a template. |
| `rebind_traits<U>` | Alias for `std::allocator_traits<rebind_alloc<U>>`. |

### B. Static Member Functions

| Static Function | Description & Fallback Mechanism |
| :--- | :--- |
| `allocate(a, n)` | Calls `a.allocate(n)`. Allocates uninitialized storage for `n` objects of type `value_type`. |
| `allocate(a, n, hint)` | Calls `a.allocate(n, hint)` if valid; otherwise falls back to calling `a.allocate(n)`. |
| `deallocate(a, p, n)` | Calls `a.deallocate(p, n)`. Frees uninitialized storage pointed to by `p`. |
| `construct(a, p, args...)` | Calls `a.construct(p, args...)` if valid; otherwise falls back to placement new: `::new (static_cast<void*>(p)) T(std::forward<Args>(args)...)`. |
| `destroy(a, p)` | Calls `a.destroy(p)` if valid; otherwise falls back to direct destructor call: `p->~T()`. |
| `max_size(a)` | Calls `a.max_size()` if valid; otherwise defaults to returning `std::numeric_limits<size_type>::max() / sizeof(value_type)`. |
| `select_on_container_copy_construction(a)` | Calls `a.select_on_container_copy_construction()` if valid; otherwise returns `a` as-is. |

---

## 2. `<iterator>` — `std::iterator_traits`

`std::iterator_traits<Iter>` enables generic algorithms to query metadata about iterators, allowing custom iterator classes and raw pointers (`T*`) to be used with the exact same template logic.

### A. Member Types

| Member Type | Description for Class Iterators | Partial Specialization for Raw Pointers (`T*`) | Partial Specialization for Const Pointers (`const T*`) |
| :--- | :--- | :--- | :--- |
| `difference_type` | `Iter::difference_type` | `std::ptrdiff_t` | `std::ptrdiff_t` |
| `value_type` | `Iter::value_type` | `std::remove_cv_t<T>` | `std::remove_cv_t<T>` |
| `pointer` | `Iter::pointer` | `T*` | `const T*` |
| `reference` | `Iter::reference` | `T&` | `const T&` |
| `iterator_category` | `Iter::iterator_category` | `std::random_access_iterator_tag` | `std::random_access_iterator_tag` |
| `iterator_concept` *(C++20)* | `Iter::iterator_concept` if present; otherwise falls back to `iterator_category` | `std::contiguous_iterator_tag` | `std::contiguous_iterator_tag` |

### B. Standard Iterator Category Tags

| Category Tag Class | Base Category Class | Operational Capabilities |
| :--- | :--- | :--- |
| `input_iterator_tag` | *(None)* | Single-pass read-only traversal (`*it`, `++it`). |
| `output_iterator_tag` | *(None)* | Single-pass write-only traversal (`*it = val`, `++it`). |
| `forward_iterator_tag` | `input_iterator_tag` | Multi-pass read/write forward traversal. |
| `bidirectional_iterator_tag` | `forward_iterator_tag` | Multi-pass forward and backward traversal (`--it`). |
| `random_access_iterator_tag` | `bidirectional_iterator_tag` | Constant-time arbitrary offset movement (`it + n`, `it[n]`, `<=>`). |
| `contiguous_iterator_tag` *(C++20)* | `random_access_iterator_tag` | Guarantees elements are stored contiguously in physical memory. |

---

## 3. `<memory>` — `std::pointer_traits`

`std::pointer_traits<Ptr>` abstracts raw and fancy pointers (e.g., offsets, shared memory pointers, custom smart pointers) for node-based containers.

### A. Member Types

| Member Type | Description & Fallback Mechanism |
| :--- | :--- |
| `pointer` | The wrapped pointer type `Ptr`. |
| `element_type` | `Ptr::element_type` if defined; otherwise `T` if `Ptr` is `Template<T, Args...>`. |
| `difference_type` | `Ptr::difference_type` if defined; otherwise defaults to `std::ptrdiff_t`. |
| `rebind<U>` | `Ptr::rebind<U>` if defined; otherwise defaults to `Template<U, Args...>`. |

### B. Static Member Functions

| Static Function | Description |
| :--- | :--- |
| `pointer_to(r)` | Returns a pointer to element `r`. Calls `Ptr::pointer_to(r)` for fancy pointers, or `std::addressof(r)` for raw pointers. |

---

## 4. `<ranges>` — Range Traits & Aliases (C++20)

C++20 introduces range trait aliases under the `std::ranges` namespace to simplify type extraction from containers and ranges.

### Member Aliases

| Trait Alias | Resolved Underlying Type |
| :--- | :--- |
| `std::ranges::iterator_t<R>` | `decltype(std::ranges::begin(std::declval<R&>()))` |
| `std::ranges::sentinel_t<R>` | `decltype(std::ranges::end(std::declval<R&>()))` |
| `std::ranges::range_value_t<R>` | `std::iter_value_t<std::ranges::iterator_t<R>>` |
| `std::ranges::range_reference_t<R>` | `std::iter_reference_t<std::ranges::iterator_t<R>>` |
| `std::ranges::range_rvalue_reference_t<R>` | `std::iter_rvalue_reference_t<std::ranges::iterator_t<R>>` |
| `std::ranges::range_difference_t<R>` | `std::iter_difference_t<std::ranges::iterator_t<R>>` |
