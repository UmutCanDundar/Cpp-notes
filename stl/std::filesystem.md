# `<filesystem>` (C++17)

| API | Priority | Description |
|-----|----------|-------------|
| `std::filesystem::path` | memorize | Cross-platform representation of a file path, handles separator differences automatically. |
| `std::filesystem::exists(path)` | know | Checks whether a file or directory exists. |
| `std::filesystem::create_directory` / `create_directories` | know | Creates a directory (single level) or full nested path. |
| `std::filesystem::remove` / `remove_all` | know | Deletes a file, or a directory and its contents recursively. |
| `std::filesystem::copy` / `rename` | know | Copies or moves/renames files and directories. |
| `std::filesystem::file_size(path)` | know | Returns file size in bytes without opening/reading the file. |
| `std::filesystem::directory_iterator` | know | Iterates over entries in a directory, usable in a range-based for loop. |
| `std::filesystem::recursive_directory_iterator` | know | Like directory_iterator, but descends into subdirectories too. |
| Filesystem ops on hot path | avoid | All filesystem calls hit the OS/syscall layer — never call these per-tick/per-message in latency-sensitive code; do I/O setup once at startup. |
