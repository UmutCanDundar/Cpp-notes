# Concurrency in C++

Concurrency lets programs do multiple things at once, improving performance on multi-core hardware. C++ provides a standard threading and synchronization library so you don't need OS-specific APIs.

## Memory Model

Defines how threads observe memory operations performed by other threads, forming the foundation for correct concurrent code.

## std::thread

Represents a single thread of execution, running a function concurrently with the caller.

```cpp
std::thread t([] { std::cout << "hello from thread"; });
t.join();
```

## std::this_thread Namespace

Provides functions operating on the current thread, like sleeping or getting its ID.

```cpp
std::this_thread::sleep_for(std::chrono::milliseconds(100));
```

## std::jthread

A "joining thread" (C++20) that automatically joins on destruction and supports cooperative cancellation via stop tokens.

```cpp
std::jthread t([](std::stop_token st) {
    while (!st.stop_requested()) { /* work */ }
});
```

## std::stop_token

Allows cooperative cancellation of a `jthread`, letting the thread check periodically if it should stop.

```cpp
void work(std::stop_token st) {
    while (!st.stop_requested()) { /* ... */ }
}
```

## thread_local Storage

Declares a variable with a separate instance per thread, avoiding shared-state bugs for per-thread data.

```cpp
thread_local int counter = 0; // each thread has its own counter
```

## Race Condition & Data Race

A race condition is when outcome depends on timing of concurrent operations. A data race specifically is unsynchronized concurrent access to the same memory with at least one write — undefined behavior in C++.

```cpp
int counter = 0;
// two threads doing counter++ without synchronization = data race
```

## Standard Mutex Classes

`std::mutex`, `std::recursive_mutex`, `std::timed_mutex`, etc. provide mutual exclusion to protect shared data.

```cpp
std::mutex m;
m.lock(); /* critical section */ m.unlock();
```

## std::lock_guard

A simple RAII wrapper that locks a mutex on construction and unlocks on destruction.

```cpp
std::lock_guard<std::mutex> lock(m); // auto-unlocked at scope end
```

## std::unique_lock

Like `lock_guard` but more flexible: can be locked/unlocked manually, deferred, or used with condition variables.

```cpp
std::unique_lock<std::mutex> lock(m);
cv.wait(lock);
```

## std::scoped_lock

Locks multiple mutexes at once safely (avoiding deadlock from lock ordering), released automatically at scope end.

```cpp
std::scoped_lock lock(m1, m2); // locks both, deadlock-safe
```

## std::shared_lock

Used with `std::shared_mutex` for reader-writer locking, allowing multiple concurrent readers.

```cpp
std::shared_mutex sm;
std::shared_lock lock(sm); // shared/read lock
```

## std::condition_variable

Lets threads wait efficiently until notified by another thread, typically used with a mutex and a predicate.

```cpp
std::condition_variable cv;
cv.wait(lock, [] { return ready; });
cv.notify_one();
```

## Deadlocks

Occur when two or more threads wait on each other's locked resources forever, none able to proceed. Avoided via consistent lock ordering or `std::scoped_lock`.

## Livelocks

Threads keep changing state in response to each other without making progress, unlike a deadlock where they're simply stuck.

## std::once_flag & std::call_once

Ensures a piece of code runs exactly once, even if called from multiple threads concurrently — useful for lazy singleton initialization.

```cpp
std::once_flag flag;
std::call_once(flag, [] { std::cout << "init once"; });
```

## std::promise

Lets one thread set a value (or exception) to be retrieved later by another thread via a matching `std::future`.

```cpp
std::promise<int> p;
std::future<int> f = p.get_future();
p.set_value(42);
```

## std::future, std::shared_future

`std::future` retrieves a value asynchronously (once). `std::shared_future` allows multiple threads/readers to retrieve the same result.

```cpp
int result = f.get(); // blocks until value is ready
```

## std::async

Runs a function asynchronously (possibly on a new thread) and returns a `std::future` for its result.

```cpp
std::future<int> f = std::async(std::launch::async, [] { return compute(); });
```

## std::packaged_task

Wraps a callable so its result can be retrieved via a `std::future`, useful for building custom task queues/thread pools.

```cpp
std::packaged_task<int()> task([] { return 5; });
std::future<int> f = task.get_future();
std::thread(std::move(task)).detach();
```

## std::atomic

Provides lock-free (on most platforms), thread-safe operations on simple types without needing a mutex.

```cpp
std::atomic<int> counter{0};
counter.fetch_add(1, std::memory_order_relaxed);
```

## Atomic Operations

Operations like `load`, `store`, `fetch_add`, `compare_exchange` that are guaranteed to execute indivisibly with respect to other threads.

## Sequenced Before / Happens Before / Synchronized With

These formal memory-model relations define the order in which threads observe each other's operations, forming the basis of the C++ memory model's correctness guarantees.

## std::atomic<shared_ptr>

Since C++20, `std::shared_ptr` can be wrapped in `std::atomic` for safe concurrent access to the pointer itself, avoiding manual locking.

```cpp
std::atomic<std::shared_ptr<Widget>> globalWidget;
```

## Thread Pools

A pool of reusable worker threads that execute submitted tasks, avoiding the overhead of creating/destroying threads repeatedly. Not standardized yet but commonly built from `std::thread` + queue.

## Sequential Consistency

The default, strongest memory ordering: all threads see all atomic operations in the same global order, easiest to reason about but potentially slower.

```cpp
std::atomic<int> x{0};
x.store(1, std::memory_order_seq_cst); // default ordering
```

## Acquire Release Semantics

A weaker, faster memory ordering: a release store makes prior writes visible to a thread that does a matching acquire load, without full sequential consistency overhead.

```cpp
flag.store(true, std::memory_order_release);
// other thread:
while (!flag.load(std::memory_order_acquire));
```

## std::counting_semaphore

Allows a limited number of threads to access a resource concurrently, generalizing a mutex to allow more than one holder.

```cpp
std::counting_semaphore<3> sem(3); // up to 3 concurrent accesses
sem.acquire(); /* ... */ sem.release();
```

## std::binary_semaphore

A semaphore with only two states (like a lightweight signal/lock), a special case of `counting_semaphore<1>`.

```cpp
std::binary_semaphore sig(0);
sig.release(); // signal
sig.acquire(); // wait for signal
```

## Parallel STL Algorithms

Many STL algorithms accept an execution policy (`std::execution::par`, etc.) to run in parallel across multiple threads automatically.

```cpp
std::sort(std::execution::par, v.begin(), v.end()); // parallel sort
```

## Concurrency Idioms & Techniques

Patterns like producer-consumer queues, double-checked locking, read-write locks, and lock-free data structures used to build correct, efficient concurrent systems.
