# `<concepts>` and `requires` — C++20 Concepts

A **concept** is a named, compile-time `bool` condition on template arguments. It replaces most `enable_if` / SFINAE tricks, gives readable error messages, and lets the compiler pick the *most constrained* overload. This file covers the standard concepts (`std::`), how to define your own, and every place a constraint can be written.

### Notation
* `T`, `U`, `Ts...`: types and type packs. `Args...`: argument types. `F`: a callable type. `R`: a relation (callable taking two arguments).
* `C<...>`: any concept. `expr`: any expression. `a`, `b`, `t`: local parameter names inside a `requires` expression (no real objects exist).
* `= default`: shown after a parameter with a default, e.g. `three_way_comparable<T, Cat = partial_ordering>`.
* **Verified**: every `→` result and every "compiles / error" statement was produced with g++ 13.3 (`-std=c++23`). Items marked "not compiled" come from the standard only.
* All concepts evaluate at compile time: no run-time cost.

---

## 1. Concept or trait?

| Question | Trait (`<type_traits>`) | Concept (`<concepts>`) |
|---|---|---|
| What is it | a class / variable template: `std::is_integral_v<T>` | a named constraint: `std::integral<T>` |
| Where it works | anywhere a `bool` constant is needed | the same, **and** directly in template headers and `auto` declarations |
| Overload ranking | none (both candidates are equal, ambiguous) | **more constrained wins** (subsumption, section 7) |
| Error message | "no matching function / no type named `type`" | "constraints not satisfied" with the failing concept |
| Typical use | transformations (`remove_const_t`) and low-level queries | requirements on interfaces |

Concepts are usually written **on top of** traits and `requires` expressions. Most standard concepts also check slightly more than their trait cousin (see the Note column below).

---

## 2. Standard concepts (`<concepts>`)

### 2.1 Core language concepts


