# Structural — Class Composition Patterns

| Pattern | Structure | HFT | Description + Example |
|---------|-----------|-----|-----------------------|
| Adapter | Wrapper class | sometimes | Makes incompatible interfaces compatible. E.g. an adapter converting an external library's Order struct to your own Order. Used for protocol conversion. |
| Bridge | Abstraction + Implementation separate | rarely | Change interface and implementation independently. Complex, rarely seen in HFT. |
| Composite | Tree structure, same interface | rarely | Use single objects and groups uniformly. Common in UI trees, rare in HFT. |
| Decorator | Wrapper + same interface | sometimes | Add behavior to an object at runtime. E.g. LoggingOrder, TimestampedOrder. Has virtual call — only outside hot path. |
| Facade | Simple interface to complex system | common | Hide subsystems, expose a single entry point. Your `TradingEngine` class is a facade — it has parser, risk, order manager inside but exposes a single interface. |
| Flyweight | Shared immutable state | critical | Keep a single copy of repeated data. E.g. symbol metadata (ISIN, lot size) is not copied per order — accessed via shared pointer or index. Saves memory and cache. |
| Proxy | Same interface, different implementation | sometimes | For lazy loading, access control, logging. A mock object in tests is a proxy. |
| CRTP | `template<class D> struct Base` | critical | Not in GoF but replaces virtual in HFT. Compile-time polymorphism. Zero-cost abstraction. No virtual call, gets inlined. |

```cpp
// CRTP — compile-time polymorphism instead of virtual
template<class Derived>
struct Parser {
    void parse(uint8_t* buf) {
        static_cast<Derived*>(this)->parse_impl(buf);  // no virtual, inlined
    }
};
struct FIXParser : Parser<FIXParser> {
    void parse_impl(uint8_t* buf) { /* ... */ }
};
```
