# False Sharing — Complete Reference Table

> **Cache line = 64 bytes.** False sharing occurs when two threads access **different variables that share the same cache line**. Even if there is no data race, the cache line is invalidated on every write — causing performance loss.

## Summary Table

| # | Thread 0 | Thread 1 | Same Cache Line? | Same Variable? | Non-atomic | Atomic | Violation |
|---|----------|----------|-----------------|----------------|------------|--------|-----------|
| 1 | write M1 | write M1 | ✅ yes | ✅ yes | ❌ Race + ❌ False sharing | ✅ No race + ❌ False sharing | Data race / False sharing |
| 2 | write M1 | read M1  | ✅ yes | ✅ yes | ❌ Race + ❌ False sharing | ✅ No race + ❌ False sharing | Data race / False sharing |
| 3 | write M1 | read M2  | ✅ yes | ❌ no  | ✅ No race + ❌ False sharing | ✅ No race + ❌ False sharing | False sharing |
| 4 | write M1 | write M2 | ✅ yes | ❌ no  | ✅ No race + ❌ False sharing | ✅ No race + ❌ False sharing | False sharing (classic) |
| 5 | read M1  | read M1  | ✅ yes | ✅ yes | ✅ No race + ✅ No false sharing | ✅ No race + ✅ No false sharing | NONE |
| 6 | read M1  | read M2  | ✅ yes | ❌ no  | ✅ No race + ✅ No false sharing | ✅ No race + ✅ No false sharing | NONE |
| 7 | write M1 | write M2 | ❌ no  | ❌ no  | ✅ No race + ✅ No false sharing | ✅ No race + ✅ No false sharing | NONE |
| 8 | write M1 | read M2  | ❌ no  | ❌ no  | ✅ No race + ✅ No false sharing | ✅ No race + ✅ No false sharing | NONE |

> **M1, M2** = different variables. Same cache line means they are allocated within the same 64-byte block.

---

## Diagnosis — Which Problem Is It?

| Symptom | Cause | Fix |
|---------|-------|-----|
| Correct result but slow | False sharing (no data race) | `alignas(64)` — separate cache lines |
| Incorrect result | Data race | `std::atomic` or mutex |
| Both incorrect result and slow | Data race + false sharing | `std::atomic` + `alignas(64)` |
| Correct result and fast | No issue | — |

---

## Fix Examples

```cpp
// ❌ PROBLEM — a and b on same cache line, written by different threads
struct Counters {
    int a;  // thread 0 writes
    int b;  // thread 1 writes — same cache line, false sharing!
};

// ✅ FIX 1 — alignas separates cache lines
struct Counters {
    alignas(64) int a;  // thread 0
    alignas(64) int b;  // thread 1 — separate cache lines
};

// ✅ FIX 2 — C++17 hardware_destructive_interference_size
struct Counters {
    alignas(std::hardware_destructive_interference_size) std::atomic<int> a;
    alignas(std::hardware_destructive_interference_size) std::atomic<int> b;
};

// ✅ FIX 3 — padding between fields
struct Counters {
    int a;
    char pad[60];  // fill to 64 bytes
    int b;
};
```

---

## Key Rules

| Rule | Explanation |
|------|-------------|
| False sharing ≠ data race | Different variables, same cache line. Logically correct but slow. |
| atomic does not eliminate false sharing | atomic prevents data race but the cache line still ping-pongs. |
| Only writes cause false sharing | Read-read sharing is not a problem. At least one write must exist. |
| Cache line = 64 bytes on x86 | `std::hardware_destructive_interference_size` gives this at compile time. |
| `alignas(64)` is the most reliable fix | Forces each variable onto its own cache line. |