| Returns | Concept | Usage | Note |
|---|---|---|---|
| `bool` | same_as | `same_as<T, U>`<br>`same_as<int, int>` → `true`<br>`same_as<int, const int>` → `false` | Trait: `is_same_v<T, U>`. Symmetric: `same_as<T,U>` implies `same_as<U,T>`. |
| `bool` | derived_from | `derived_from<Derived, Base>`<br>`derived_from<Derived, Base>` → `true`<br>`derived_from<Base, Derived>` → `false`<br>`derived_from<Priv, Base>` → `false`<br>`derived_from<int, int>` → `false` | Trait: `is_base_of_v<Base, Derived>` (**argument order swapped**). Also needs an accessible, unambiguous base: a private base fails here but passes the trait. Never true for non-class types. |
| `bool` | convertible_to | `convertible_to<From, To>`<br>`convertible_to<int, double>` → `true`<br>`convertible_to<int, Expl>` → `false`<br>`convertible_to<const char*, string>` → `true`<br>`convertible_to<string, const char*>` → `false` | Trait: `is_convertible_v<From, To>`. Needs the implicit **and** the explicit (`static_cast`) conversion to work. |
| `bool` | common_reference_with | `common_reference_with<T, U>`<br>`common_reference_with<int&, const int&>` → `true` | Both types convert to `common_reference_t<T, U>`. Used when comparing or assigning across related types. |
| `bool` | common_with | `common_with<T, U>`<br>`common_with<int, double>` → `true`<br>`common_with<int, string>` → `false` | Both convert to `common_type_t<T, U>` (and their common reference exists). |
| `bool` | integral / signed_integral / unsigned_integral | `integral<T>`<br>`signed_integral<T>`<br>`unsigned_integral<T>`<br>`integral<bool>` → `true`<br>`integral<char>` → `true`<br>`integral<float>` → `false`<br>`signed_integral<int>` → `true`<br>`signed_integral<unsigned>` → `false`<br>`signed_integral<bool>` → `false`<br>`unsigned_integral<bool>` → `true`<br>`unsigned_integral<unsigned>` → `true` | Traits: `is_integral_v` (+ `is_signed_v` / `is_unsigned_v`). `bool` and the `char` types count as integral; `bool` is unsigned. |
| `bool` | floating_point | `floating_point<T>`<br>`floating_point<double>` → `true`<br>`floating_point<int>` → `false` | Trait: `is_floating_point_v<T>`. |
| `bool` | assignable_from | `assignable_from<To, From>`<br>`assignable_from<int&, int>` → `true`<br>`assignable_from<int, int>` → `false`<br>`assignable_from<const int&, int>` → `false`<br>`assignable_from<string&, const char*>` → `true` | Trait: `is_assignable_v<To, From>`. `To` must be an **lvalue reference**, and the assignment must return `To`. |
| `bool` | swappable / swappable_with | `swappable<T>`<br>`swappable_with<T, U>`<br>`swappable<int>` → `true`<br>`swappable_with<int&, long&>` → `false`<br>`swappable<unique_ptr<int>>` → `true` | Trait: `is_swappable_v`. Checks `ranges::swap` (the customization point used by the ranges algorithms). |
| `bool` | destructible | `destructible<T>`<br>`destructible<int>` → `true`<br>`destructible<PrivDtor>` → `false` | Trait: `is_destructible_v`, but stricter: the destructor must be `noexcept`. A type with `~Th() noexcept(false)` is `is_destructible_v` but **not** `destructible` (verified). |
| `bool` | constructible_from | `constructible_from<T, Args...>`<br>`constructible_from<string, const char*>` → `true`<br>`constructible_from<Expl, int>` → `true`<br>`constructible_from<NoDef>` → `false`<br>`constructible_from<unique_ptr<int>, unique_ptr<int>&>` → `false` | Trait: `is_constructible_v<T, Args...>`, plus `T` must be `destructible`. Explicit constructors count (compare with `convertible_to`). |
| `bool` | default_initializable | `default_initializable<T>`<br>`default_initializable<int>` → `true`<br>`default_initializable<NoDef>` → `false`<br>`default_initializable<const int>` → `false` | Needs `T{}`, `T()` **and** `T t;` to be valid. `const int` fails because `const int t;` is ill-formed, while `is_default_constructible_v<const int>` is `true` (verified). |
| `bool` | move_constructible / copy_constructible | `move_constructible<T>`<br>`copy_constructible<T>`<br>`move_constructible<unique_ptr<int>>` → `true`<br>`copy_constructible<unique_ptr<int>>` → `false`<br>`copy_constructible<string>` → `true` | Traits: `is_move_constructible_v` / `is_copy_constructible_v`, plus destructible and convertible from `T`, `const T&`, ... as appropriate. |


### 2.2 Comparison concepts


| Returns | Concept | Usage | Note |
|---|---|---|---|
| `bool` | equality_comparable / equality_comparable_with | `equality_comparable<T>`<br>`equality_comparable_with<T, U>`<br>`equality_comparable<int>` → `true`<br>`equality_comparable<S>` → `false`<br>`equality_comparable_with<int, double>` → `true`<br>`equality_comparable_with<string, const char*>` → `true`<br>`equality_comparable_with<int, string>` → `false` | `==` and `!=` are valid and give a boolean-like result. `_with` also needs a common reference type. |
| `bool` | totally_ordered / totally_ordered_with | `totally_ordered<T>`<br>`totally_ordered_with<T, U>`<br>`totally_ordered<string>` → `true`<br>`totally_ordered<S>` → `false`<br>`totally_ordered_with<int, double>` → `true` | `<`, `>`, `<=`, `>=` on top of `equality_comparable`. |
| `bool` | three_way_comparable / three_way_comparable_with (`<compare>`) | `three_way_comparable<T, Cat = partial_ordering>`<br>`three_way_comparable_with<T, U, Cat = partial_ordering>`<br>`three_way_comparable<int>` → `true`<br>`three_way_comparable<S>` → `false` | `<=>` is valid and its result category is at least `Cat` (`strong_ordering` > `weak_ordering` > `partial_ordering`). |


