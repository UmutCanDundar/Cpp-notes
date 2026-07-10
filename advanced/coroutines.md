# Coroutines (C++20)

Coroutines are functions that can suspend and resume execution, preserving state between suspensions. They enable elegant asynchronous code, generators, and cooperative multitasking without manual state machines or callbacks.

## Basics

A function becomes a coroutine if it uses `co_await`, `co_yield`, or `co_return`. The compiler transforms it into a state machine automatically.

```cpp
Task doWork() {
    co_await someAsyncOp();
}
```

## promise_type

Every coroutine needs an associated `promise_type` that defines how it behaves: what happens on start, suspend, return, and exceptions.

```cpp
struct Task {
    struct promise_type {
        Task get_return_object() { return {}; }
        std::suspend_never initial_suspend() { return {}; }
        std::suspend_never final_suspend() noexcept { return {}; }
        void return_void() {}
        void unhandled_exception() {}
    };
};
```

## Awaitables and Awaiters

An awaitable is anything usable with `co_await`; an awaiter implements `await_ready`, `await_suspend`, `await_resume` to control suspension behavior.

```cpp
struct MyAwaiter {
    bool await_ready() { return false; }
    void await_suspend(std::coroutine_handle<> h) { /* schedule resume */ }
    void await_resume() {}
};
```

## co_await

Suspends the coroutine until the awaited operation completes, without blocking the thread — the foundation of async/await style code.

```cpp
co_await asyncReadFile("data.txt");
```

## co_yield

Suspends the coroutine and produces a value to the caller, used to implement generators.

```cpp
Generator<int> range(int n) {
    for (int i = 0; i < n; ++i) co_yield i;
}
```

## co_return

Ends a coroutine, optionally returning a value (handled via the promise's `return_value`/`return_void`).

```cpp
Task<int> compute() { co_return 42; }
```

## Promise

(See promise_type above.) The promise object controls the coroutine's overall behavior and communication with the caller.

## Generators

A coroutine pattern that lazily produces a sequence of values via `co_yield`, one at a time, on demand.

```cpp
Generator<int> fib() {
    int a = 0, b = 1;
    while (true) { co_yield a; auto next = a + b; a = b; b = next; }
}
```

## Tasks

A coroutine pattern representing a deferred, awaitable computation (like a lightweight future), central to async C++ frameworks.

```cpp
Task<std::string> fetchData();
```

## Cooperative Multitasking

Coroutines let multiple tasks share a single thread by voluntarily suspending at `co_await` points, avoiding the overhead of OS thread context switches.
