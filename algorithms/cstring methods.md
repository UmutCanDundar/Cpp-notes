# str* — Null-Terminated C String Functions `<cstring>`
| Function | Signature | Description | Notes |
|----------|-----------|-------------|-------|
| `strlen` | `strlen(s)` | Counts up to null. | SIMD optimized on modern CPUs. Avoid in hot path on large strings — cache miss risk. Cache result if called repeatedly. |
| `strcpy` | `strcpy(dst, src)` | Copies src to dst. | Unsafe. Use `memcpy` if size known — explicit size enables better SIMD. |
| `strncpy` | `strncpy(dst, src, n)` | Copies at most n chars. | Size known → more predictable. Does NOT append null if n < strlen(src). |
| `strcat` | `strcat(dst, src)` | Appends src to dst. | Unsafe. Calls strlen(dst) internally — O(n²) trap in loops. Never use in hot path. |
| `strncat` | `strncat(dst, src, n)` | Appends at most n chars. | Still calls strlen(dst) internally. Same loop trap as strcat. |
| `strcmp` | `strcmp(a, b)` | Lexicographic comparison. | SIMD optimized. Early exit on mismatch — short strings very fast. |
| `strncmp` | `strncmp(a, b, n)` | Compares first n chars. | Prefer over strcmp when length known. More predictable branch behavior. |
| `strchr` | `strchr(s, c)` | Searches for char c. | Linear scan. Fine for small strings. Avoid on large strings in hot path. |
| `strrchr` | `strrchr(s, c)` | Searches from the end. | Must scan full string to find last occurrence. More expensive than strchr. |
| `strstr` | `strstr(haystack, needle)` | Searches for substring. | Naive O(n*m). Platform-dependent optimization. Avoid in hot path for large inputs. |
| `strtok` | `strtok(s, delim)` | Splits into tokens. | Static internal state — thread-unsafe. Use `strtok_r` in multithreaded code. |
| `strtol/strtod` | `strtol(s, &end, base)` | String → number. | More controlled than `stoi`. End pointer shows where parse stopped. Error handling explicit. |
| `sprintf` | `sprintf(buf, "%d", x)` | Formatted output to buffer. | Unsafe — no size check. Never use. |
| `snprintf` | `snprintf(buf, n, "%d", x)` | Formatted output, at most n chars. | Safe. Return value = length that would have been written — use to detect truncation. |

> In modern C++ code, use `std::string` or `std::string_view` instead of str* functions. Use `c_str()` when passing to C APIs.
