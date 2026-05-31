# `<chrono>` — Time Measurement

| API | Priority | Description |
|-----|----------|-------------|
| `std::chrono::high_resolution_clock::now()` | memorize | Highest resolution clock. Wall clock. Usually same as steady_clock. Has syscall cost. |
| `std::chrono::steady_clock::now()` | memorize | Monotonic — never goes backward. Prefer for measuring time intervals. |
| `std::chrono::system_clock::now()` | know | Wall clock, can change with NTP. For log timestamps. Do not use for latency measurement. |
| `duration_cast<nanoseconds>(d)` | memorize | Duration conversion. Get ns value with `.count()` on `(end-start)`. |
| `nanoseconds` / `microseconds` / `milliseconds` | memorize | Duration types. Literals: `100ns`, `5us`, `1ms`. |
| `time_point` | know | Represents a specific point in time. Difference between two time_points gives a duration. |
| `std::chrono::seconds(n)` | know | Creates a duration of n seconds. Use with `sleep_for`. |

> In HFT, use `rdtsc` for real latency — chrono carries syscall cost. Use chrono to report benchmark results (which you're already doing correctly in your code).
