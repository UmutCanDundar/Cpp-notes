# `<sstream>`

| API | Priority | Description |
|-----|----------|-------------|
| `std::istringstream` | know | Parses formatted values out of a string (e.g. tokenizing "1 2 3" into ints). |
| `std::ostringstream` | avoid | Builds a string via `<<` operators. Allocation-heavy and slow — prefer `std::format`/`to_chars` on hot paths. |
| `std::stringstream` | avoid | Bidirectional version. Same perf caveat, only use for convenience code/tests, not hot loops. |
| `.str()` | know | Get/set the underlying string content of a stringstream. |
| `>> std::ws` | know | Skips leading whitespace when reading from a stream — useful in parsing loops. |
