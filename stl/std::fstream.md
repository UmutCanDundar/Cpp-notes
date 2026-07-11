# `<fstream>`

| API | Priority | Description |
|-----|----------|-------------|
| `std::ifstream` | memorize | Input file stream, for reading files. |
| `std::ofstream` | memorize | Output file stream, for writing files. |
| `std::fstream` | know | Bidirectional file stream (read + write). |
| `.open(path, mode)` / constructor with path | memorize | Opens a file. Check success with `if (!file)` or `.is_open()`. |
| `std::ios::binary` | memorize | Open mode flag — disables text-mode newline translation, required for raw/binary data. |
| `std::ios::app` / `std::ios::trunc` | know | Append to end of file / truncate (clear) existing content on open. |
| `.close()` | know | Closes the file explicitly; also happens automatically via RAII on destruction. |
| `<<` / `>>` on file streams | careful | Formatted, allocation-heavy, locale-aware — slow for high-throughput logging; prefer `.write()`/buffered binary I/O on hot paths. |
| `.read(buf, n)` / `.write(buf, n)` | memorize | Unformatted binary I/O — fastest way to move raw bytes to/from a file. |
| `.rdbuf()` | know | Access the underlying stream buffer, e.g. to redirect `std::cout` to a file. |
