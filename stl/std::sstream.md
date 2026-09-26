# `<sstream>`

### Notation
* `s`: an initial `std::string` to seed the stream's buffer with (optional).
* `mode`: an optional `std::ios::` open-mode bitmask (rarely needed for string streams beyond the default).

---

### Constructors

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| *(constructor)* | default | `istringstream ss;`<br>`ostringstream ss;`<br>`stringstream ss;` | O(1) | Constructs with an empty underlying string buffer. |
| *(constructor)* | from string | `istringstream ss(s)`<br>`ostringstream ss(s)`<br>`stringstream ss(s)` | O(n) | Initializes the underlying buffer's content to `s` (copied in). |
| *(constructor)* | move | `stringstream ss(std::move(other))` | O(1) | Transfers the underlying buffer; streams are not copyable. |

---

### Methods

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| `string` | str() (getter) | `ss.str()` | O(n) | Returns a copy of the current underlying string content. |
| `void` | str(s) (setter) | `ss.str(s)` | O(n) | Replaces the underlying content with `s`, resetting the read/write position. |
| `istream&` / `ostream&` | operator>> / operator<< | `ss >> x;`<br>`ss << x;` | O(n) | Formatted extraction/insertion, same semantics as file/console streams — see `<fstream>` reference for the shared `istream`/`ostream` interface (`getline`, `tellg`/`seekg`, `eof`/`fail`/`good`, etc.). |
| `istream&` | operator>> std::ws | `ss >> std::ws` | O(k), k = whitespace skipped | Skips leading whitespace before the next formatted read — useful inside parsing loops that alternate between whitespace-sensitive and whitespace-insensitive reads. |
| `streambuf*` | rdbuf() | `ss.rdbuf()` | O(1) | Access to the underlying string buffer object directly. |
| *(class)* | istringstream | `istringstream ss(s)` | O(n) | Read-only — parses formatted values out of a string (e.g. tokenizing `"1 2 3"` into ints via repeated `>>`). |
| *(class)* | ostringstream | `ostringstream ss` | O(n) per insertion | Write-only — builds a string via `<<` operators. Allocation-heavy and slow — prefer `std::format`/`to_chars` on hot paths. |
| *(class)* | stringstream | `stringstream ss` | O(n) | Bidirectional version of both above. Same perf caveat as `ostringstream` — use for convenience code/tests, not hot loops. |
