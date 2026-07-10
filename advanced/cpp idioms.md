# C++ Idioms and Techniques

Idioms are proven patterns for solving recurring design problems in C++. Knowing them helps you write safer, more efficient, and more maintainable code, and recognize them in others' code.

## ADL + Fallback

Argument-Dependent Lookup (ADL) finds free functions in the namespace of their arguments; a fallback (like `using std::swap;`) ensures a default is available if no ADL match exists.

```cpp
using std::swap;
swap(a, b); // finds custom swap via ADL, or falls back to std::swap
```

## Attorney-Client

Restricts access to a class's private members to only specific "friend" classes, via an intermediate "Attorney" class, instead of granting full friendship.

```cpp
class Attorney {
    static void call(Client& c) { c.privateMethod(); }
    friend class Trusted;
};
```

## Biased Distribution (std::discrete_distribution)

Generates random numbers according to custom weighted probabilities, useful for simulations, loot tables, or weighted sampling.

```cpp
std::discrete_distribution<int> dist({10, 30, 60}); // weighted choices
```

## Clamp

Restricts a value to a given range, avoiding manual min/max chains.

```cpp
int v = std::clamp(150, 0, 100); // 100
```

## Construction Tracker

A debugging idiom where a class counts/logs its constructions and destructions to detect leaks or unexpected copies.

```cpp
struct Tracker {
    Tracker() { ++count; }
    ~Tracker() { --count; }
    static inline int count = 0;
};
```

## Copy & Swap Idiom

Implements assignment operators safely and exception-safely by constructing a temporary copy and swapping it with `*this`.

```cpp
Widget& operator=(Widget other) { // pass by value (copy)
    swap(*this, other);
    return *this;
}
```

## Container Idioms

Patterns for building custom containers: providing `begin()/end()`, value semantics, and following STL conventions so they work with range-based for and algorithms.

## Exception Dispatcher

See Exception Handling — catching a base exception type and routing to different handling logic based on the concrete type.

## Factories

Functions or classes dedicated to creating objects, hiding construction complexity and enabling polymorphic creation.

```cpp
std::unique_ptr<Shape> createShape(const std::string& type) {
    if (type == "circle") return std::make_unique<Circle>();
    return std::make_unique<Square>();
}
```

## Gather Algorithm

Rearranges elements in a range so that all elements satisfying a predicate are moved together around a pivot point.

```cpp
// std::stable_partition twice implements "gather" around a point
```

## Guarded Suspension

A concurrency idiom where a thread waits (blocks) until a condition becomes true before proceeding, typically via a condition variable.

```cpp
std::unique_lock<std::mutex> lock(m);
cv.wait(lock, [] { return ready; }); // guarded suspension
```

## Hidden Friends

Defining `friend` functions (like `operator==`) inside the class body so they're only found via ADL, reducing overload resolution ambiguity and compile time.

```cpp
class Point {
    friend bool operator==(const Point& a, const Point& b) { return a.x==b.x; }
};
```

## IIFE (Immediately Invoked Lambda Expression)

Calling a lambda right after defining it to compute a value once, keeping complex initialization logic local and `const`-friendly.

```cpp
const int x = []{ return 42; }();
```

## Implementation Class

Separates a class's interface from its implementation details by delegating to an internal "impl" class (related to Pimpl).

## Local Buffer Optimization

Small objects (like short strings) store data inline instead of on the heap, avoiding allocation for common small cases (e.g., Small String Optimization).

```cpp
std::string s = "short"; // likely stored inline, no heap allocation
```

## Memory Ownership

Refers to clearly designing who is responsible for freeing a resource, expressed via smart pointers (`unique_ptr` = exclusive, `shared_ptr` = shared).

```cpp
std::unique_ptr<Widget> owner = std::make_unique<Widget>(); // clear ownership
```

## Mixin Classes

Small reusable classes combined via inheritance (often CRTP) to add functionality to other classes without a rigid hierarchy.

```cpp
template <typename Derived>
struct Printable { void print() { std::cout << static_cast<Derived*>(this)->toString(); } };
```

## Named Parameter

Simulates named/keyword arguments in C++ (which lacks native support) using builder-style chained setter calls.

```cpp
Config c = ConfigBuilder().setWidth(100).setHeight(50).build();
```

## Nifty Counter

Ensures a global object used across translation units is initialized before first use and destroyed only once, solving the "static initialization order fiasco."

```cpp
// Header declares a counter; each including TU increments it in a static initializer
```

## Non-Virtual Interface (NVI)

Public non-virtual functions call private/protected virtual functions, giving the base class control over pre/post logic around customizable behavior.