> These concepts check **syntax only**. `equality_comparable` cannot verify that your `==` is a real equivalence relation; that part is a documented *semantic* requirement.

### 2.3 Object concepts (each one adds to the previous)

| Returns | Concept | Usage | Note |
|---|---|---|---|

| `bool` | movable | `movable<T>`<br>`movable<unique_ptr<int>>` → `true`<br>`movable<const int>` → `false` | `T` is an object type, `move_constructible`, `assignable_from<T&, T>` and `swappable`. |
| `bool` | copyable | `copyable<T>`<br>`copyable<unique_ptr<int>>` → `false`<br>`copyable<string>` → `true` | `movable` + `copy_constructible` + copy-assignable. Move-only types such as `unique_ptr` stop here. |
| `bool` | semiregular | `semiregular<T>`<br>`semiregular<string>` → `true`<br>`semiregular<NoDef>` → `false` | `copyable` + `default_initializable`. Behaves like a plain value that can be default-created. |
| `bool` | regular | `regular<T>`<br>`regular<string>` → `true`<br>`regular<unique_ptr<int>>` → `false` | `semiregular` + `equality_comparable`. The "works like `int`" ideal. |


Hierarchy: `regular` ⊃ `semiregular` ⊃ `copyable` ⊃ `movable`.

### 2.4 Callable concepts


| Returns | Concept | Usage | Note |
|---|---|---|---|
| `bool` | invocable / regular_invocable | `invocable<F, Args...>`<br>`regular_invocable<F, Args...>`<br>`invocable<void(*)(int), int>` → `true`<br>`invocable<void(*)(int), string>` → `false` | Trait: `is_invocable_v<F, Args...>` (calls through `std::invoke`, so member pointers work). `regular_invocable` additionally promises the call does not change its inputs (not checkable by the compiler). |
| `bool` | predicate | `predicate<F, Args...>`<br>`predicate<bool(*)(int), int>` → `true`<br>`predicate<void(*)(int), int>` → `false` | `regular_invocable` and the result converts to `bool`. |
| `bool` | relation / equivalence_relation / strict_weak_order | `relation<R, T, U>`<br>`equivalence_relation<R, T, U>`<br>`strict_weak_order<R, T, U>`<br>`relation<less<>, int, int>` → `true`<br>`equivalence_relation<equal_to<>, int, int>` → `true`<br>`strict_weak_order<less<>, int, int>` → `true` | `R` is a `predicate` for all four orderings of `T` and `U`. The two refinements are semantic promises (equivalence relation; strict weak ordering, as `std::sort` requires). |


### 2.5 Concepts that live in other headers

| Returns | Header | Concepts | Note |
|---|---|---|---|
| `bool` | `<iterator>` | `input_iterator`, `forward_iterator`, `bidirectional_iterator`, `random_access_iterator`, `contiguous_iterator`, `sentinel_for`, `indirectly_readable`, `sortable`, `mergeable`, `permutable`, ... | The iterator hierarchy and the algorithm requirement bundles. |
| `bool` | `<ranges>` | `range`, `view`, `borrowed_range`, `sized_range`, `input_range` ... `contiguous_range`, `viewable_range`, `common_range`, `constant_range` (C++23) | Listed in full in `ranges_views.md`. |
| `bool` | `<random>` | `uniform_random_bit_generator` | `uniform_random_bit_generator<mt19937>` → `true`. Required of the generator argument of `shuffle`, `sample`, distributions. |
| `bool` | `<compare>` | `three_way_comparable`, `three_way_comparable_with` | Listed in 2.2. |
| `bool` | `<execution>` (C++26, not compiled) | the sender / receiver / scheduler concepts of `std::execution` | Introduced with the C++26 asynchronous model. |

---

## 3. Defining a concept

A concept is always a **template** with a `bool` constant expression after `=`. The name is used like `C<T>` and yields a `bool` constant.


