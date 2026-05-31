# Creational — Object Creation Patterns

| Pattern | Structure | HFT | Description + Example |
|---------|-----------|-----|-----------------------|
| Factory Method | Base class + virtual create() | sometimes | Subclass decides which object to create. E.g. `OrderFactory::create(type)` → returns FIXOrder or OUCHOrder. Has virtual call — do not use on hot path. |
| Abstract Factory | Factory of factories | not used | Produce families of related objects. E.g. `BISTFactory` → BISTOrder + BISTSession + BISTParser. Complex, runtime polymorphism — too heavy for HFT. |
| Singleton | Static instance, private ctor | careful | Guarantees a single instance. Used for Logger, Config. Since C++11, static local variable is sufficient for thread-safe singleton. Makes testing harder due to global state. |
| Builder | Step-by-step construction | sometimes | Build a complex object step by step. E.g. `OrderBuilder().side(Buy).qty(100).price(50).build()`. Used for config or test data preparation. Not on hot path. |
| Prototype | clone() method | rarely | Create new object by copying an existing one. C++ copy constructor already does this. |
| Object Pool | Pre-allocated object pool | critical | Not in GoF but the most important HFT pattern. Take from pool instead of malloc/new, return when done. Your `orderrisk_pool_` is exactly this. Allocation latency = zero. |

```cpp
// Singleton — C++11 thread-safe
class Logger {
public:
    static Logger& instance() {
        static Logger inst;  // thread-safe, initialized once
        return inst;
    }
private:
    Logger() = default;
};
```
