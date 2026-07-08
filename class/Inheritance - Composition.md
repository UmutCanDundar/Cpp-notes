# Inheritance & Composition in C++

---

## 1. INHERITANCE BASICS

```cpp
class Base {
public:
    int pub = 1;
protected:
    int prot = 2;
private:
    int priv = 3; // never accessible from derived
};

class Derived : public Base {
    void f() {
        pub  = 10; // OK
        prot = 20; // OK
        priv = 30; // ERROR: private
    }
};
```

---

## 2. ACCESS SPECIFIERS IN INHERITANCE

| Member in Base | `public` inheritance | `protected` inheritance | `private` inheritance |
|----------------|----------------------|-------------------------|-----------------------|
| `public`       | `public`             | `protected`             | `private`             |
| `protected`    | `protected`          | `protected`             | `private`             |
| `private`      | inaccessible         | inaccessible            | inaccessible          |

```cpp
class A { public: int x; protected: int y; };

class B : public    A {}; // x→public,    y→protected
class C : protected A {}; // x→protected, y→protected
class D : private   A {}; // x→private,   y→private
```

> **Default:** `class` → `private`, `struct` → `public`

---

## 3. VIRTUAL FUNCTIONS & VTABLE

```cpp
class Shape {
public:
    virtual double area() const { return 0.0; }  // virtual: runtime dispatch
    virtual ~Shape() = default;                   // always virtual destructor
};

class Circle : public Shape {
    double r_;
public:
    Circle(double r) : r_(r) {}
    double area() const override { return 3.14 * r_ * r_; } // override: checked by compiler
};

Shape* s = new Circle(5.0);
s->area(); // calls Circle::area() — dynamic dispatch via vtable
delete s;  // calls Circle::~Circle() then Shape::~Shape()
```

**How vtable works:**
- Each class with virtual functions gets a `vtable` (array of function pointers)
- Each object has a hidden `vptr` pointing to its class's vtable
- `s->area()` → dereference `vptr` → look up `area` → call it
- Cost: one extra pointer indirection per virtual call

**`override` keyword:**
```cpp
double area() const override; // compiler error if signature doesn't match Base
```
Always use `override` — catches typos silently breaking dispatch.

**`final` keyword:**
```cpp
class Circle final : public Shape {}; // cannot be further derived
double area() const final override;   // cannot be overridden in subclasses
```

---

## 4. ABSTRACT CLASS & PURE VIRTUAL

```cpp
class Shape {
public:
    virtual double area() const = 0;  // pure virtual → Shape is abstract
    virtual ~Shape() = default;
};

// Shape s;          // ERROR: cannot instantiate abstract class
Shape* s = new Circle(5.0); // OK: pointer/reference to abstract is fine
```

- A class with at least one pure virtual function is **abstract**
- Derived class must override all pure virtuals, otherwise it's also abstract
- Pure virtual can have a body (rarely used, provides default impl)

```cpp
virtual void log() const = 0; // pure virtual with optional body
void Shape::log() const { std::puts("Shape"); } // definition in .cpp
```

---

## 5. CONSTRUCTOR & DESTRUCTOR ORDER

```cpp
class A {
public:
    A()  { std::puts("A ctor"); }
    ~A() { std::puts("A dtor"); }
};

class B : public A {
public:
    B()  { std::puts("B ctor"); }
    ~B() { std::puts("B dtor"); }
};

class C : public B {
public:
    C()  { std::puts("C ctor"); }
    ~C() { std::puts("C dtor"); }
};

C c;
// Construction:  A ctor → B ctor → C ctor  (base first)
// Destruction:   C dtor → B dtor → A dtor  (derived first)
```

---

## 6. CONSTRUCTOR DELEGATION & BASE INIT

```cpp
class Animal {
    std::string name_;
public:
    Animal(std::string name) : name_(std::move(name)) {}
};

class Dog : public Animal {
    int age_;
public:
    Dog(std::string name, int age)
        : Animal(std::move(name)) // base constructor must be called explicitly
        , age_(age)
    {}
};
```

**Using-declaration to inherit constructors:**
```cpp
class Dog : public Animal {
public:
    using Animal::Animal; // inherit all Animal constructors
};

Dog d("Rex"); // calls Animal(string)
```

---

## 7. SLICING

```cpp
Circle c(5.0);
Shape s = c;       // SLICING: Circle part is cut off, s is just a Shape
s.area();          // calls Shape::area() — not Circle::area()

Shape& ref = c;    // OK: no slicing, ref sees Circle through vtable
Shape* ptr = &c;   // OK: no slicing
```

> Always use pointer or reference for polymorphism. Avoid copying base types.

---

## 8. MULTIPLE INHERITANCE

```cpp
class Flyable  { public: virtual void fly()  {} };
class Swimmable{ public: virtual void swim() {} };

class Duck : public Flyable, public Swimmable {
public:
    void fly()  override { std::puts("Duck flies"); }
    void swim() override { std::puts("Duck swims"); }
};
```

**Name ambiguity:**
```cpp
class A { public: void f() {} };
class B { public: void f() {} };
class C : public A, public B {};

C c;
c.f();    // ERROR: ambiguous
c.A::f(); // OK: explicit scope
c.B::f(); // OK
```

---

## 9. DIAMOND PROBLEM & VIRTUAL INHERITANCE

