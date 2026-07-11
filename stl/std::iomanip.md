# `<iomanip>`

| API | Priority | Description |
|-----|----------|-------------|
| `std::setprecision(n)` | know | Sets the number of significant digits (or decimal digits with `fixed`) printed for floating point. |
| `std::fixed` / `std::scientific` | know | Formatting flags controlling float display style — `fixed` for decimal notation, `scientific` for exponent notation. |
| `std::setw(n)` | know | Sets the minimum field width for the *next* output only — must be reapplied every time. |
| `std::setfill(c)` | know | Sets the padding character used with `setw` (default is space). |
| `std::hex` / `std::dec` / `std::oct` | know | Number base for integer formatting. |
| `std::showbase` / `std::showpos` | know | Adds prefixes (`0x`) or forces a `+` sign on positive numbers. |
| `std::left` / `std::right` | know | Alignment of the field within its width. |
| `std::boolalpha` | know | Prints `bool` as "true"/"false" instead of 1/0. |
| Stream formatting on hot path | avoid | All `<iomanip>` manipulators go through `iostream`'s locale-aware formatted output — slow. Use `<charconv>`/`<format>` for latency-sensitive code. |
