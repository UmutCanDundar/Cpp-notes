# Questions to Ask Before Writing Code — In Priority Order

| # | Question |
|---|----------|
| 1 | **Latency, throughput, or both?** Everything changes based on the answer. Latency → single-threaded pipeline, lock-free, busy-wait. Throughput → multi-threaded, batch processing, async I/O. They may conflict. |
| 2 | **How many threads? Is there shared state?** Thread count and data sharing between them determines the architecture. Zero sharing → fastest. If sharing is necessary → lock-free, mutex, or message passing? |
| 3 | **Will memory allocation happen on the hot path?** malloc/new = syscall = unpredictable latency. If yes → object pool, arena allocator, stack allocation. If no, standard containers are fine. |
| 4 | **What is the data lifetime? Who owns it?** unique_ptr, shared_ptr, raw pointer, or stack? shared_ptr = atomic ref count = cache miss. If ownership is clear, unique_ptr or raw pointer is enough. |
| 5 | **What is the data size and access pattern?** Cache line is 64 bytes. Is frequently accessed data stored together (SoA vs AoS)? Hot and cold data separated? False sharing present? These affect throughput by 10x. |
| 6 | **Is exception handling needed?** Generally no in HFT. Compiled with `-fno-exceptions`. Error handling → return code, `std::expected`, or error state. Exception = zero-cost, but throw = very expensive. |
| 7 | **Will the interface stay fixed or change?** If it will change, add abstraction layers. If fixed, don't add unnecessary abstraction — keep it simple. Over-engineering is the enemy of latency. |
| 8 | **Is testability important?** Will dependency injection be used? Can it be mocked? HFT systems are generally hard to test — fake server, fake market data; your benchmark approach is exactly right. |
| 9 | **Will it need to scale?** 1 venue now, 10 venues later — what changes? Template parameter or runtime config? Design for it now but don't over-engineer. |
| 10 | **What happens on error?** Network drops, corrupted message, memory fills — all must have an answer. Graceful degradation, hard stop, or reconnect? |
