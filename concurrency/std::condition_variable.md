# `<condition_variable>`

| API | Priority | Description |
|-----|----------|-------------|
| `std::condition_variable` | memorize | Lets a thread block until notified, used with `std::unique_lock<std::mutex>`. Foundation of producer-consumer patterns. |
| `.wait(lock)` | know | Blocks, atomically releasing the lock, until notified — then reacquires the lock before returning. |
| `.wait(lock, predicate)` | memorize | Preferred form — loops internally, protects against spurious wakeups by rechecking `predicate()` after each wake. |
| `.notify_one()` | memorize | Wakes exactly one waiting thread — use when only one waiter needs to proceed (e.g. one item produced). |
| `.notify_all()` | know | Wakes all waiting threads — use when the condition change could satisfy multiple waiters. |
| `.wait_for(lock, duration)` / `.wait_until(lock, tp)` | know | Bounded wait, returns a status indicating whether it timed out or was notified. |
| `std::condition_variable_any` | know | Works with any lock type (not just `unique_lock<mutex>`), at the cost of extra overhead — use the plain version unless you need this flexibility. |
| Missed wakeup bug | careful | Calling `notify` before the other thread starts `wait` loses the signal — always guard the actual condition with a checked predicate, not just timing. |
