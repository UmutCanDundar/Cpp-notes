# `<thread>` / `<mutex>` / `<condition_variable>`

| API | Priority | Description |
|-----|----------|-------------|
| `std::thread(fn, args...)` | memorize | Create a thread. Must call `join()` or `detach()`. |
| `.join()` | memorize | Wait until the thread finishes. |
| `.detach()` | know | Release the thread. You lose control of it. |
| `std::this_thread::get_id()` | know | Current thread ID. |
| `std::this_thread::sleep_for(dur)` | careful | Sleep the thread. Never on hot path. Fine in init/teardown. |
| `std::this_thread::yield()` | careful | Release the CPU. Prefer `_mm_pause()` in spin-wait loops. |
| `std::mutex` | avoid | Never on hot path. Syscall + context switch risk. Use lock-free structures. |
| `std::lock_guard<mutex>` | careful | RAII mutex. Fine on cold paths (init, config, log). |
| `std::unique_lock<mutex>` | careful | Used with condition variables. lock/unlock can be called manually. |
| `std::condition_variable` | avoid | Never on hot path. Syscall. Only for slow control paths. |
| `std::jthread` | know | C++20. Auto join, stop_token support. |
| `std::hardware_destructive_interference_size` | memorize | Cache line size (usually 64). Use with `alignas` to prevent false sharing. |
