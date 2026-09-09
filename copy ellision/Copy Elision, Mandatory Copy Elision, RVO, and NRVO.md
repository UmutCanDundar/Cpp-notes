# Copy Elision, Mandatory Copy Elision, RVO, and NRVO in C++

## 1. Definitions

### Copy Elision (general term)
The compiler is permitted (or, since C++17, in some cases *required*) to skip a copy/move constructor call and construct an object directly in its final storage location, instead of constructing a temporary and then copying/moving it.

### RVO (Return Value Optimization)
Copy elision applied when a function returns a **prvalue** (a temporary, e.g. `return string("x");` or `return string(a, b);`).

### NRVO (Named Return Value Optimization)
Copy elision applied when a function returns a **named local variable or parameter** (an lvalue with a name, e.g. `return result;`).

### Mandatory Copy Elision (C++17, "Guaranteed RVO")
Since C++17, when a prvalue is used to initialize an object (including a function return value), **no temporary is materialized at all** — the object is constructed directly in the destination. This is not an "optimization" the compiler may skip; it is required by the standard. It applies **only to prvalues**, never to named objects.

```
[C++17] return string("x");      // no temporary exists conceptually; mandatory elision
[C++17] string s = string("x");  // same — mandatory
```

**Critical distinction:** Mandatory copy elision in C++17 removed the *temporary materialization* step for prvalues. It did **not** make NRVO mandatory. NRVO remains a *permitted* optimization, never guaranteed by the standard, in any C++ version including 23.

---

## 2. All Scenarios

