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
