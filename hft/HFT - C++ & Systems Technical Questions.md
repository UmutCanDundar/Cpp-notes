# HFT — C++ & Systems Technical Questions

---

## Memory & Cache

**Q: What is false sharing and how do you fix it?**
Two threads write to different variables that share the same cache line (64 bytes). The CPU invalidates the entire line on every write, causing performance collapse even though there is no data race. Fix with `alignas(64)` or `std::hardware_destructive_interference_size`.

**Q: What is cache line size on x86-64?**
64 bytes. Always align hot data structures to 64 bytes in HFT.

**Q: What is the difference between AoS and SoA?**
- AoS (Array of Structs): `struct Point {float x,y,z}; Point arr[N]` — reading all x values requires jumping 12 bytes between each. Cache-unfriendly for SIMD.
- SoA (Struct of Arrays): `float xs[N], ys[N], zs[N]` — all x values are contiguous. Cache-friendly, SIMD-friendly.

**Q: What is a memory pool / object pool? Why use it in HFT?**
Pre-allocate a large block of memory upfront, hand out fixed-size chunks via a free list. `malloc`/`new` calls into the OS heap with unpredictable latency (~100ns+). Object pool: allocation = O(1), deterministic, no syscall.

**Q: What is placement new?**
`new(ptr) T(args)` — constructs an object into already-allocated memory without calling `malloc`. The foundation of object pools and arena allocators.

**Q: What is the difference between stack and heap allocation performance?**
Stack: single `sub rsp, N` instruction — effectively free. Heap: `malloc` goes through the heap allocator, possibly triggering a syscall. In HFT, avoid heap allocation on the hot path entirely.

**Q: What is an arena allocator?**
A bump-pointer allocator. Keeps a large pre-allocated buffer and a pointer to the next free byte. Allocation = pointer increment, O(1). Deallocation = reset the pointer (bulk free). No per-object free.

---

## Concurrency & Atomics

**Q: What is the difference between `memory_order_relaxed`, `acquire`, and `release`?**
- `relaxed`: atomicity only, no ordering guarantees. Use for counters.
- `release`: all writes before this store are visible to any thread that `acquire`-loads this variable.
- `acquire`: all reads after this load see everything that was `release`-stored.
- Together, acquire/release form a happens-before relationship without a full `mfence`.

**Q: Why is `memory_order_seq_cst` expensive?**
It generates a full `mfence` instruction on x86, which flushes the store buffer and stalls the pipeline. Use `acquire`/`release` instead when sequential consistency across all threads is not required.

**Q: What is a data race vs false sharing?**
- Data race: two threads access the same variable, at least one writes, no synchronization → undefined behavior.
- False sharing: two threads access different variables on the same cache line → no UB, but severe performance degradation.

**Q: What is compare-and-swap (CAS)?**
`compare_exchange_weak(expected, desired)` — atomically: if `*this == expected`, write `desired` and return true; else load current value into `expected` and return false. Maps to `lock cmpxchg` on x86. Foundation of all lock-free data structures.

**Q: Why use `compare_exchange_weak` in a loop instead of `strong`?**
`weak` allows spurious failures (returns false even when value matches) but is faster on some architectures (ARM). Since lock-free code already loops on failure, spurious failures cost nothing. `strong` is only useful for a single-attempt outside a loop.

**Q: What is `_mm_pause()` and when do you use it?**
Maps to the `PAUSE` instruction. In spin-wait loops, it hints to the CPU that this is a spin loop, reducing power consumption and improving performance when the other hyper-thread is doing real work. Also prevents memory order violations in tight loops.

**Q: What is a lock-free ring buffer and why is it central to HFT?**
A fixed-size circular buffer with a producer and consumer pointer. Producer writes, advances head. Consumer reads, advances tail. With `atomic` head/tail and `release`/`acquire` ordering, it is fully lock-free and allocation-free. Zero syscalls, zero mutex contention. The core of most HFT order pipelines.

---

## Low-Latency C++ Patterns

**Q: Why avoid virtual functions on the hot path?**
Virtual call = indirect call through vtable pointer → CPU cannot predict the target address → branch predictor miss → ~15 cycle penalty + possible i-cache miss. Replace with CRTP or `std::variant` + `std::visit`.

**Q: What is CRTP and what problem does it solve?**
Curiously Recurring Template Pattern. `template<class D> struct Base { void foo() { static_cast<D*>(this)->foo_impl(); } }`. Gives compile-time polymorphism with zero overhead. No vtable, fully inlined.

