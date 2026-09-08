# C++ `<iostream>` and Stream Library — Full Reference

## Class Hierarchy

```
ios_base
  └─ basic_ios<CharT, Traits>
        ├─ basic_istream<CharT, Traits>
        │     ├─ basic_iostream<CharT, Traits>  (also inherits basic_ostream)
        │     ├─ basic_ifstream<CharT, Traits>       (<fstream>)
        │     ├─ basic_istringstream<CharT, Traits>  (<sstream>)
        │     └─ basic_istream (istream_iterator source, etc.)
        └─ basic_ostream<CharT, Traits>
              ├─ basic_iostream<CharT, Traits>
              ├─ basic_ofstream<CharT, Traits>       (<fstream>)
              └─ basic_ostringstream<CharT, Traits>  (<sstream>)

basic_iostream<CharT, Traits>
  ├─ basic_fstream<CharT, Traits>       (<fstream>)
  └─ basic_stringstream<CharT, Traits>  (<sstream>)
```

Typedefs: `std::ios` = `basic_ios<char>`, `std::istream` = `basic_istream<char>`, `std::ostream` = `basic_ostream<char>`, `std::iostream` = `basic_iostream<char>`. Wide-char versions prefixed `w` (`std::wios`, `std::wistream`, ...).

Global objects (`<iostream>`): `std::cin`, `std::cout`, `std::cerr`, `std::clog` (and wide versions `std::wcin`, `std::wcout`, `std::wcerr`, `std::wclog`).

---

## `ios_base`

Base class holding format state, error state, locale, and static constants. Not templated on `CharT`.

### Nested types / data

