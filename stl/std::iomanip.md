# `<iomanip>`

### Notation
* `n`: an integer argument (precision, width, etc.). `c`: a fill character.
* All entries below are **stream manipulators** — objects returned by a function, meant to be sent via `<<` into a stream (e.g. `std::cout << std::setw(10) << x;`). None are user-constructed classes, so there is no Constructors section.
* Manipulators from `<ios>`/`<iostream>` that always travel alongside `<iomanip>` usage are included here too, since they're used together in practice.

---

### Methods

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| *(manipulator)* | setprecision | `std::setprecision(n)` | O(1) | Number of significant digits (or decimal digits, combined with `fixed`) printed for floating point. |
| *(flag manipulator)* | fixed / scientific | `std::fixed`<br>`std::scientific` | O(1) | Float display style — decimal notation · exponent (`1.23e+04`) notation. |
| *(manipulator)* | setw | `std::setw(n)` | O(1) | Minimum field width for the **next** output only — must be reapplied before every subsequent insertion. |
| *(manipulator)* | setfill | `std::setfill(c)` | O(1) | Padding character used together with `setw` (default is space). Sticky — stays set until changed. |
| *(flag manipulator)* | hex / dec / oct | `std::hex`<br>`std::dec`<br>`std::oct` | O(1) | Number base for integer formatting. Sticky until changed. |
| *(flag manipulator)* | showbase / showpos | `std::showbase`<br>`std::showpos` | O(1) | Adds base prefixes (`0x`, `0`) to hex/octal output · forces a leading `+` on positive numbers. |
| *(flag manipulator)* | noshowbase / noshowpos | `std::noshowbase`<br>`std::noshowpos` | O(1) | Turns the corresponding flag back off. |
| *(flag manipulator)* | uppercase | `std::uppercase` | O(1) | Uses uppercase letters in hex digits and exponent notation (`0XFF`, `1.5E+02`). |
| *(flag manipulator)* | left / right / internal | `std::left`<br>`std::right`<br>`std::internal` | O(1) | Alignment of the field within its `setw` width — pad on the right · pad on the left (default) · pad between the sign and digits. |
| *(flag manipulator)* | boolalpha / noboolalpha | `std::boolalpha`<br>`std::noboolalpha` | O(1) | Prints `bool` as `"true"`/`"false"` instead of `1`/`0` · reverts to numeric. |
| *(manipulator)* | setiosflags / resetiosflags | `std::setiosflags(flags)`<br>`std::resetiosflags(flags)` | O(1) | Sets/clears an arbitrary combination of format flags via a bitmask, for cases the named manipulators above don't cover directly. |
| *(manipulator)* | put_time | `std::put_time(&tm, "%Y-%m-%d")` | O(n) | Formats a `std::tm` calendar time using `strftime`-style format specifiers, for output streams. |
| *(manipulator)* | get_time | `std::get_time(&tm, "%Y-%m-%d")` | O(n) | Parses a calendar time from an input stream using the same format specifiers. |
| *(manipulator)* | put_money / get_money | `std::put_money(cents)`<br>`std::get_money(cents)` | O(n) | Formats/parses a monetary value according to the stream's locale. |
| *(manipulator)* | quoted (C++14) | `std::quoted(str)` | O(n) | Wraps a string in quotes on output (and un-escapes/un-quotes on input) — handy for round-tripping strings with embedded spaces through a stream. |

> **Stream formatting on the hot path — avoid.** All `<iomanip>` manipulators go through `iostream`'s locale-aware formatted output — slow. Use `<charconv>`/`<format>` for latency-sensitive code.
