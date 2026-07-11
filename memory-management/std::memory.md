# `<memory>` — Smart Pointers and Allocators

| API | Priority | Description |
|-----|----------|-------------|
| `std::unique_ptr<T>` | memorize | Single owner. Zero overhead — same as raw pointer except for the destructor. Prefer this. |
| `std::make_unique<T>(args)` | memorize | Creates unique_ptr. Use instead of new — exception safe. |
| `.get()` | memorize | Get raw pointer. For passing to C APIs. |
| `.release()` | know | Give up ownership, returns raw pointer. unique_ptr no longer deletes. |
| `.reset(ptr)` | know | Delete old, take ownership of new. |
| `std::shared_ptr<T>` | avoid | Ref count is atomic — cache miss + atomic overhead on hot path. Only use if ownership truly needs to be shared. |
| `std::weak_ptr<T>` | careful | Non-owning reference to shared_ptr. Breaks circular dependencies. `lock()` call has overhead. |
| `std::make_shared<T>` | careful | Object and ref count in a single allocation. Prefer if you must use shared_ptr. |
| `std::allocator<T>` | know | Default allocator for STL containers. Base for writing custom allocators. |
| `std::allocator_traits<A>` | know | Custom allocator interface. allocate/deallocate/construct/destroy. |
| `std::align(align, size, ptr, space)` | know | Align a pointer. Used when writing arena allocators. |
| `::operator new(n, std::nothrow)` | know | Returns nullptr instead of throwing exception on allocation failure. |
| placement new | memorize | `new(ptr) T(args)` — no allocation, constructs into existing memory. The foundation of object pools. |
| `std::pmr::polymorphic_allocator<T>` | know | C++17. Runtime-polymorphic allocator, lets you swap allocation strategy without changing the container's type. |
| `std::pmr::monotonic_buffer_resource` | memorize | Bump-pointer arena over a fixed buffer — near-zero allocation cost, ideal for per-tick/per-request scratch memory in hot loops. |
| `std::pmr::unsynchronized_pool_resource` | know | Pooled allocator for fixed-size objects, avoids `malloc` overhead for frequent same-size allocations. |
| `std::uninitialized_copy` / `uninitialized_fill` | know | Construct objects into raw, unconstructed memory — used when implementing custom containers. |
| `std::construct_at` / `std::destroy_at` | know | C++20. Placement-new/destroy a single object at a given address, constexpr-friendly. |
| `std::addressof(x)` | know | Gets the true address of x even if `operator&` is overloaded. Safer than `&x` in generic code. |
