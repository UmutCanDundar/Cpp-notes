# Concrete Before/After Optimization Examples

```cpp
// BAD — allocation inside loop
for (auto& msg : messages) {
    std::string s = msg.to_string();  // heap alloc on every iteration
    process(s);
}

// GOOD — allocate once, reuse
std::string s;
s.reserve(256);
for (auto& msg : messages) {
    s.clear();
    msg.append_to(s);  // no allocation
    process(s);
}
```

---

```cpp
// BAD — linear search every time
for (auto& order : orders) {
    auto it = std::find(symbols.begin(), symbols.end(), order.symbol);
    if (it != symbols.end()) { /* ... */ }
}

// GOOD — O(1) with hash map
std::unordered_map<std::string, SymbolInfo> symbol_map;
for (auto& order : orders) {
    auto it = symbol_map.find(order.symbol);  // O(1)
    if (it != symbol_map.end()) { /* ... */ }
}
```

---

```cpp
// BAD — switch on every message
void handle(MsgType type, uint8_t* buf) {
    switch(type) {
        case NEW_ORDER:    handle_new(buf);    break;
        case CANCEL:       handle_cancel(buf); break;
        case ACK:          handle_ack(buf);    break;
    }
}

// GOOD — dispatch table, no branch
using Handler = void(*)(uint8_t*);
Handler dispatch[] = {handle_new, handle_cancel, handle_ack};
void handle(MsgType type, uint8_t* buf) {
    dispatch[static_cast<int>(type)](buf);
}
```

---

```cpp
// BAD — AoS, must enter each struct to read x values
struct Point { float x, y, z; };
Point points[1000];

// GOOD — SoA, x values are contiguous, SIMD-friendly
struct Points {
    float x[1000];
    float y[1000];
    float z[1000];
};
```
