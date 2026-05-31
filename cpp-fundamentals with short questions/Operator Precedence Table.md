# C++ Operator Precedence Table

Higher number = lower precedence. Operators on the same row have equal precedence.

| # | Operators | Description | Associativity |
|---|-----------|-------------|---------------|
| 1 | `::` | Scope resolution | Left-to-right |
| 2 | `a++` `a--` | Postfix increment/decrement | Left-to-right |
|   | `()` | Function call | Left-to-right |
|   | `[]` | Array subscript | Left-to-right |
|   | `.` `->` | Member access | Left-to-right |
| 3 | `++a` `--a` | Prefix increment/decrement | Right-to-left |
|   | `+a` `-a` | Unary plus/minus | Right-to-left |
|   | `!` `~` | Logical NOT / Bitwise NOT | Right-to-left |
|   | `(type)` | C-style cast | Right-to-left |
|   | `*a` | Dereference | Right-to-left |
|   | `&a` | Address-of | Right-to-left |
|   | `sizeof` | Size of type/object | Right-to-left |
|   | `co_await` | Await expression (C++20) | Right-to-left |
|   | `new` `new[]` | Dynamic allocation | Right-to-left |
|   | `delete` `delete[]` | Dynamic deallocation | Right-to-left |
| 4 | `.*` `->*` | Pointer-to-member | Left-to-right |
| 5 | `*` `/` `%` | Multiplication, Division, Modulo | Left-to-right |
| 6 | `+` `-` | Addition, Subtraction | Left-to-right |
| 7 | `<<` `>>` | Bitwise shift left/right | Left-to-right |
| 8 | `<=>` | Three-way comparison (C++20) | Left-to-right |
| 9 | `<` `<=` `>` `>=` | Relational operators | Left-to-right |
| 10 | `==` `!=` | Equality operators | Left-to-right |
| 11 | `&` | Bitwise AND | Left-to-right |
| 12 | `^` | Bitwise XOR | Left-to-right |
| 13 | `\|` | Bitwise OR | Left-to-right |
| 14 | `&&` | Logical AND | Left-to-right |
| 15 | `\|\|` | Logical OR | Left-to-right |
| 16 | `?:` | Ternary conditional | Right-to-left |
|    | `throw` | Throw operator | Right-to-left |
|    | `co_yield` | Yield expression (C++20) | Right-to-left |
| 17 | `=` | Direct assignment | Right-to-left |
|    | `+=` `-=` | Compound assignment (add/sub) | Right-to-left |
|    | `*=` `/=` `%=` | Compound assignment (mul/div/mod) | Right-to-left |
|    | `<<=` `>>=` | Compound assignment (shift) | Right-to-left |
|    | `&=` `^=` `\|=` | Compound assignment (bitwise) | Right-to-left |
| 18 | `,` | Comma operator | Left-to-right |

---

## Common Gotchas

| Expression | What You Might Think | What Actually Happens |
|------------|---------------------|-----------------------|
| `a & b == c` | `(a & b) == c` | `a & (b == c)` — `==` has higher precedence than `&` |
| `a \| b && c` | `(a \| b) && c` | `a \| (b && c)` — `&&` has higher precedence than `\|` |
| `*p++` | `(*p)++` | `*(p++)` — `++` postfix binds tighter than `*` |
| `a << 1 + 2` | `(a << 1) + 2` | `a << (1 + 2)` — `+` has higher precedence than `<<` |
| `!x == y` | `!(x == y)` | `(!x) == y` — `!` has higher precedence than `==` |
| `x = y == z` | `(x = y) == z` | `x = (y == z)` — `==` has higher precedence than `=` |

> **Rule:** When in doubt, use parentheses. Explicit grouping is always clearer than relying on precedence rules.

---

## Quick Memory Aid

```
Scope → Postfix → Prefix/Unary → Ptr-to-member
→ Mul/Div/Mod → Add/Sub → Shift → Spaceship
→ Relational → Equality → Bitwise AND → XOR → OR
→ Logical AND → Logical OR → Ternary → Assignment → Comma
```