| Form | Syntax | Example → result | Note |
|---|---|---|---|
| Constant expression | `template<class T> concept Small = sizeof(T) <= 4;` | `Small<int>` → `true`, `Small<double>` → `false` | Any compile-time `bool`. |
| From a trait | `template<class T> concept Integral = std::is_integral_v<T>;` | usable as `template<Integral T>` | Wrapping a trait in a **named concept** is what makes subsumption work (section 7). |
| Conjunction / disjunction | `template<class T> concept Number = std::integral<T> \|\| std::floating_point<T>;`<br>`template<class T> concept SignedNumber = Number<T> && std::is_signed_v<T>;` | `SignedNumber<int>` → `true`, `SignedNumber<unsigned>` → `false`, `SignedNumber<double>` → `true` | `&&` and `\|\|` combine concepts and boolean expressions. |
| Negation | `template<class T> concept NotIntegral = !std::integral<T>;` | `NotIntegral<double>` → `true`, `NotIntegral<int>` → `false` | `!` makes an *atomic* constraint: it does not take part in subsumption. |
| requires-expression | `template<class T> concept HasSize = requires(const T& t) { t.size(); };` | `HasValueType<int>` (with `typename T::value_type;`) → `false`, `HasValueType<std::vector<int>>` → `true` | An invalid requirement makes the concept `false`; it is not a hard error. |
| Several parameters | `template<class T, class U> concept SameSize = sizeof(T) == sizeof(U);` | `SameSize<int, float>` → `true`, `SameSize<int, char>` → `false` | Used as `template<SameSize<float> T>`: the constrained type is passed as the **first** argument (section 7). |
| Parameter pack | `template<class... Ts> concept AllIntegral = (std::integral<Ts> && ...);` | `AllIntegral<int, char, bool>` → `true`, `AllIntegral<int, double>` → `false` | Fold expression over a pack; needs the parentheses. |
| All requirement kinds | `template<class T> concept Container = requires(T c) {`<br>`  typename T::value_type;`<br>`  { c.size() } -> std::convertible_to<std::size_t>;`<br>`  { c.begin() } -> std::input_or_output_iterator;`<br>`  requires std::copyable<typename T::value_type>;`<br>`};` | `Container<std::vector<int>>` → `true`, `Container<int>` → `false` | Type, compound and nested requirements in one concept (section 4). |
| `noexcept` check | `template<class T> concept NoThrowMove = requires(T a) { { T(std::move(a)) } noexcept; };` | `NoThrowMove<int>` → `true` | A compound requirement can demand `noexcept`. |


Rules for concept definitions (each one verified):

