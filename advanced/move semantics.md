
# Move Semantics

Move semantics let you transfer resources from one object to another instead of copying them. This avoids expensive deep copies. It is a core C++11 feature that makes containers, strings, and smart pointers fast.

## Basic Features of Move Semantics

Moving "steals" the internal resources (like a heap buffer) from a source object. The source is left valid but unspecified. This is much cheaper than copying large data.

```cpp
std::vector<int> a(1000000, 1);
std::vector<int> b = std::move(a); // steals a's buffer, no copy
```

## std::move

`std::move` does not move anything itself. It just casts its argument to an rvalue reference, signaling "this may be moved from."

```cpp
std::string s1 = "hello";
std::string s2 = std::move(s1); // s1 is now unspecified but valid
```

## Move Semantics & Special Member Functions

The compiler can auto-generate a move constructor and move assignment operator. These are used when the source is an rvalue.

```cpp
class Buffer {
public:
    Buffer(Buffer&& other) noexcept
        : data(other.data), size(other.size) {
        other.data = nullptr;
        other.size = 0;
    }
private:
    int* data;
    size_t size;
};
```

## How to Use Move Semantics

Pass by value and move internally, or overload with rvalue reference parameters, to enable efficient transfer of ownership.

```cpp
void setName(std::string name) { // takes by value
    this->name = std::move(name); // moves into member
}
```

## Reference Qualifiers & Move Semantics

Reference qualifiers (`&` and `&&`) let you overload member functions based on whether `*this` is an lvalue or rvalue. This enables optimizations when calling on temporaries.

```cpp
class Widget {
public:
    std::string getName() const & { return name; }       // copy
    std::string getName() && { return std::move(name); } // move
private:
    std::string name;
};
```

## Moved-From State

After a move, the source object is in a valid but unspecified state. You should not assume its value, only that it can be destroyed or reassigned.

```cpp
std::vector<int> v1{1,2,3};
std::vector<int> v2 = std::move(v1);
v1.clear(); // safe: v1 is valid, just unspecified
```

## Move Semantics & Exception Guarantees

Move operations should be marked `noexcept` when possible. STL containers use this to decide whether to move or copy elements during reallocation, for strong exception safety.

```cpp
class MyType {
public:
    MyType(MyType&&) noexcept = default; // enables vector optimizations
};
```

## Move Only Types

Some types (like `std::unique_ptr`, `std::thread`) can only be moved, not copied. This models exclusive ownership.

```cpp
std::unique_ptr<int> p1 = std::make_unique<int>(5);
std::unique_ptr<int> p2 = std::move(p1); // p1 is now nullptr
```

## Copy Elision

Copy elision means the compiler skips creating a temporary copy entirely, constructing the object directly in its final location. This is faster than even a move.

### Temporary Materialization

A prvalue only becomes an actual object (materializes) when it's needed, e.g. bound to a reference.

```cpp
const std::string& r = std::string("temp"); // materializes a temporary
```

### Mandatory Copy Elision

Since C++17, certain cases (like returning a prvalue) *must* elide the copy/move by the standard, not just as an optimization.

```cpp
std::string make() { return std::string("hi"); } // no copy/move, guaranteed
std::string s = make();
```

### Prvalue to Xvalue Conversion

A prvalue converts to an xvalue when it needs to be treated as an expiring object, such as during a move.

```cpp
std::string f();
std::string s = std::move(f()); // f() prvalue -> xvalue via std::move
```

### Unmaterialized Object Passing

C++17 allows passing prvalues directly to initialize objects without ever creating a temporary.

```cpp
struct Big { Big(int); };
void take(Big b);
take(Big(42)); // Big(42) constructed directly as the parameter
```

### Return Value Optimization (RVO)

RVO avoids copying a locally constructed temporary when it's returned directly.

```cpp
std::vector<int> create() {
    return std::vector<int>{1,2,3}; // constructed directly at call site
}
```

### Named Return Value Optimization (NRVO)

NRVO is an optional optimization that elides the copy for a *named* local variable returned from a function.

```cpp
std::vector<int> create() {
    std::vector<int> v{1,2,3};
    return v; // may be elided (not guaranteed)
}
```

### Throwing by Value

Exceptions are thrown by value. Copy elision often applies here too, avoiding unnecessary copies of the exception object.

```cpp
throw std::runtime_error("failure"); // temporary, elision may apply
```

### Catching by Value

Catching by value copies the exception object; catching by `const&` avoids that copy and is generally preferred.

```cpp
try { risky(); }
catch (const std::runtime_error& e) { /* no copy */ }
```

### Scenarios Blocking Copy Elision

Some situations prevent elision: returning a parameter by value, returning different named variables on different branches, or returning by `std::move` on a local (which can hurt NRVO).

```cpp
std::string f(std::string s) { return s; } // parameter, elision less likely
```

## Typical Mistakes & Misconceptions

A common mistake is calling `std::move` on a `const` object (it just becomes a const rvalue reference, so no actual move happens). Another is using an object after moving from it and assuming its old value.

```cpp
const std::string s = "hi";
std::string s2 = std::move(s); // still copies, because s is const
```

## Move Semantics in STL

STL containers use move constructors when growing, and algorithms like `std::move` (the algorithm) move ranges of elements efficiently.

```cpp
std::vector<std::string> v;
v.push_back(std::string("temp")); // moves the temporary in
```

## STL Moving Algorithms

`std::move` (algorithm header) moves a range of elements into another range.

```cpp
std::vector<std::string> src = {"a","b","c"};
std::vector<std::string> dst(3);
std::move(src.begin(), src.end(), dst.begin());
```

## Typical Mistakes & Misconceptions (STL context)

Moving from a container you still need afterward, or moving in a loop where elements alias each other, can cause bugs. Always ensure the moved-from container isn't relied upon for its values.

## Guidelines

- Mark move constructors/assignment `noexcept` when possible.
- Don't use an object's value after moving from it.
- Prefer passing by value + move for sink parameters.
- Let the compiler generate move operations when you can (Rule of Zero).
