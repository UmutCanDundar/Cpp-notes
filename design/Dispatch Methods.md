# All Dispatch Methods — Comparison Table

## Main Comparison

| Method | Dispatch Time | Code Size | i-cache | Latency | Debug/Profiling | When to Use |
|--------|--------------|-----------|---------|---------|-----------------|-------------|
| `switch` / `if-else` | Runtime | Small | Bad (branch tree) | Medium | Easy | Few cases, low traffic |
| Function Pointer Table | Runtime | Small | Good | Low | Very good | **Your situation (HFT)** |
| Template Dispatch | Compile-time | Large | Bad | Low | Hard | T known at compile-time |
| `constexpr switch` | Compile-time | Small | Very good | Very low | Medium | Constant/fixed input |
| Virtual (vtable) | Runtime | Medium | Medium | Medium | Good | OOP, wide API |
| CRTP | Compile-time | Large | Medium | Low | Hard | Static polymorphism |
| `std::variant` + `std::visit` | Compile-time* | Medium | Good | Low | Medium | Type-safe union, HFT |
| `std::function` | Runtime | Small | Bad | High | Easy | Flexibility, non-hot path |
| JIT / Codegen | Runtime | Dynamic | Risky | Lowest | Hard | Ultra-specialized |
| X-Macro | Compile-time | Small | Good | Low | Medium | Enum + handler sync |
| Tag Dispatch | Compile-time | Medium | Good | Low | Medium | Template overload selection |
| `if constexpr` | Compile-time | Medium | Good | Zero | Easy | Single-template branching |

> *`std::visit` resolves at compile-time via a vtable-like jump table, but the type is determined at runtime.

---

## Method Details

### 1. `switch` / `if-else`
```cpp
void handle(MsgType type, uint8_t* buf) {
    switch(type) {
        case NEW_ORDER: handle_new(buf); break;
        case CANCEL:    handle_cancel(buf); break;
        // 20 more cases...
    }
}
```
- ✅ Simple, readable
- ❌ Compiler generates branch tree → i-cache unfriendly with many cases
- ❌ Adding new case requires modifying the switch

---

### 2. Function Pointer Table (Recommended for HFT)
```cpp
using Handler = void(*)(uint8_t*);
static const Handler dispatch[] = {
    handle_new,     // index 0
    handle_cancel,  // index 1
    handle_ack,     // index 2
};
void handle(MsgType type, uint8_t* buf) {
    dispatch[static_cast<int>(type)](buf);  // single indirect call
}
```
- ✅ O(1) dispatch — array index
- ✅ No branch, branch predictor not needed
- ✅ i-cache friendly — compact jump table
- ✅ Easy to profile (function addresses visible)
- ❌ No type safety — raw function pointers

---

### 3. Template Dispatch
```cpp
template<MsgType T>
void handle(uint8_t* buf);

template<> void handle<NEW_ORDER>(uint8_t* buf) { /* ... */ }
template<> void handle<CANCEL>(uint8_t* buf)    { /* ... */ }
```
- ✅ Zero overhead — fully inlined at compile-time
- ✅ Type safe
- ❌ T must be known at compile-time — not suitable when message type arrives at runtime
- ❌ Code bloat — separate binary per specialization → i-cache pressure

---

### 4. `constexpr switch`
```cpp
template<int N>
constexpr auto make_handler() {
    if constexpr (N == 0) return handle_new;
    else if constexpr (N == 1) return handle_cancel;
}
```
- ✅ Evaluated at compile-time
- ✅ No runtime overhead
- ❌ Input must be a compile-time constant

---

### 5. Virtual (vtable)
```cpp
struct Handler {
    virtual void handle(uint8_t* buf) = 0;
};
struct NewOrderHandler : Handler {
    void handle(uint8_t* buf) override { /* ... */ }
};
```
- ✅ Easy to extend — add new handler without modifying existing code
- ✅ Good for profiling/debugging
- ❌ vtable indirect call + possible i-cache miss
- ❌ Each object carries a vtable pointer (8 bytes overhead)
- ❌ Prevent inlining

---

### 6. CRTP (Curiously Recurring Template Pattern)
```cpp
template<class Derived>
struct HandlerBase {
    void handle(uint8_t* buf) {
        static_cast<Derived*>(this)->handle_impl(buf);  // no vtable
    }
};
struct NewOrderHandler : HandlerBase<NewOrderHandler> {
    void handle_impl(uint8_t* buf) { /* ... */ }
};
```
- ✅ Zero overhead — inlined, no vtable
- ✅ Type safe
- ❌ Type must be known at compile-time
- ❌ Code bloat per specialization

---

### 7. `std::variant` + `std::visit`
```cpp
using Message = std::variant<NewOrder, Cancel, Ack>;
std::visit([](auto& msg) { handle(msg); }, message);
```
- ✅ Type safe — no raw union
- ✅ No heap allocation
- ✅ Compiler generates a jump table — no branch tree
- ✅ HFT friendly — your project already uses this pattern
- ❌ All types must be known at compile-time
- ❌ visit has slight overhead vs raw function pointer table

---

### 8. X-Macro
```cpp
#define MESSAGE_TYPES \
    X(NEW_ORDER, handle_new) \
    X(CANCEL, handle_cancel) \
    X(ACK, handle_ack)

// Generate enum
enum MsgType {
#define X(name, fn) name,
    MESSAGE_TYPES
#undef X
};

// Generate dispatch table
static Handler dispatch[] = {
#define X(name, fn) fn,
    MESSAGE_TYPES
#undef X
};
```
- ✅ Enum and handler table always in sync — no mismatch risk
- ✅ Adding a new type requires changing only one place
- ❌ Macro-heavy, hard to read

---

## Decision Guide

| Situation | Recommended Method |
|-----------|--------------------|
| Message type known at runtime, HFT hot path | **Function Pointer Table** |
| Type-safe union, type known at compile-time | **std::variant + std::visit** |
| OOP design, wide API | **Virtual** |
| Static polymorphism, zero overhead | **CRTP** |
| Type known at compile-time, single function | **Template Dispatch** or **if constexpr** |
| Few cases, simple code | **switch / if-else** |
| Enum and handlers must stay in sync | **X-Macro + Function Pointer Table** |

---

