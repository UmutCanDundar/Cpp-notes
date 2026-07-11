# `<mutex>` `<shared_mutex>`

| API | Priority | Description |
|-----|----------|-------------|
| `std::mutex` | memorize | Basic mutual exclusion lock. Locking it twice from the same thread (without releasing) is UB (deadlock). |
| `std::recursive_mutex` | avoid | Allows the same thread to lock repeatedly — usually a sign of a design problem; prefer restructuring code to avoid needing it. |
| `std::timed_mutex` | know | Adds `try_lock_for`/`try_lock_until` — lock attempt with a timeout. |
| `std::shared_mutex` | know | Reader-writer lock — multiple concurrent readers OR one exclusive writer. Good when reads vastly outnumber writes. |
| `std::lock_guard<Mutex>` | memorize | Simplest RAII lock wrapper. Locks on construction, unlocks on destruction. No manual unlock/relock. |
| `std::unique_lock<Mutex>` | memorize | More flexible RAII lock — can be deferred, manually unlocked/relocked, moved, used with condition_variable. Slightly more overhead than lock_guard. |
| `std::scoped_lock` | memorize | C++17. Locks multiple mutexes at once, deadlock-avoiding algorithm built in. Prefer over manually locking multiple mutexes in a fixed order. |
| `std::shared_lock<SharedMutex>` | know | RAII shared/read lock for use with `shared_mutex`. |
| `std::lock(m1, m2, ...)` | know | Locks multiple mutexes together without deadlock, if you need to lock outside of `scoped_lock`'s RAII pattern. |
| `.try_lock()` | know | Non-blocking lock attempt, returns false immediately if unavailable instead of waiting. |
| Mutex on hot path | careful | Uncontended `std::mutex` is cheap (userspace futex fast path) but contended locking causes syscalls/context switches — consider lock-free (`atomic`) structures for the hottest paths. |
