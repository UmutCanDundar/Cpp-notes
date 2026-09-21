# `<chrono>` — Time Measurement

### Notation
* `Rep`: the arithmetic type used to store a duration's tick count (e.g. `int64_t`, `double`).
* `Period`: a `std::ratio` describing the tick period in seconds (e.g. `std::milli` = 1/1000s).
* `Clock`: one of `system_clock`, `steady_clock`, `high_resolution_clock` (or a user-defined clock).
* `Duration`, `ToDuration`: a `std::chrono::duration<Rep, Period>` specialization.
* `d`, `d1`, `d2`: instances of a `duration`.
* `tp`, `tp1`, `tp2`: instances of a `time_point`.

---

### Constructors

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| *(constructor)* | duration, default | `duration<Rep,Period> d;` | O(1) | Default-constructs; the stored value is default-initialized (zero for arithmetic `Rep`, but not guaranteed for a class `Rep`). |
| *(constructor)* | duration, from rep | `duration<Rep,Period> d(count)` | O(1) | Constructs from a raw tick count `count` of type `Rep2`. Explicit unless `Rep2` converts losslessly (e.g. integral to floating-point `Rep`). |
| *(constructor)* | duration, converting | `duration<Rep,Period> d(other_duration)` | O(1) | Constructs from another `duration` with a different `Rep`/`Period`, doing the ratio conversion. Implicit only if no precision would be lost (e.g. seconds → milliseconds, not the reverse). |
| *(constructor)* | time_point, default | `time_point<Clock,Duration> tp;` | O(1) | Default-constructs to the clock's epoch (`duration` of zero). |
| *(constructor)* | time_point, from duration | `time_point<Clock,Duration> tp(d)` | O(1) | Constructs a point in time that is `d` after the clock's epoch. Marked `explicit`. |
| *(constructor)* | time_point, converting | `time_point<Clock,Duration> tp(other_tp)` | O(1) | Constructs from another `time_point` of the **same** `Clock` but a different `Duration`, converting the duration. |

---

### Methods

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| `time_point` | now() | `high_resolution_clock::now()`<br>`steady_clock::now()`<br>`system_clock::now()` | O(1) (carries syscall cost) | Highest-resolution clock (usually aliases `steady_clock`) · monotonic, never goes backward — prefer for measuring intervals · wall clock, can jump with NTP — use for log timestamps, **not** for latency measurement. |
| `bool` *(static constexpr member, not a call)* | is_steady | `Clock::is_steady` | O(1) | `true` only for `steady_clock`. Check before trusting a custom/unknown clock for interval measurement. |
| `time_t` | to_time_t | `system_clock::to_time_t(tp)` | O(1) | Converts a `system_clock::time_point` to a C-style `time_t` (`system_clock` only — other clocks have no calendar meaning). |
| `time_point` | from_time_t | `system_clock::from_time_t(t)` | O(1) | Converts a C-style `time_t` back to a `system_clock::time_point`. |
| `Rep` | count | `d.count()` | O(1) | Returns the raw tick count stored in the duration. |
| `duration` *(static)* | zero / min / max | `duration<Rep,Period>::zero()`<br>`duration<Rep,Period>::min()`<br>`duration<Rep,Period>::max()` | O(1) | The zero value · smallest (most negative) representable value · largest representable value for this duration type. |
| `duration` | operator+ / operator- (unary) | `+d` / `-d` | O(1) | Returns a copy unchanged · returns a copy with the tick count negated. |
| `duration&` / `duration` | operator++ / operator-- | `++d` / `d++`<br>`--d` / `d--` | O(1) | Increments/decrements the tick count by exactly 1 (not 1 second — 1 tick of whatever `Period` is). Pre-forms return `duration&`, post-forms return `duration`. |
| `duration&` | operator+= / operator-= | `d1 += d2` / `d1 -= d2` | O(1) | Adds/subtracts another duration in place (after an implicit common-period conversion). |
| `duration&` | operator*= / operator/= / operator%= | `d *= n` / `d /= n` / `d %= n` | O(1) | Scales the tick count by a `Rep`-convertible value `n`, or (for `/=`/`%=`) divides by another duration. |
| `duration` | time_since_epoch | `tp.time_since_epoch()` | O(1) | Returns the `duration` elapsed since the clock's epoch. |
| `time_point&` | operator+= / operator-= | `tp += d` / `tp -= d` | O(1) | Advances/rewinds the time point by a duration, in place. |
| `time_point` *(static)* | min / max | `time_point<Clock,Duration>::min()`<br>`time_point<Clock,Duration>::max()` | O(1) | Smallest / largest representable time point for this specialization. |
| `ToDuration` | duration_cast\<ToDuration\> | `duration_cast<nanoseconds>(d)` | O(1) | Converts between duration types. Always **truncates toward zero** — get the value with `.count()`. |
| `time_point<Clock,ToDuration>` | time_point_cast\<ToDuration\> | `time_point_cast<milliseconds>(tp)` | O(1) | Same conversion as `duration_cast`, but for a `time_point`. |
| `Duration` (C++17) | floor\<Duration\> / ceil\<Duration\> / round\<Duration\> | `floor<seconds>(d)`<br>`ceil<seconds>(d)`<br>`round<seconds>(d)` | O(1) | Unlike `duration_cast`'s truncation, these round down · round up · round to nearest (ties to even). Work on both `duration` and `time_point`. |
| `Duration` (C++17) | abs | `abs(d)` | O(1) | Absolute value of a signed duration. |

---

### Duration Types & Literals

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| *(type alias)* | nanoseconds / microseconds / milliseconds / seconds / minutes / hours | `nanoseconds`, `microseconds`, `milliseconds`, `seconds`, `minutes`, `hours` | N/A | Predefined `duration<Rep, Period>` specializations. |
| `duration` | `chrono_literals` | `100ns`, `5us`, `1ms`, `2s`, `1min`, `1h` | O(1) | Literal suffixes (need `using namespace std::chrono_literals`). |
| `duration` *(constructor)* | seconds(n) | `std::chrono::seconds(n)` | O(1) | Creates a duration of `n` seconds. Commonly passed to `std::this_thread::sleep_for`. |

> **In HFT contexts:** use `rdtsc` for real latency measurement — `chrono` (even `steady_clock`) carries syscall cost. 
