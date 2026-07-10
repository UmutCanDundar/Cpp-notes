# `<atomic>` — Foundation of HFT, Memorize

| API | Priority | Description |
|-----|----------|-------------|
| `std::atomic<T>` | memorize | Lock-free variable. T must be trivially copyable. Guaranteed lock-free for bool, int, uint64_t, pointer. |
| `.load(order)` | memorize | Atomic read. Use `memory_order_acquire` or `relaxed`. |
| `.store(val, order)` | memorize | Atomic write. Use `memory_order_release` or `relaxed`. |
| `.fetch_add(n, order)` | memorize | Atomic add, returns old value. For sequence counters. → `lock xadd`. |
| `.fetch_sub` / `fetch_and` / `fetch_or` | know | Atomic arithmetic/bitwise operations. |
| `.compare_exchange_weak(exp, des, order)` | memorize | CAS. Use in a loop (spurious failure possible). → `lock cmpxchg`. Foundation of lock-free structures. |
| `.compare_exchange_strong` | know | No spurious failure but slower. For single attempt outside a loop. |
| `.exchange(val, order)` | know | Write and return old value. Atomic swap. |
| `.is_lock_free()` | know | Check if truly lock-free. May return false for 128-bit structs. |
| `std::atomic_thread_fence(order)` | memorize | Standalone memory barrier. Maps to mfence/lfence/sfence. |
| `std::atomic_signal_fence(order)` | know | Only prevents compiler reordering, not a CPU barrier. For signal handler synchronization. |
| `memory_order_relaxed` | memorize | No ordering guarantee, only atomicity. For counters and independent variables. |
| `memory_order_acquire` | memorize | Reads/writes after this load cannot move before it. Consumer side. |
| `memory_order_release` | memorize | Reads/writes before this store cannot move after it. Producer side. |
| `memory_order_seq_cst` | careful | Strongest guarantee, slowest. Total order across all threads. Default, but produces mfence on hot path. |
| `memory_order_acq_rel` | know | Both acquire and release. For RMW operations (fetch_add etc.). |
