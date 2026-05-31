# What Should NOT Be Used on the Hot Path

| API / Structure | Why It's Slow | Alternative |
|-----------------|---------------|-------------|
| `std::mutex` / `lock_guard` | Syscall, context switch | `std::atomic`, lock-free ring buffer |
| `std::condition_variable` | Syscall, sleep/wake latency | Atomic spin-wait, `_mm_pause()` |
| `std::shared_ptr` | Atomic ref count, cache miss | `unique_ptr`, raw pointer, index |
| `std::function` | Heap alloc, virtual call, type erasure | Template lambda, function pointer |
| `std::any` | Heap alloc, type erasure | `std::variant` |
| `std::list` / `std::deque` | Each node on heap, cache miss | `std::vector`, ring buffer |
| `std::map` / `std::set` | Tree node on heap, cache miss | Sorted vector + binary_search, custom hash map |
| `std::unordered_map` default | Rehash, bucket pointer chase | Custom open-addressing hash, flat hash map |
| `new` / `delete` on hot path | Syscall, heap lock, fragmentation | Object pool, arena allocator, stack allocation |
| `std::sort` on hot path | O(n log n), every iteration | Keep data sorted already, insertion sort for small N |
| `std::cout` / `cerr` on hot path | Syscall, lock, buffer flush | Lock-free logger, ring buffer logger |
| Virtual function | vtable indirect call, branch miss | CRTP, `std::variant` + visit, template |
| Exception throw | Very expensive (~1000 cycles) | Return code, `std::expected`, error enum |
| `memory_order_seq_cst` unnecessarily | mfence = full barrier | acquire/release is sufficient most of the time |
| `std::this_thread::sleep_for` | OS scheduler, sleep latency | Busy-wait + `_mm_pause()` |
| `std::bind` | Overhead, complexity | Lambda |

> **Rule:** If you see heap allocation, virtual call, syscall, or mutex on the hot path — question it. All of these can be eliminated or moved to a cold path.
