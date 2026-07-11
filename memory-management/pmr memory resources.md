# `<memory_resource>` (PMR) — Polymorphic Memory Resources

## The problem PMR solves

Normally, containers like `std::vector<T>` allocate memory through `std::allocator<T>`, which just calls global `operator new` → `malloc`. Every `push_back` that triggers growth means a `malloc` call: a syscall-adjacent, lock-guarded, cache-unfriendly operation with unpredictable latency. In HFT/low-latency code this jitter is unacceptable — you want allocation to be a few cycles, not a trip through the general-purpose heap.

The traditional fix is a **custom allocator**: write your own `MyAllocator<T>` and use `std::vector<T, MyAllocator<T>>`. This works, but has a big downside — `vector<T, std::allocator<T>>` and `vector<T, MyAllocator<T>>` are **different types**. You can't pass one where the other is expected, every function template gets instantiated twice, and swapping the allocation strategy means changing types everywhere.

## What PMR actually is

`std::pmr` (C++17, `<memory_resource>`) fixes this by moving the allocation *strategy* from compile-time (template parameter) to **runtime** (a pointer to a polymorphic base class).

- `std::pmr::memory_resource` — an abstract base class with virtual `do_allocate` / `do_deallocate`. This is the interface any allocation strategy implements.
- `std::pmr::polymorphic_allocator<T>` — a thin allocator that just holds a `memory_resource*` and forwards allocation calls to it through a virtual call.
- `std::pmr::vector<T>` is simply `std::vector<T, std::pmr::polymorphic_allocator<T>>` — **one single type**, regardless of which memory_resource backs it. You can swap the underlying resource at runtime without changing the container's type.

```cpp
std::pmr::monotonic_buffer_resource arena(buffer, sizeof(buffer));
std::pmr::vector<int> v{&arena};   // same type as any other std::pmr::vector<int>
```

## PMR vs "an arena allocator"

PMR *is* the standard's official mechanism for plugging in an arena allocator — it's not an alternative to one, it's the interface. `std::pmr::monotonic_buffer_resource` **is** the standard library's built-in arena/bump allocator:

- It owns (or is given) a chunk of memory.
- Each `allocate()` call just bumps an internal pointer forward and returns the old position — no bookkeeping, no free-list, effectively O(1) with a handful of instructions.
- `deallocate()` on a monotonic_buffer_resource is a no-op for individual objects. You don't free one-by-one — you reset or destroy the whole arena at once, freeing everything it handed out in one shot.

This is exactly the "arena" pattern: allocate fast, don't bother freeing individually, reclaim everything together.

## Pool resource — the other PMR strategy

`std::pmr::unsynchronized_pool_resource` (and its thread-safe sibling `synchronized_pool_resource`) is a different strategy: it groups allocations into pools of same-sized chunks. Unlike the arena, you *can* deallocate individual objects — they go back into their size-class's free list for reuse. This suits workloads with many same-sized alloc/free cycles (e.g., order objects, small message structs) rather than "allocate a batch, discard together."

| Resource | Individual deallocate? | Speed | Best for |
|---|---|---|---|
| `monotonic_buffer_resource` | No (bulk reset only) | Fastest (bump pointer) | Per-tick/per-request scratch data, build-then-discard |
| `unsynchronized_pool_resource` | Yes | Fast (free-list per size class) | Frequent alloc/dealloc of similar-sized objects, single-threaded |
| `synchronized_pool_resource` | Yes | Fast but has locking | Same as above but shared across threads |
| `new_delete_resource()` | Yes | Slow (regular malloc/free) | Default fallback, non-critical paths |

## Difference vs a plain `std::vector`

A plain `std::vector<int> v;` uses `std::allocator<int>`, which means every reallocation calls `malloc`/`free` through the global heap — general-purpose, thread-safe, but comparatively slow and non-deterministic in latency.

A `std::pmr::vector<int> v{&arena};` has the *exact same API and behavior* as a normal vector — same growth strategy, same iterators — but every allocation it does is redirected to your `arena` object instead of the global heap. If `arena` is a `monotonic_buffer_resource` wrapping a stack or pre-reserved buffer, growing the vector costs a pointer bump instead of a heap call.

```cpp
// Hot-path pattern: reuse one arena per tick, reset instead of freeing
std::byte buffer[64 * 1024];
std::pmr::monotonic_buffer_resource arena(buffer, sizeof(buffer));

void onTick() {
    arena.release();               // "free" everything from last tick, O(1)
    std::pmr::vector<Order> orders{&arena};
    // ... build orders using zero real heap allocations ...
}
```

## Key API summary

| API | Priority | Description |
|-----|----------|-------------|
| `std::pmr::memory_resource` | know | Abstract interface for an allocation strategy (`do_allocate`/`do_deallocate`). |
| `std::pmr::polymorphic_allocator<T>` | memorize | Allocator that holds a `memory_resource*`, used by all `std::pmr::*` containers. |
| `std::pmr::monotonic_buffer_resource` | memorize | Bump-pointer arena. Fastest option. No per-object dealloc, `.release()` frees everything at once. |
| `std::pmr::unsynchronized_pool_resource` | know | Pooled, fixed-size-class allocator with real per-object deallocation. Single-threaded. |
| `std::pmr::synchronized_pool_resource` | know | Same as above, thread-safe (adds locking overhead). |
| `std::pmr::new_delete_resource()` | know | Falls back to global `new`/`delete` — the default if no resource is given. |
| `std::pmr::null_memory_resource()` | know | Always throws `bad_alloc` — used to assert "this code path must never allocate." |
| `std::pmr::vector<T>` / `std::pmr::string` / `std::pmr::map<K,V>` | memorize | Standard containers pre-aliased to use `polymorphic_allocator<T>`, drop-in replacements for their `std::` counterparts. |
| `.resource()` | know | Returns the `memory_resource*` a `polymorphic_allocator` is using — for introspection/passing along to nested allocations. |

## Bottom line

- **Arena allocator** = the *concept* (allocate fast, free in bulk).
- `monotonic_buffer_resource` = the standard library's *implementation* of that concept.
- **PMR** = the *plumbing* (`memory_resource` + `polymorphic_allocator`) that lets any container use that arena — or a pool, or the regular heap — without changing the container's type. That's very likely what you were seeing used: `std::pmr::vector` backed by a `monotonic_buffer_resource`, not a plain `std::vector`.
