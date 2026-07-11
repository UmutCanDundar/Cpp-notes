# `<thread>`

| API | Priority | Description |
|-----|----------|-------------|
| `std::thread` | memorize | Represents an OS thread of execution. Must be `.join()`ed or `.detach()`ed before destruction, or `std::terminate` is called. |
| `std::jthread` | memorize | C++20 "joining thread" — auto-joins on destruction and supports cooperative cancellation via `stop_token`. Prefer over raw `std::thread`. |
| `.join()` | memorize | Blocks the calling thread until the target thread finishes. |
| `.detach()` | careful | Thread runs independently, no longer joinable — dangerous if it outlives objects it references; prefer `jthread` + explicit lifetime management. |
| `.get_id()` | know | Returns a unique `std::thread::id`, usable as a map key or for logging. |
| `std::thread::hardware_concurrency()` | know | Hint for the number of concurrent threads supported (e.g. core count) — use to size thread pools, not a hard guarantee. |
| `std::hardware_destructive_interference_size` | memorize | Cache line size (usually 64). Use with `alignas` to prevent false sharing. |
| `std::this_thread::get_id()` | know | Gets the current thread's id. |
| `std::this_thread::sleep_for(d)` / `sleep_until(tp)` | know | Blocks the current thread for a duration or until a time point. |
| `std::this_thread::yield()` | know | Hints the scheduler to let other threads run — used in spin-wait loops to reduce contention. |
| `thread_local` | memorize | Storage keyword (not strictly this header) giving each thread its own independent instance of a variable. |