```cpp
class Animal { public: int id = 0; };

class Dog    : public Animal {};
class Robot  : public Animal {};

// Without virtual: two copies of Animal
class RobotDog : public Dog, public Robot {
    // RobotDog::Dog::Animal::id
    // RobotDog::Robot::Animal::id  — ambiguous!
};

// With virtual: one shared Animal
class Dog2   : virtual public Animal {};
class Robot2 : virtual public Animal {};
class RobotDog2 : public Dog2, public Robot2 {
    // single Animal::id — no ambiguity
};
```

> Virtual inheritance adds overhead (extra pointer, complex construction order). Prefer composition over deep diamond hierarchies.

---

## 10. CRTP (Curiously Recurring Template Pattern)

Static polymorphism — dispatch resolved at compile time, zero virtual overhead.

```cpp
template<typename Derived>
class Shape {
public:
    double area() const {
        return static_cast<const Derived*>(this)->area_impl();
    }
};

class Circle : public Shape<Circle> {
    double r_;
public:
    Circle(double r) : r_(r) {}
    double area_impl() const { return 3.14 * r_ * r_; }
};

Circle c(5.0);
c.area(); // calls area_impl() directly — no vtable, no vptr
```

**CRTP vs virtual:**
| | Virtual | CRTP |
|---|---|---|
| Dispatch | Runtime (vtable) | Compile-time |
| Overhead | vptr + indirection | None |
| Flexibility | Heterogeneous collections | Same template instantiation |
| Use case | Plugin systems, type erasure | Performance-critical, mixins |

**CRTP mixin pattern:**
```cpp
template<typename Derived>
class Printable {
public:
    void print() const {
        static_cast<const Derived*>(this)->print_impl();
    }
};

template<typename Derived>
class Serializable {
public:
    std::string serialize() const {
        return static_cast<const Derived*>(this)->serialize_impl();
    }
};

class Order : public Printable<Order>, public Serializable<Order> {
public:
    void print_impl()            const { std::puts("Order"); }
    std::string serialize_impl() const { return "order_data"; }
};
```

---

## 11. NON-VIRTUAL INTERFACE (NVI) PATTERN

Public non-virtual calls private virtual — enforces pre/post conditions.

```cpp
class Logger {
public:
    void log(const std::string& msg) { // non-virtual, called by user
        pre_log();
        do_log(msg);  // virtual, overridden by derived
        post_log();
    }
private:
    virtual void do_log(const std::string& msg) = 0;
    virtual void pre_log()  {}
    virtual void post_log() {}
};

class FileLogger : public Logger {
    void do_log(const std::string& msg) override {
        // write to file
    }
};
```

---

## 12. COMPOSITION

**"Has-a" relationship** — prefer over inheritance for flexibility.

```cpp
// Bad: inheritance for reuse (Circle "is-a" Shape is OK, but...)
class Engine {};
class Car : public Engine {}; // wrong: Car "is-a" Engine? No.

// Good: composition
class Car {
    Engine engine_; // Car "has-a" Engine
public:
    void start() { engine_.start(); }
};
```

**Composition vs Inheritance:**
| | Inheritance | Composition |
|---|---|---|
| Relationship | "is-a" | "has-a" |
| Coupling | Tight (base changes affect derived) | Loose |
| Flexibility | Fixed at compile time | Swappable at runtime |
| Encapsulation | Exposes base internals (protected) | Fully encapsulated |
| Test | Harder to mock base | Easy to inject mock |

---

## 13. COMPOSITION WITH DEPENDENCY INJECTION

```cpp
class ILogger {
public:
    virtual void log(const std::string&) = 0;
    virtual ~ILogger() = default;
};

class ConsoleLogger : public ILogger {
public:
    void log(const std::string& msg) override { std::puts(msg.c_str()); }
};

class OrderManager {
    ILogger& logger_; // injected — not owned
public:
    explicit OrderManager(ILogger& logger) : logger_(logger) {}
    void place_order() { logger_.log("Order placed"); }
};

ConsoleLogger cl;
OrderManager om(cl); // inject dependency
```

---

## 14. PRIVATE/PROTECTED INHERITANCE AS COMPOSITION

```cpp
// "implemented-in-terms-of" — not "is-a"
class Stack : private std::vector<int> {
public:
    void push(int x) { push_back(x); }
    void pop()       { pop_back(); }
    int  top()       { return back(); }
};
// Stack s; s.push_back(1); // ERROR: push_back is private
```

> Prefer composition (member) over private inheritance — cleaner and less surprising.

---

## 15. QUICK REFERENCE

```
Inheritance types:
  public    → is-a relationship, polymorphism
  protected → rarely used
  private   → implemented-in-terms-of (prefer composition instead)

Keywords:
  virtual          → runtime dispatch via vtable
  override         → verify signature matches base
  final            → prevent further override/derivation
  = 0              → pure virtual, makes class abstract
  virtual (base)   → virtual inheritance, solves diamond

Patterns:
  CRTP    → static polymorphism, zero overhead, mixins
  NVI     → enforce invariants around virtual calls
  DI      → inject dependencies via interface, improves testability

Rules:
  - Always virtual destructor in base class
  - Prefer override over implicit override
  - Prefer composition over inheritance
  - Avoid deep inheritance hierarchies
  - Use pointer/reference for polymorphism, never slice
```
