
# `<semaphore>` `<latch>` `<barrier>` (C++20)

| API | Priority | Description |
|-----|----------|-------------|
| `std::counting_semaphore<Max>` | know | Allows up to `Max` concurrent holders of a resource — generalizes a mutex (which is like a semaphore with max=1). |
| `std::binary_semaphore` | know | Alias for `counting_semaphore<1>` — a lightweight signal/lock, often faster than a mutex+condition_variable for simple signaling. |
| `.acquire()` / `.release()` | know | Decrement/increment the semaphore's count, blocking on acquire if the count is zero. |
| `.try_acquire()` / `.try_acquire_for(d)` | know | Non-blocking or timed acquire attempts. |
| `std::latch` | know | A single-use countdown gate — threads call `.count_down()`, others call `.wait()` until the count reaches zero. Cannot be reset or reused. |
| `std::barrier` | know | A *reusable* synchronization point — a group of threads all wait at `.arrive_and_wait()` until every thread has arrived, then all proceed together, and the barrier resets for the next round. Optional completion callback runs when each phase finishes. |
| Semaphore vs mutex for signaling | know | For producer→consumer "wake up, data is ready" signaling, a `binary_semaphore` is often simpler and cheaper than a mutex + condition_variable pair. |