| Attempt | Result |
|---|---|
| `concept C = true;` without a `template<...>` header | error |
| `template<class T> concept R = R<T>;` (recursive) | error |
| `template<> concept C<int> = false;` (specialization) | error |
| `template<std::integral T> concept C = true;` (constraining the concept's own parameter) | error |

---

## 4. The requires-expression

`requires(parameters) { requirements }` is a `bool` expression. Parameters only *describe* values for the requirements; nothing is created or run.

| Kind | Syntax | Example | Meaning |
|---|---|---|---|
| Simple | `expr;` | `a + b;` | The expression is valid (it is not evaluated). |
| Type | `typename T::value_type;` | `typename T::iterator;` | The named type exists. |
| Compound | `{ expr } noexcept -> C<Args...>;` | `{ c.size() } -> std::convertible_to<std::size_t>;` | `expr` is valid; optionally `noexcept`; optionally `decltype((expr))` satisfies `C<decltype((expr)), Args...>`. |
| Nested | `requires constraint;` | `requires std::copyable<typename T::value_type>;` | An extra condition that must be `true` (not just valid). |

Facts (verified):

| Statement | Result |
|---|---|
| `{ t.size() } -> std::size_t;` (a plain type after `->`) | error: a type-constraint (concept) is required |
| `{ t.size() } -> std::convertible_to<std::size_t>;` | compiles |
| Invalid *dependent* expression, e.g. `requires(T t) { t.nothing(); }` with `T = int` | evaluates to `false` |
| Invalid *non-dependent* expression, e.g. `static_assert(requires { std::declval<int>().nothing(); });` | hard error |
| `requires requires(T t) { t.size(); }` | the first `requires` starts a requires-**clause**, the second is the requires-**expression** |

---

## 5. Where a constraint can be written

Every form below was compiled and run (g++ 13.3). `❌` forms are errors.

| Form | Position | Example | Note |
|---|---|---|---|
| Constrained template parameter | in the `template<...>` list | `template<std::integral T> T twice(T x)` | Shortest form for a single constraint. |
| requires-clause after the header | between `template<...>` and the declaration | `template<class T> requires std::integral<T> T twice(T x)` | Needed for several parameters or non-concept conditions. |
| **Trailing requires-clause** | after the function declarator (`)`, `const`, `noexcept`) | `template<class T> T twice(T x) requires std::integral<T>` | Can use the function's parameter names. Also on abbreviated templates: `void f(std::integral auto x) requires (sizeof(x) > 1)`. |
| Abbreviated function template | `auto` in the parameter list | `auto twice(std::integral auto x)` | Every `auto` is its own template parameter. |
| Constrained return type | before the function name | `std::integral auto get_int() { return 42; }` | Constrains the deduced return type. ❌ `requires` itself cannot appear in a return type. |
| Constrained variable | `auto` declaration | `std::integral auto v = 5;` | ❌ `std::integral auto x = 1.5;` is an error; ❌ not allowed on a structured binding. |
| `requires requires` | clause + expression | `template<class T> requires requires(T t) { t.size(); } auto len(const T& t)` | Ad-hoc constraint without naming a concept. |
| Several constraints | requires-clause | `requires std::integral<T> && std::signed_integral<T>` | Concept-ids joined by `&&` / `\|\|` need no parentheses. |
| Class template | parameter or clause | `template<std::integral T> struct Counter { T n{}; };`<br>`template<class T> requires std::floating_point<T> struct Meter { T v{}; };` | |
| Constrained partial specialization | specialization head | `template<std::integral T> struct Info<T> { ... };` | `Info<int>::name` → `"integral"`, `Info<double>::name` → `"generic"`. |
| **Constructor, trailing requires** | after the constructor's `)` | `Box() requires std::default_initializable<T> : v{} {}` | Only for members of a **class template**. |
| **Constructor template, clause before** | between `template<...>` and the constructor | `template<class U> requires std::constructible_from<T, U> explicit Box(U&& u)` | |
| Member function, trailing requires | after the member's `)` / `const` | `void print() const requires requires(const T& t) { std::cout << t; }` | The member exists only when the condition holds. ❌ On a member of a non-template class it is an error. |
| Conditionally trivial destructor | after `~Box()` | `~Box() requires std::is_trivially_destructible_v<T> = default;`<br>`~Box() requires (!std::is_trivially_destructible_v<T>) {}` | `Box<int>` is trivially destructible, `Box<std::string>` is not (verified). |
| Lambda | template list, `auto`, or trailing | `[]<std::integral T>(T x) { return x; }`<br>`[](std::integral auto x) { return x; }`<br>`[]<class T>(T x) requires std::integral<T> { return x; }` | |
| Alias / variable template | template header | `template<std::integral T> using IVec = std::vector<T>;`<br>`template<std::integral T> constexpr T zero = 0;` | |
| Type-constraint with extra arguments | template parameter | `template<std::convertible_to<int> T> int to_int(T x)` | Means `convertible_to<T, int>`. `to_int(2.9)` → `2`. |
| Parameter pack | clause | `template<class... Ts> requires AllIntegral<Ts...> int count_ints(Ts...)` | `count_ints(1, 2, 3)` → `3`. |
| As a plain `bool` | `static_assert`, `if constexpr` | `static_assert(std::integral<int>);`<br>`if constexpr (requires { t.size(); }) { ... }` | Branch on whether an expression is valid; the untaken branch is not compiled. |
| ❌ Non-template function | trailing clause | `void f() requires true;` | Error: only templated entities can be constrained. |
| ❌ Structured binding | `auto [a, b]` | `std::integral auto [a, b] = ...` | Error. |

---

## 6. What is allowed after `requires`

The requires-clause grammar accepts only a few expression shapes unparenthesized.

| requires-clause | Result |
|---|---|
| `requires std::integral<T>` | compiles |
| `requires std::integral<T> && std::signed_integral<T>` | compiles (concept-ids joined by `&&`) |
| `requires std::is_integral_v<T>` | compiles (a single id-expression) |
| `requires (sizeof(T) > 2 && !std::is_integral_v<T>)` | compiles (parenthesized) |
| `requires std::integral<T> && sizeof(T) > 2` | error: needs parentheses around `sizeof(T) > 2` |
| `requires !std::is_integral_v<T>` | error: needs `requires (!std::is_integral_v<T>)` |

Rule: after `requires`, write a concept-id, a single id-expression, or a parenthesized expression; combine those with `&&` / `||`.

---

## 7. Choosing between overloads: subsumption

When several constrained overloads match, the compiler picks the one whose constraints **imply** the others (the most constrained). Verified output of

```cpp
template<class T> const char* pick(T)                         { return "unconstrained"; }
template<std::integral T> const char* pick(T)                 { return "integral"; }
template<std::signed_integral T> const char* pick(T)          { return "signed"; }
```

| Call | Result | Why |
|---|---|---|
| `pick(1)` | `signed` | `signed_integral` is defined as `integral<T> && is_signed_v<T>`, so it subsumes `integral`. |
| `pick(1u)` | `integral` | Unsigned: only the first two match; `integral` is more constrained than the bare template. |
| `pick(1.0)` | `unconstrained` | Neither concept holds. |

Subsumption only works through **named concepts** (verified):

| Code | Result |
|---|---|
| `template<class T> requires std::is_integral_v<T> ...` and `template<class T> requires std::is_integral_v<T> && std::is_signed_v<T> ...` | `pick(1)` is **ambiguous**: the two copies of `is_integral_v<T>` are different expressions |
| `template<class T> concept Integral = std::is_integral_v<T>;` then `template<Integral T> ...` and `template<Integral T> requires std::is_signed_v<T> ...` | `p(1)` → the signed one, `p(1u)` → the plain one |

Other rules:
* `A<T> && B<T>` is more constrained than `A<T>`; `A<T> || B<T>` is less constrained.
* `!C<T>` and parenthesized expressions are atomic: they never subsume anything but themselves.
* For `template<C<X, Y> T>` the compiler checks `C<T, X, Y>`: the constrained type is always the **first** argument (so `std::convertible_to<int> T` means `convertible_to<T, int>`).

---

## 8. Trait or concept: differences that matter (verified)

| Check | Trait result | Concept result |
|---|---|---|
| Private base: `Priv : private Base` | `is_base_of_v<Base, Priv>` → `true` | `derived_from<Priv, Base>` → `false` |
| Explicit constructor `Expl(int)` | `is_convertible_v<int, Expl>` → `false` | `convertible_to<int, Expl>` → `false`; `constructible_from<Expl, int>` → `true` |
| `const int` | `is_default_constructible_v<const int>` → `true` | `default_initializable<const int>` → `false` |
| Throwing destructor `~Th() noexcept(false)` | `is_destructible_v<Th>` → `true` | `destructible<Th>` → `false` |
| Type with a throwing destructor (`Th`) | `is_constructible_v<Th>` → `true` | `constructible_from<Th>` → `false` (needs `destructible`) |
| Assigning to a prvalue | `is_assignable_v<int, int>` → `false` | `assignable_from<int, int>` → `false`; `assignable_from<int&, int>` → `true` |

---

## 9. Patterns

All of these compiled and ran (g++ 13.3, `-std=c++23`).

```cpp
// 1. Define a concept, then use it in three equivalent ways
template<class T> concept Number = std::integral<T> || std::floating_point<T>;

template<Number T> T twice_a(T x) { return x * 2; }                       // constrained parameter
template<class T> requires Number<T> T twice_b(T x) { return x * 2; }     // requires-clause
template<class T> T twice_c(T x) requires Number<T> { return x * 2; }     // trailing requires
auto twice_d(Number auto x) { return x * 2; }                             // abbreviated

// 2. Members that exist only for suitable T
template<class T> struct Box {
    T v;
    Box() requires std::default_initializable<T> : v{} {}
    Box(const T& x) requires std::copy_constructible<T> : v(x) {}
    void print() const requires requires(const T& t) { std::cout << t; } { std::cout << v << "\n"; }
    ~Box() requires std::is_trivially_destructible_v<T> = default;
    ~Box() requires (!std::is_trivially_destructible_v<T>) {}
};

// 3. Branch on "does this expression compile?"
template<class T> void describe(T t) {
    if constexpr (requires { t.size(); }) std::cout << "has size\n";   // std::string -> has size
    else                                  std::cout << "no size\n";    // int         -> no size
}

// 4. A container concept with every requirement kind
template<class T> concept Container = requires(T c) {
    typename T::value_type;                              // type requirement
    { c.size()  } -> std::convertible_to<std::size_t>;   // compound requirement
    { c.begin() } -> std::input_or_output_iterator;
    requires std::copyable<typename T::value_type>;      // nested requirement
};
```

---

## 10. Pitfalls

| Surprise | Why | Fix |
|---|---|---|
| `requires std::integral<T> && sizeof(T) > 2` does not compile | Only concept-ids, id-expressions and parenthesized expressions may appear unparenthesized | `requires std::integral<T> && (sizeof(T) > 2)` |
| `requires !std::is_integral_v<T>` does not compile | `!` is not allowed unparenthesized | `requires (!std::is_integral_v<T>)` |
| Two overloads constrained by the same raw trait are ambiguous | Subsumption compares *named* atomic constraints | Wrap the trait in a concept |
| `template<std::convertible_to<int> T>` reads as "int converts to T" | The constrained type is the **first** argument | It means `convertible_to<T, int>` |
| `{ expr } -> std::size_t;` does not compile | A compound requirement needs a concept after `->`, not a type | `-> std::same_as<std::size_t>` or `-> std::convertible_to<std::size_t>` |
| Trailing `requires` on a plain function or a member of a normal class is an error | Only templated entities can be constrained | Make it a template, or use a member of a class template |
| `std::integral auto x = 1.5;` is a hard error, not SFINAE | An `auto` variable has no overload set to fall back on | Check the type first, or use `static_assert` |
| `requires { std::declval<int>().nothing(); }` outside a template is an error | Non-dependent invalid expressions are not turned into `false` | Put it in a template so it becomes dependent |
| `integral<bool>` and `unsigned_integral<bool>` are `true` | `bool` is an integral type and unsigned | Exclude it with `!std::same_as<T, bool>` |
| `std::regular<T>` passes for a type whose `==` is not an equivalence relation | The compiler checks syntax only | Document and test the semantic requirement yourself |
| `std::destructible` fails for a type with a `noexcept(false)` destructor | Concept needs `noexcept` | Add `noexcept` or use `is_destructible_v` |

---

## 11. Which standard

| Standard | What |
|---|---|
| C++20 | Concepts, `requires` clauses and expressions, constrained `auto`, `<concepts>`, the iterator and range concepts, conditionally trivial special members (`~Box() requires ... = default;`). |
| C++23 | New domain concepts in other headers, for example `constant_range` in `<ranges>`. |
| C++26 | New domain concepts such as the sender / receiver concepts of `std::execution` (not compiled here). |

*Source: g++ 13.3.0 (Ubuntu 24.04), `-std=c++23`. The library-concept results, the 26 syntax checks (compile / error), the overload-selection outputs and the full usage program were all compiled and run.*