| Name | Type | Meaning |
|---|---|---|
| `fmtflags` | bitmask type | formatting flags (see [Formatting Flags](#formatting-flags-fmtflags)) |
| `iostate` | bitmask type | stream error state (see [Stream State Flags](#stream-state-flags-iostate)) |
| `openmode` | bitmask type | file open mode (see [Open Mode Flags](#open-mode-flags-openmode)) |
| `seekdir` | enum | seek direction (see [Seek Direction](#seek-direction-seekdir)) |
| `Init` | nested class | ensures standard streams are initialized before `main()` |
| `event` | enum | `erase_event`, `imbue_event`, `copyfmt_event` — passed to registered callbacks |
| `event_callback` | function pointer type | `void(*)(event, ios_base&, int)` |
| `failure` | nested class, derives `system_error` (C++11) / `exception` (pre-C++11) | thrown by `exceptions()` mechanism |

### Member functions

| Signature | Returns | Description |
|---|---|---|
| `fmtflags flags() const` | current flags | get all format flags |
| `fmtflags flags(fmtflags flags)` | previous flags | replace all format flags |
| `fmtflags setf(fmtflags flags)` | previous flags | set (OR in) given flags |
| `fmtflags setf(fmtflags flags, fmtflags mask)` | previous flags | set flags within `mask` only, clearing others in that mask |
| `void unsetf(fmtflags flags)` | `void` | clear given flags |
| `streamsize precision() const` | current precision | floating-point precision (default 6) |
| `streamsize precision(streamsize new_precision)` | previous precision | set precision |
| `streamsize width() const` | current field width | width for next formatted operation |
| `streamsize width(streamsize new_width)` | previous width | set width (resets to 0 after each formatted I/O op) |
| `locale imbue(const locale& loc)` | previous locale | set the imbued locale |
| `locale getloc() const` | current locale | get the imbued locale |
| `static int xalloc()` | unique index | allocate a unique index for `iword`/`pword` extensible storage |
| `long& iword(int idx)` | reference to long | per-stream, per-index integer storage slot |
| `void*& pword(int idx)` | reference to void* | per-stream, per-index pointer storage slot |
| `static bool sync_with_stdio(bool sync = true)` | previous sync state | enable/disable synchronization with C stdio |
| `void register_callback(event_callback fn, int idx)` | `void` | register a callback invoked on copyfmt/imbue/destruction |
| `ios_base()` (protected) | — | default constructor, must be followed by `basic_ios::init` |
| `virtual ~ios_base()` | — | virtual destructor |

### Static format-flag constants (members of `ios_base`)

`skipws, unitbuf, uppercase, showbase, showpoint, showpos, left, right, internal, dec, oct, hex, scientific, fixed, boolalpha` (fmtflags); `goodbit, badbit, failbit, eofbit` (iostate); `app, ate, binary, in, out, trunc` (openmode); `beg, cur, end` (seekdir). Full meanings tabulated in the Flags section below.

---

## `basic_ios<CharT, Traits>`

Inherits `ios_base`. Adds the stream-state / buffer / tie machinery common to both input and output streams. Templated on character type and traits.

### Data (conceptual — actual members are implementation-defined, but this is the state it manages)

| Conceptual member | Type | Meaning |
|---|---|---|
| rdstate | `iostate` | current error state |
| tie target | `basic_ostream<CharT,Traits>*` | stream to flush before I/O on this stream |
| stream buffer | `basic_streambuf<CharT,Traits>*` | underlying buffer |
| fill character | `CharT` | padding character for field width |

### Member functions

| Signature | Returns | Description |
|---|---|---|
| `bool good() const` | bool | `rdstate() == 0` (no bits set) |
| `bool eof() const` | bool | `eofbit` set |
| `bool fail() const` | bool | `failbit` or `badbit` set |
| `bool bad() const` | bool | `badbit` set |
| `bool operator!() const` | bool | equivalent to `fail()` |
| `explicit operator bool() const` | bool | equivalent to `!fail()` |
| `iostate rdstate() const` | iostate | get current state flags |
| `void setstate(iostate state)` | `void` | OR additional state bits in (calls `clear(rdstate() \| state)`) |
| `void clear(iostate state = goodbit)` | `void` | replace state flags entirely (throws if new state & `exceptions()` mask is nonzero) |
| `iostate exceptions() const` | iostate | mask of states that trigger an exception throw |
| `void exceptions(iostate except)` | `void` | set the exception-triggering mask |
| `basic_ostream<CharT,Traits>* tie() const` | pointer | get tied output stream (flushed before this stream's I/O) |
| `basic_ostream<CharT,Traits>* tie(basic_ostream<CharT,Traits>* newtie)` | previous tie | set tied output stream |
| `basic_streambuf<CharT,Traits>* rdbuf() const` | pointer | get underlying stream buffer |
| `basic_streambuf<CharT,Traits>* rdbuf(basic_streambuf<CharT,Traits>* sb)` | previous buf | set underlying stream buffer, clears state to `goodbit` |
| `basic_ios& copyfmt(const basic_ios& other)` | `*this` | copy formatting state, locale, exception mask, tie, and callbacks (not the buffer) |
| `CharT fill() const` | CharT | get fill character |
| `CharT fill(CharT ch)` | previous fill | set fill character |
| `CharT narrow(CharT c, char dflt) const` | char | narrow a character using imbued locale |
| `CharT widen(char c) const` | CharT | widen a character using imbued locale |
| `basic_ios(basic_streambuf<CharT,Traits>* sb)` (protected/explicit) | — | construct, associating a buffer |
| `void init(basic_streambuf<CharT,Traits>* sb)` (protected) | `void` | initialize state (used by derived class constructors) |
| `void move(basic_ios& rhs)` (protected) | `void` | move state from another `basic_ios` |
| `void swap(basic_ios& rhs) noexcept` (protected) | `void` | swap state |
| `void set_rdbuf(basic_streambuf<CharT,Traits>* sb)` (protected) | `void` | set buffer without clearing state |

---

## `basic_istream<CharT, Traits>`

Inherits `basic_ios`. Provides formatted and unformatted input.

### Nested class: `sentry`

Constructed at the start of every input operation; checks stream state, skips whitespace if `skipws` is set (for formatted extraction), and flushes the tied stream. `explicit operator bool() const` reports whether it's safe to proceed.

### Member functions

| Signature | Returns | Description |
|---|---|---|
| `basic_istream& operator>>(int& val)` (and overloads for all arithmetic types, `bool`, pointers, `basic_streambuf*`) | `*this` | formatted extraction |
| `basic_istream& operator>>(basic_istream& (*func)(basic_istream&))` | `*this` | manipulator support (e.g., `std::ws`) |
| `streamsize gcount() const` | streamsize | number of characters extracted by last unformatted input op |
| `int_type get()` | int_type | extract single character (or `Traits::eof()`) |
| `basic_istream& get(CharT& ch)` | `*this` | extract single character into `ch` |
| `basic_istream& get(CharT* s, streamsize count)` | `*this` | extract up to `count-1` chars until `'\n'` or EOF, null-terminate |
| `basic_istream& get(CharT* s, streamsize count, CharT delim)` | `*this` | same, custom delimiter |
| `basic_istream& get(basic_streambuf<CharT,Traits>& sb)` | `*this` | extract into a streambuf until `'\n'` |
| `basic_istream& get(basic_streambuf<CharT,Traits>& sb, CharT delim)` | `*this` | same, custom delimiter |
| `basic_istream& getline(CharT* s, streamsize count)` | `*this` | like `get` but also extracts and discards the delimiter |
| `basic_istream& getline(CharT* s, streamsize count, CharT delim)` | `*this` | same, custom delimiter |
| `basic_istream& ignore(streamsize count = 1, int_type delim = Traits::eof())` | `*this` | extract and discard up to `count` chars, stopping early at `delim` |
| `int_type peek()` | int_type | look at next character without extracting it |
| `basic_istream& read(CharT* s, streamsize count)` | `*this` | extract exactly `count` characters (sets `failbit` if fewer available) |
| `streamsize readsome(CharT* s, streamsize count)` | streamsize | extract up to `count` chars, only as many as immediately available (non-blocking-ish) |
| `basic_istream& putback(CharT ch)` | `*this` | push one character back into the stream |
| `basic_istream& unget()` | `*this` | step back one character in the stream |
| `int sync()` | int | synchronize with underlying source, `-1` on failure |
| `pos_type tellg()` | pos_type | current get-position |
| `basic_istream& seekg(pos_type pos)` | `*this` | set absolute get-position |
| `basic_istream& seekg(off_type off, ios_base::seekdir dir)` | `*this` | set relative get-position |
| `basic_istream(basic_streambuf<CharT,Traits>* sb)` (explicit) | — | construct bound to a buffer |
| `basic_istream& operator>>(basic_istream& (*)(basic_istream&))` | `*this` | manipulator overload |

### Non-member function template

| Signature | Returns | Description |
|---|---|---|
| `template<class CharT,class Traits> basic_istream<CharT,Traits>& operator>>(basic_istream<CharT,Traits>&, CharT&)` | stream ref | extract single char |
| `operator>>(basic_istream&, CharT*)` | stream ref | extract a whitespace-delimited word into a buffer |
| `operator>>(basic_istream&, basic_string<CharT,Traits,Alloc>&)` (`<string>`) | stream ref | extract into `std::string` |

---

## `basic_ostream<CharT, Traits>`

Inherits `basic_ios`. Provides formatted and unformatted output.

### Nested class: `sentry`

Constructed at the start of every output operation; flushes the tied input stream. `explicit operator bool() const` reports readiness.

### Member functions

| Signature | Returns | Description |
|---|---|---|
| `basic_ostream& operator<<(int val)` (and overloads for all arithmetic types, `bool`, pointers, `basic_streambuf*`) | `*this` | formatted insertion |
| `basic_ostream& operator<<(basic_ostream& (*func)(basic_ostream&))` | `*this` | manipulator support (e.g., `std::endl`) |
| `basic_ostream& operator<<(ios_base& (*func)(ios_base&))` | `*this` | manipulator support (e.g., `std::hex`) |
| `basic_ostream& put(CharT ch)` | `*this` | insert single character (unformatted) |
| `basic_ostream& write(const CharT* s, streamsize count)` | `*this` | insert `count` raw characters |
| `basic_ostream& flush()` | `*this` | flush the associated buffer |
| `pos_type tellp()` | pos_type | current put-position |
| `basic_ostream& seekp(pos_type pos)` | `*this` | set absolute put-position |
| `basic_ostream& seekp(off_type off, ios_base::seekdir dir)` | `*this` | set relative put-position |
| `basic_ostream(basic_streambuf<CharT,Traits>* sb)` (explicit) | — | construct bound to a buffer |

### Non-member function template

| Signature | Returns | Description |
|---|---|---|
| `operator<<(basic_ostream&, CharT)` / `(basic_ostream&, char)` | stream ref | insert single character |
| `operator<<(basic_ostream&, const CharT*)` | stream ref | insert C-string |
| `operator<<(basic_ostream&, const basic_string<CharT,Traits,Alloc>&)` (`<string>`) | stream ref | insert `std::string` |

---

## `basic_iostream<CharT, Traits>`

Multiply inherits `basic_istream` and `basic_ostream`. Adds no new members beyond constructors:

| Signature | Description |
|---|---|
| `basic_iostream(basic_streambuf<CharT,Traits>* sb)` (explicit) | construct bound to a shared buffer for both directions |
| `virtual ~basic_iostream()` | destructor |

---

## Formatting Flags (`fmtflags`)

| Flag | Group | Effect |
|---|---|---|
| `skipws` | whitespace | skip leading whitespace on formatted extraction |
| `unitbuf` | buffering | flush after every output operation |
| `uppercase` | case | use uppercase letters for hex digits / scientific `E` |
| `showbase` | number formatting | show numeric base prefix (`0x`, `0`) on output |
| `showpoint` | number formatting | always show decimal point in floating output |
| `showpos` | number formatting | show `+` before positive numbers |
| `left` | adjustment (mutually exclusive group) | left-justify within field width |
| `right` | adjustment | right-justify within field width (default) |
| `internal` | adjustment | pad between sign/base prefix and value |
| `dec` | base (mutually exclusive group) | integers in decimal |
| `oct` | base | integers in octal |
| `hex` | base | integers in hexadecimal |
| `scientific` | float format (mutually exclusive group) | scientific notation |
| `fixed` | float format | fixed-point notation |
| `boolalpha` | bool formatting | print `bool` as `true`/`false` instead of `1`/`0` |

`ios_base::adjustfield`, `ios_base::basefield`, `ios_base::floatfield` are masks grouping the mutually-exclusive sets above, used with the two-argument `setf(flags, mask)`.

---

## Stream State Flags (`iostate`)

| Flag | Meaning |
|---|---|
| `goodbit` | no error (value `0`) |
| `eofbit` | end-of-file reached during an input operation |
| `failbit` | logical error (e.g., type mismatch on extraction, formatting failure) — stream still usable after `clear()` |
| `badbit` | serious I/O error (e.g., loss of integrity of the stream buffer) |

---

## Open Mode Flags (`openmode`)

| Flag | Meaning |
|---|---|
| `app` | seek to end before every write (append) |
| `ate` | seek to end immediately after opening (at-end, but subsequent writes are not forced to end) |
| `binary` | open in binary mode (no text translation) |
| `in` | open for reading |
| `out` | open for writing |
| `trunc` | discard existing file contents on open |
| `noreplace` (C++23) | fail if the file already exists (used with `out`) |

---

## Seek Direction (`seekdir`)

| Value | Meaning |
|---|---|
| `beg` | seek relative to beginning of stream |
| `cur` | seek relative to current position |
| `end` | seek relative to end of stream |

---

## `<fstream>`

### `basic_filebuf<CharT, Traits>`

Underlying stream buffer for file streams (rarely used directly).

| Signature | Returns | Description |
|---|---|---|
| `bool is_open() const` | bool | whether a file is currently associated |
| `basic_filebuf* open(const char* filename, ios_base::openmode mode)` | pointer or `nullptr` | open a file |
| `basic_filebuf* close()` | pointer or `nullptr` | close the file, flushing first |

### `basic_ifstream<CharT, Traits>` (inherits `basic_istream`)

| Signature | Returns | Description |
|---|---|---|
| `basic_ifstream()` | — | default constructor |
| `explicit basic_ifstream(const char* filename, ios_base::openmode mode = ios_base::in)` | — | construct and open |
| `explicit basic_ifstream(const string& filename, ios_base::openmode mode = ios_base::in)` | — | same, `std::string` overload |
| `void open(const char* filename, ios_base::openmode mode = ios_base::in)` | `void` | open a file |
| `void open(const string& filename, ios_base::openmode mode = ios_base::in)` | `void` | same |
| `bool is_open() const` | bool | whether a file is open |
| `void close()` | `void` | close the file |
| `basic_filebuf<CharT,Traits>* rdbuf() const` | pointer | get the associated filebuf |

### `basic_ofstream<CharT, Traits>` (inherits `basic_ostream`)

Same member set as `basic_ifstream` but default mode `ios_base::out`, and inherited operations are output-oriented.

### `basic_fstream<CharT, Traits>` (inherits `basic_iostream`)

Same member set, default mode `ios_base::in | ios_base::out`. Supports both directions on a single file handle.

---

## `<sstream>`

### `basic_stringbuf<CharT, Traits, Allocator>`

| Signature | Returns | Description |
|---|---|---|
| `basic_string<CharT,Traits,Allocator> str() const` | string | copy of the buffer's current contents |
| `void str(const basic_string<CharT,Traits,Allocator>& s)` | `void` | replace buffer contents |
| `basic_string_view<CharT,Traits> view() const noexcept` (C++20) | string_view | non-owning view of buffer contents (avoids a copy) |

### `basic_istringstream<CharT, Traits, Allocator>` (inherits `basic_istream`)

| Signature | Description |
|---|---|
| `explicit basic_istringstream(ios_base::openmode mode = ios_base::in)` | default constructor |
| `explicit basic_istringstream(const basic_string<...>& str, ios_base::openmode mode = ios_base::in)` | construct with initial content |
| `basic_stringbuf<CharT,Traits,Allocator>* rdbuf() const` | get underlying stringbuf |
| `basic_string<...> str() const` | get copy of buffer contents |
| `void str(const basic_string<...>& s)` | replace buffer contents |

### `basic_ostringstream<CharT, Traits, Allocator>` (inherits `basic_ostream`)

Same member set as `basic_istringstream`, default mode `ios_base::out`.

### `basic_stringstream<CharT, Traits, Allocator>` (inherits `basic_iostream`)

Same member set, default mode `ios_base::in | ios_base::out`.

---

## `<iomanip>`

Manipulators that take arguments (return unspecified proxy objects usable with `operator<<`/`operator>>`).

| Manipulator | Effect |
|---|---|
| `setw(int n)` | set field width for the next formatted I/O operation |
| `setfill(CharT c)` | set fill character |
| `setprecision(int n)` | set floating-point precision |
| `setbase(int base)` | set output integer base (`8`, `10`, `16`; others reset to default) |
| `setiosflags(ios_base::fmtflags mask)` | set flags matching `mask` |
| `resetiosflags(ios_base::fmtflags mask)` | clear flags matching `mask` |
| `get_money(MoneyT& amount, bool intl = false)` | parse a monetary value on extraction |
| `put_money(const MoneyT& amount, bool intl = false)` | format a monetary value on insertion |
| `get_time(struct tm* tmb, const char* fmt)` | parse a date/time on extraction per `fmt` |
| `put_time(const struct tm* tmb, const char* fmt)` | format a date/time on insertion per `fmt` |
| `quoted(const CharT* s, CharT delim = '"', CharT escape = '\\')` | insert/extract a quoted string, escaping embedded delimiters |
| `quoted(basic_string<...>& s, ...)` | same, for `std::string`, supports round-trip extraction |

---

## Manipulators Without Arguments (in `<ios>` / `<ostream>` / `<istream>`)

| Manipulator | Header | Effect |
|---|---|---|
| `endl` | `<ostream>` | insert `'\n'` and flush |
| `ends` | `<ostream>` | insert `'\0'` |
| `flush` | `<ostream>` | flush the stream |
| `ws` | `<istream>` | extract and discard leading whitespace |
| `boolalpha` / `noboolalpha` | `<ios>` | toggle `boolalpha` flag |
| `showbase` / `noshowbase` | `<ios>` | toggle `showbase` flag |
| `showpoint` / `noshowpoint` | `<ios>` | toggle `showpoint` flag |
| `showpos` / `noshowpos` | `<ios>` | toggle `showpos` flag |
| `skipws` / `noskipws` | `<ios>` | toggle `skipws` flag |
| `uppercase` / `nouppercase` | `<ios>` | toggle `uppercase` flag |
| `unitbuf` / `nounitbuf` | `<ios>` | toggle `unitbuf` flag |
| `left` / `right` / `internal` | `<ios>` | set adjustment field |
| `dec` / `hex` / `oct` | `<ios>` | set base field |
| `fixed` / `scientific` | `<ios>` | set float format field |
| `hexfloat` (C++11) | `<ios>` | hexadecimal floating-point format (`fixed \| scientific` combination) |
| `defaultfloat` (C++11) | `<ios>` | reset float format to default |

---

## Global Stream Objects (`<iostream>`)

| Object | Type | Bound to | Tied to |
|---|---|---|---|
| `cin` | `istream` | stdin | `cout` (flushed before `cin` reads) |
| `cout` | `ostream` | stdout | — |
| `cerr` | `ostream` | stderr | `cout`; unbuffered (`unitbuf` set) |
| `clog` | `ostream` | stderr | — (buffered, unlike `cerr`) |
| `wcin` / `wcout` / `wcerr` / `wclog` | wide versions | same targets | same tying rules |

`std::ios_base::sync_with_stdio(bool)` controls whether these are synchronized with C's `stdio` streams — disabling it (typically `sync_with_stdio(false)`) can significantly speed up `cin`/`cout` but makes mixing with `printf`/`scanf` unsafe.

---

## `basic_streambuf<CharT, Traits>` (`<streambuf>`)

The abstract buffer base class underlying all stream classes. Key protected virtual interface (overridden by `basic_filebuf`, `basic_stringbuf`, etc.):

| Signature | Description |
|---|---|
| `virtual int_type overflow(int_type ch = Traits::eof())` | handle buffer full during output |
| `virtual int_type underflow()` | handle buffer empty during input (peek without consuming) |
| `virtual int_type uflow()` | like `underflow` but also consumes the character |
| `virtual streamsize xsputn(const CharT* s, streamsize count)` | bulk output |
| `virtual streamsize xsgetn(CharT* s, streamsize count)` | bulk input |
| `virtual pos_type seekoff(off_type off, ios_base::seekdir dir, ios_base::openmode which)` | reposition by offset |
| `virtual pos_type seekpos(pos_type pos, ios_base::openmode which)` | reposition to absolute position |
| `virtual int sync()` | synchronize with underlying device |
| `virtual int_type pbackfail(int_type ch = Traits::eof())` | handle putback failure |

Public non-virtual members (`sgetc`, `sbumpc`, `sputc`, `sputn`, `in_avail`, `pubsync`, `pubseekoff`, `pubseekpos`) forward to the above virtuals and are what `basic_istream`/`basic_ostream` actually call.
