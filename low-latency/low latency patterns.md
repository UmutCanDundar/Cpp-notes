# Low-Latency Pattern Usage Guide

| Pattern | On Hot Path | Why / Alternative |
|---------|-------------|-------------------|
| Virtual function (polymorphism) | do not use | vtable lookup + indirect call = branch predictor miss. Replace with CRTP or templates. |
| CRTP | prefer | Compile-time polymorphism, zero overhead, gets inlined. |
| Object Pool | critical | Instead of malloc/new. Allocation latency = 0. Present in your project. |
| Flyweight | critical | Symbol metadata, lookup table — single copy, access by index. |
| std::variant + visit | prefer | Instead of virtual. Type-safe union, compile-time dispatch. |
| Singleton | careful | Fine for Config/Logger. On hot path, global access = cache miss risk. |
| Observer (dynamic) | not on hot path | Callback list + virtual = latency. Use on config path. |
| Strategy (template) | policy-based | Pass as template parameter, zero-cost. |
| State machine | enum + array | Not state objects — use enum index + transition table array. Lookup instead of branch. |
| Lock-free ring buffer | critical | Not in GoF. The most fundamental HFT structure. Core of your pipeline. |

> **Rule:** inheritance + virtual = runtime polymorphism = vtable = latency. In HFT, replace with template + CRTP + std::variant.
