# String ↔ Number Conversions + `string_view`

| Function | Usage | Note |
|----------|-------|------|
| `std::to_string` | `to_string(42)` / `to_string(3.14)` | Number to string. Not fast — allocates. |
| `std::stoi` | `stoi("42")` / `stoi("0xFF", nullptr, 16)` | String → int. Base can be specified. Throws `invalid_argument`. |
| `std::stol` / `stoll` | `stoll("9999999999")` | long / long long. |
| `std::stof` / `stod` | `stod("3.14")` | float / double. |
| `std::from_chars` (C++17) | `from_chars(s.data(), s.data()+s.size(), val)` | No allocation, no exceptions. Preferred for HFT. Check error with `result.ec`. |
| `std::to_chars` (C++17) | `to_chars(buf, buf+20, val)` | Number to char buffer. Fastest method, no allocation. |
| `std::string_view` (C++17) | `string_view sv = s;` / `sv.substr(0,3)` | No allocation, just pointer+size. Read-only. Most string methods available here too. |
| `std::ostringstream` | `oss << x << y; oss.str()` | Concatenate multiple things. Slow but easy. |
| `std::istringstream` | `iss.str(s); iss >> x >> y` | Parse from string. Use `getline` for tokenizing. |
| `std::format` (C++20) | `format("val={}", x)` | Python f-string style. Type-safe replacement for sprintf. |

## Quick Init Patterns

| Pattern | Code |
|---------|------|
| n copies of a char | `string s(5, 'x')` → `"xxxxx"` |
| Concatenate | `string s = a + " " + b` |
| Append | `s += " world"` |
| Find and replace substring | `size_t p = s.find("old"); if(p!=npos) s.replace(p,3,"new")` |
| Trim left | `s.erase(0, s.find_first_not_of(" \t\n"))` |
| Trim right | `s.erase(s.find_last_not_of(" \t\n")+1)` |
| Convert to uppercase | `transform(s.begin(),s.end(),s.begin(),::toupper)` |