**Q: What is the difference between `std::function` and a raw function pointer / template lambda?**
`std::function`: type erasure via heap allocation + virtual call inside. ~50-100ns overhead. Never on hot path.
Raw function pointer: 8 bytes, single indirect call. Fast.
Template lambda: inlined by compiler. Zero overhead.

**Q: Why is `std::shared_ptr` problematic in HFT?**
Ref count is `atomic<int>` — every copy/destroy triggers an atomic increment/decrement, which is a cache-coherency operation. If the ref count block is on a hot cache line, performance collapses. Use `unique_ptr` or raw pointers with clear ownership.

**Q: What is `std::variant` + `std::visit` and why prefer it over virtual?**
`std::variant<A,B,C>` is a type-safe stack-allocated union. `std::visit` generates a compile-time jump table. No heap allocation, no vtable pointer per object, fully type-safe. Ideal for message dispatch in HFT where all message types are known at compile-time.

**Q: What compiler flags matter most for HFT?**
| Flag | Effect |
|------|--------|
| `-O3` | Full optimization |
| `-march=native` | Use all CPU features (AVX2, etc.) |
| `-funroll-loops` | Unroll loops |
| `-fno-exceptions` | Remove exception overhead |
| `-fno-rtti` | Remove RTTI overhead |
| `-flto` | Link-time optimization — cross-TU inlining |
| `-fPGO` | Profile-guided optimization |

---

## Profiling & Measurement

**Q: What is `rdtsc` and when do you use it instead of `chrono`?**
`rdtsc` reads the CPU's time-stamp counter in a single instruction (~1 cycle). `std::chrono::high_resolution_clock::now()` may call a syscall (`clock_gettime`). For sub-microsecond latency measurement on the hot path, use `rdtsc`. For benchmark reporting, `chrono` is fine.

**Q: How do you prevent compiler reordering around a benchmark measurement?**
```cpp
asm volatile("" ::: "memory");  // compiler memory barrier
// or
std::atomic_signal_fence(std::memory_order_acq_rel);
```
Also use `__asm__ volatile("mfence")` if you need CPU ordering.

**Q: What tools do you use to profile C++ HFT code?**
- `perf stat` / `perf record` — hardware counters, cache misses, branch mispredictions
- `perf c2c` — false sharing detection
- `valgrind --tool=cachegrind` — cache simulation
- `likwid-perfctr` — NUMA, cache, memory bandwidth
- `Godbolt` (compiler explorer) — inspect generated assembly
- `gprof` — call graph profiling
- `Intel VTune` — full pipeline analysis

**Q: What is a cache miss rate and what is acceptable in HFT?**
L1 cache: ~4 cycles. L2: ~12 cycles. L3: ~40 cycles. RAM: ~100-200 cycles. In HFT hot paths, L1 miss rate should be near zero. Profile with `perf stat -e cache-misses,cache-references`.

---

## Systems & OS

**Q: What is CPU affinity and why does it matter in HFT?**
Pinning a thread to a specific CPU core with `pthread_setaffinity_np` or `sched_setaffinity`. Prevents the OS from migrating the thread to another core (which would flush L1/L2 cache). In HFT, the order-processing thread is always pinned.

**Q: What is NUMA and why does it matter?**
Non-Uniform Memory Access. On multi-socket servers, RAM is local to one socket. Accessing memory on the remote socket takes ~2x longer. In HFT, allocate memory on the same NUMA node as the CPU doing the work (`numactl --membind`).

**Q: What is busy-waiting and when is it appropriate?**
Instead of sleeping (`sleep_for`, condition variable), the thread continuously polls (`while (!ready.load(acquire)) _mm_pause()`). Avoids OS scheduler latency (~50-100µs on wakeup). Appropriate in HFT when latency < 10µs is required and a dedicated core is available.

**Q: What is huge pages and why use them in HFT?**
Default page size = 4KB. Huge pages = 2MB or 1GB. With large working sets, 4KB pages cause many TLB misses (TLB = translation cache for virtual→physical address). Huge pages reduce TLB miss rate dramatically. Enable with `mmap(MAP_HUGETLB)` or `libhugetlbfs`.

**Q: What is the difference between a kernel bypass and a standard socket?**
Standard socket: send → kernel → NIC driver → wire. Latency ~1-10µs.
Kernel bypass (DPDK, Solarflare OpenOnload): userspace directly writes to NIC ring buffer. No syscall, no context switch. Latency ~100-200ns.