```cpp
class Base {
public:
    void run() { setup(); doWork(); } // non-virtual interface
private:
    virtual void doWork() = 0;
    void setup() {}
};
```

## Pimpl & Fast Pimpl

"Pointer to implementation" hides private members behind an opaque pointer, reducing compile-time dependencies and stabilizing ABI.

```cpp
class Widget {
public:
    Widget();
    ~Widget();
private:
    class Impl;
    std::unique_ptr<Impl> pImpl;
};
```

## Positive Lambda

Prefixing a lambda with unary `+` converts it to a plain function pointer (works only for captureless lambdas), useful for C APIs expecting function pointers.

```cpp
auto fp = +[](int x) { return x * 2; }; // function pointer
```

## propagate_const

A wrapper (`std::experimental::propagate_const`) that makes const-ness of a pointer member propagate to the pointee, so a `const` method can't modify pointed-to data through a raw/smart pointer member.

## Proxy

An object that stands in for another object, controlling access to it — used for lazy evaluation, reference counting, or access control (e.g., `std::vector<bool>`'s proxy reference).

```cpp
std::vector<bool> v(5);
auto proxy = v[0]; // proxy object, not a real bool&
```

## RAII

"Resource Acquisition Is Initialization" ties a resource's lifetime to an object's lifetime, so destructors automatically release resources, preventing leaks.

```cpp
std::lock_guard<std::mutex> lock(m); // mutex released automatically
```

## Scope Guards

An RAII object that runs arbitrary cleanup code when it goes out of scope, useful for ad-hoc cleanup without writing a full RAII class.

```cpp
auto guard = std::experimental::scope_exit([]{ cleanup(); });
```

## Reference Counting

Tracks how many owners reference a resource, freeing it when the count hits zero — the mechanism behind `std::shared_ptr`.

```cpp
std::shared_ptr<int> p1 = std::make_shared<int>(5);
std::shared_ptr<int> p2 = p1; // ref count = 2
```

## Return Type Resolver

A technique where a proxy return type is implicitly convertible to multiple different types, letting a function's return "type" depend on the calling context.

```cpp
struct AnyReturn {
    operator int() { return 1; }
    operator std::string() { return "one"; }
};
```

## Scope Guard

(See Scope Guards above.) A single-use idiom for guaranteed cleanup at scope exit, often with commit/dismiss capability.

## Slide Algorithm

Moves a range of elements to a new position within a sequence, shifting other elements accordingly (like PowerPoint's "move slide").

```cpp
std::rotate(v.begin() + from, v.begin() + from + 1, v.begin() + to);
```

## Strong Types

Wraps a primitive type (like `int`) in a distinct class to prevent mixing up semantically different values of the same underlying type (e.g., `Meters` vs `Seconds`).

```cpp
struct Meters { explicit Meters(double v) : value(v) {} double value; };
```

## Swap Functions

A custom, efficient `swap` (usually `noexcept`) enables the copy-and-swap idiom and STL algorithm optimizations.

```cpp
friend void swap(Widget& a, Widget& b) noexcept { /* swap members */ }
```

## Tag Dispatch

(See Generic Programming.) Uses empty tag types to select overloads at compile time based on a type trait.

## Threadsafe Interface

Designing a class's public interface so that its methods are safe to call concurrently, often via internal locking, distinct from just making the implementation thread-safe.

## Type Erasure

Hides a concrete type behind a uniform interface, allowing heterogeneous objects to be treated polymorphically without a common base class (e.g., `std::function`, `std::any`).

```cpp
std::function<int(int)> f = [](int x) { return x * 2; }; // erases the lambda's type
```

## Variant Overloader

Combines multiple lambdas into one callable object (via multiple inheritance of `operator()`) for use with `std::visit` on a `std::variant`.

```cpp
template <typename... Ts> struct overload : Ts... { using Ts::operator()...; };
template <typename... Ts> overload(Ts...) -> overload<Ts...>;

std::visit(overload{
    [](int i) { std::cout << "int " << i; },
    [](std::string s) { std::cout << "string " << s; }
}, myVariant);
```

## Virtual Constructor

Simulates a "virtual constructor" via a virtual `clone()` method, allowing polymorphic copying of objects through a base pointer.

```cpp
class Shape {
public:
    virtual std::unique_ptr<Shape> clone() const = 0;
};
```

## Virtual Friend

Combines a non-virtual friend function with a virtual member function it delegates to, letting free functions (like `operator<<`) behave polymorphically.

```cpp
class Shape {
public:
    friend std::ostream& operator<<(std::ostream& os, const Shape& s) { return s.print(os); }
private:
    virtual std::ostream& print(std::ostream& os) const = 0;
};
```
