# `<fstream>`

### Notation
* `path`: a filename, as `const char*`, `std::string`, or `std::filesystem::path`.
* `mode`: a bitmask of `std::ios::` open-mode flags (see table below).
* `buf`, `n`: a raw `char*` buffer and its size in bytes, for unformatted I/O.

---

### Constructors

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| *(constructor)* | default | `ifstream f;` / `ofstream f;` / `fstream f;` | O(1) | Constructs a closed stream, not yet associated with a file. |
| *(constructor)* | from path | `ifstream f(path)`<br>`ofstream f(path)`<br>`fstream f(path, mode)` | O(1) (1 syscall) | Opens the file immediately. Check success with `if (!f)` or `.is_open()` — a failed open does **not** throw by default. |
| *(constructor)* | move | `ifstream f(std::move(other))` | O(1) | Transfers ownership of the open file handle; streams are not copyable. |

---

### Methods

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| `void` | open | `f.open(path, mode)` | O(1) (1 syscall) | Opens a file on an already-constructed (or previously closed) stream. |
| `bool` | is_open | `f.is_open()` | O(1) | Whether a file is currently associated and open. |
| `void` | close | `f.close()` | O(1) (1 syscall) | Closes the file explicitly; also happens automatically via RAII on destruction. |
| `istream&` | read | `f.read(buf, n)` | O(n) | Unformatted binary read of exactly `n` bytes into `buf` — fastest way to pull raw bytes from a file. |
| `ostream&` | write | `f.write(buf, n)` | O(n) | Unformatted binary write of `n` bytes from `buf` — fastest way to push raw bytes to a file. |
| `streamsize` | gcount | `f.gcount()` | O(1) | Number of characters actually extracted by the last unformatted input operation (useful if `read` hit EOF early). |
| `istream&` | getline | `std::getline(f, str)` | O(n) | Reads a full line into a `std::string`, stopping at (and discarding) `'\n'`. |
| `streambuf*` | rdbuf | `f.rdbuf()` | O(1) | Access the underlying stream buffer — e.g. to redirect `std::cout` to a file (`std::cout.rdbuf(f.rdbuf())`). |
| `pos_type` | tellg / tellp | `f.tellg()` / `f.tellp()` | O(1) | Current read position / current write position. |
| `istream&` / `ostream&` | seekg / seekp | `f.seekg(off, dir)`<br>`f.seekp(off, dir)` | O(1) | Moves the read/write position, `dir` is `ios::beg`/`ios::cur`/`ios::end`. |
| `bool` | eof / fail / bad / good | `f.eof()`<br>`f.fail()`<br>`f.bad()`<br>`f.good()` | O(1) | End-of-file reached · a recoverable format/logic error occurred · an unrecoverable I/O error occurred · none of the above (stream is fully usable). |
| `void` | clear | `f.clear()` | O(1) | Resets the stream's error state flags, e.g. after checking/handling `eof()`. |
| `istream&` / `ostream&` | operator>> / operator<< | `f >> x;`<br>`f << x;` | O(n) | Formatted, allocation-heavy, locale-aware I/O — slow for high-throughput logging; prefer `.write()`/buffered binary I/O on hot paths. |

---

### Open Mode Flags (`std::ios::`)

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| *(flag)* | binary | `std::ios::binary` | N/A | Disables text-mode newline translation — required for raw/binary data. |
| *(flag)* | app | `std::ios::app` | N/A | Append — all writes go to the end of the file, existing content preserved. |
| *(flag)* | trunc | `std::ios::trunc` | N/A | Truncate — clears existing content when the file is opened. |
| *(flag)* | in / out | `std::ios::in` / `std::ios::out` | N/A | Open for reading / open for writing (implicit for `ifstream`/`ofstream` respectively). |
| *(flag)* | ate | `std::ios::ate` | N/A | "At end" — seeks to the end immediately after opening (unlike `app`, subsequent writes are **not** forced to stay at the end). |

> Combine flags with `|`, e.g. `ofstream f(path, std::ios::binary | std::ios::app)`.