### Scenario A — Returning a prvalue
```cpp
string func() {
    return string("hello");   // or: return {"hello"};
}
```
- **Pre-C++17:** RVO (optional, but virtually always performed by all major compilers).
- **C++17 and later:** Mandatory copy elision. No copy/move constructor call is even a candidate; it is ill-formed to require one to exist.
- **Constructors called:** 1 (the constructor building the string directly at the caller's location). Zero copy/move ctors.

### Scenario B — Returning a named local object
```cpp
string func() {
    string result = "hello";
    return result;
}
```
- This is **NRVO territory**, not mandatory elision (a name is involved).
- **If NRVO applied (optimizer's choice):** 1 constructor total (direct construction of `result` in caller's slot). Zero copy/move ctor calls.
- **If NRVO not applied:** the compiler must still try to treat `result` as an rvalue for overload resolution (implicit move rule, see §3). So it calls the **move constructor** if available and eligible, otherwise falls back to **copy constructor**.
- **You cannot force NRVO**; it's purely a compiler heuristic (usually enabled at -O1+, defeated by conditional returns of different names, exception paths, etc.).

### Scenario C — Returning a function parameter by value
```cpp
string func(string s) {
    return s;
}
```
- `s` is a named object → NRVO candidate, same rules as Scenario B.
- **Additionally**, constructing `s` itself depends on what the caller passes:
  - Called with an lvalue → **copy constructor** for parameter init.
  - Called with an rvalue (`std::move(x)`, a temporary, a literal) → **move constructor** for parameter init.
- **Best case** (rvalue argument + NRVO applied): 1 move ctor total (parameter init), 0 for the return.
- **Worst case** (lvalue argument + NRVO not applied): 1 copy ctor (parameter init) + 1 move ctor (return, via implicit move rule) = 2 constructor calls.
- This is the classic **"unifying sink parameter" idiom**: pass by value, then move from the parameter inside the body. It adapts automatically to lvalue/rvalue callers and is *at least as good as*, often better than, taking `const&` + copying internally.

### Scenario D — Returning via `const&` parameter, copying inside
```cpp
string func(const string& s) {
    auto result{s};     // must copy: cannot move from a const&
    return result;      // NRVO candidate
}
```
- Copying `s` into `result` is **never a move** — `s` is `const&`, move ctor isn't viable (would need to bind non-const rvalue-ref), so this is unconditionally a **copy constructor** call. Not eligible for elision of any kind (two distinct objects, not an initialization from a prvalue).
- `return result;` → NRVO candidate exactly like Scenario B: 0 extra calls if NRVO applies, otherwise 1 move (or copy, if move ctor absent).
- **Total: minimum 1 copy ctor (mandatory) + 0-or-1 move/copy** — strictly never as cheap as Scenario C's best case, because the initial copy from `const&` can never be avoided or turned into a move, regardless of what the caller passes.

### Scenario E — Returning different named variables on different branches
```cpp
string func(bool b) {
    string a = "a";
    string c = "c";
    if (b) return a;
    return c;
}
```
- NRVO is **disabled** by the standard here: two different named variables can't share one elided slot deterministically across compilers/paths. (Some compilers may still special-case identical types via extra tricks, but this is **not guaranteed** and commonly does *not* elide.)
- Falls back to the implicit-move-on-return rule for whichever variable is returned on the taken path.

### Scenario F — Returning a parameter that is also modified
```cpp
string func(string s) {
    s += "!";
    return s;
}
```
- Same as Scenario C structurally. Mutating `s` before returning does not disable NRVO by itself.

### Scenario G — `std::move` on a named return value
```cpp
string func() {
    string result = "hello";
    return std::move(result);   // ANTI-PATTERN
}
```
- Explicit `std::move` **disables NRVO**. The compiler now sees an xvalue expression, not a plain named-lvalue return, and NRVO is only defined for the exact pattern `return <local-name>;`.
- Forces a **move constructor** call (no elision possible), which is strictly worse than or equal to a correctly NRVO-eligible plain `return result;`.
- **Never write `return std::move(local);`** — it's pessimization, not optimization.

### Scenario H — Returning a member/global, or `std::move`d parameter of different type
```cpp
string func(string s) {
    return std::move(s);   // still an anti-pattern for the same reason as G
}
Base func(Derived d) {
    return d;   // NRVO impossible: type mismatch (Derived -> Base), copy/move ctor of Base is used with a Derived arg... actually triggers Base's converting ctor
}
```
- Type mismatch between the named local and the return type disqualifies NRVO entirely (elision requires identical type, ignoring cv-qualification). A conversion/converting-constructor call is required.

### Scenario I — Function-local static or thrown/caught exception object
- **Static locals** are never elided (they don't get destroyed at scope exit, so "returning" one is really invoking copy/move like Scenario B but the source persists — same NRVO rule still applies for the *return*, the static-ness is irrelevant to elision itself).
- **Exception objects** (`throw expr;` and the catch parameter) have their own elision rules, analogous to NRVO but governed separately (`throw` on a local name is elision-eligible in many compilers, not standardized as mandatory).

### Quick Reference Table — Scenarios

| # | Code pattern | Elision type | Guaranteed? | Ctor calls (best case) | Ctor calls (worst case) |
|---|---|---|---|---|---|
| A | `return T(args);` (prvalue) | Mandatory elision (C++17+) | ✅ Yes (C++17+) | 0 | 0 |
| B | `T r = ...; return r;` | NRVO | ❌ No (compiler heuristic) | 0 | 1 move (or copy if no move ctor) |
| C | `T func(T s) { return s; }`, called with rvalue | NRVO + param move | ❌ No | 1 move (param init only) | 1 move (param) + 1 move (return) |
| C' | same, called with lvalue | NRVO + param copy | ❌ No | 1 copy (param init only) | 1 copy (param) + 1 move (return) |
| D | `func(const T&) { auto r{s}; return r; }` | Mandatory copy + NRVO | ❌ No (copy is mandatory, NRVO isn't) | 1 copy (mandatory) | 1 copy + 1 move |
| E | `if(b) return a; return c;` (different names) | None possible | ❌ Never | 1 move/copy | 1 move/copy |
| G | `return std::move(result);` | **Disables NRVO** | ❌ Actively broken | 1 move | 1 move |
| H | Returning mismatched type (`Base` from `Derived`) | None possible | ❌ Never | 1 conversion ctor | 1 conversion ctor |

---

## 3. The "Implicit Move on Return" Rule (why NRVO failure isn't always a copy)

When a function returns a named object (id-expression naming a variable/parameter of automatic storage duration matching the return type) and NRVO isn't performed, the standard mandates that overload resolution for the return is first performed **as if the named object were an rvalue**. Only if no viable move constructor is found does it fall back to copy. This is why Scenarios B, C, D, F all say "move constructor if available, else copy" as the fallback — never a hard copy unless no move ctor exists (e.g., a copy-only type).

---

## 4. Final Ranking Table — Best to Worst

| Rank | Pattern | Total ctor calls | Why |
|---|---|---|---|
| 1 | `return T(args);` (prvalue, C++17+) | **0** | Mandatory elision — guaranteed by standard |
| 2 | Sink param (`T func(T s){return s;}`), called with rvalue, NRVO applies | **1 move** | Param init needs 1 move (ownership transfer); return elided |
| 3 | Sink param, called with lvalue, NRVO applies | **1 copy** | Param init needs 1 copy (source preserved); return elided |
| 4 | Plain local, NRVO applies (`T r=...; return r;`) | **0** | Not guaranteed, but reliable on GCC/Clang/MSVC for simple returns |
| 5 | Plain local, NRVO fails | **1 move** (or copy if no move ctor) | Implicit-move-on-return rule (§3) |
| 6 | `const T&` param + internal copy, NRVO applies to the copy | **1 copy** (mandatory) | Copy from `const&` can never be a move; unconditional regardless of caller |
| 7 | `const T&` param + internal copy, NRVO fails | **1 copy + 1 move** (or 2 copies) | Worst *legitimate* path |
| 8 | `return std::move(local);` | **1 move** | Actively disables NRVO — pessimization, never justified |
| 9 | Mismatched types / different named vars per branch | **1 conversion/copy/move** | Elision impossible by rule — no fix except redesign |

## 5. Design Rules (short)

| Situation | Use |
|---|---|
| Function only **reads** input, builds a new value | `const T&` param, return a plain local (`return result;`), never `std::move` it |
| Function **consumes/transforms** input into output | Pass **by value**, move internally, `return` the named parameter as-is (sink-parameter idiom) |
| Building the return value inline | Prefer `return T(...);` — the only pattern with a **hard C++17 guarantee** |
| Any named return | **Never** write `return std::move(x);` — it defeats NRVO for a move you'd often get for free (rank 4) |
