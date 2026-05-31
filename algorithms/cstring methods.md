# str* — Null-Terminated C String Functions `<cstring>`

| Function | Signature | Description |
|----------|-----------|-------------|
| strlen | `strlen(s)` | Counts up to null. Null not included. O(n). |
| strcpy (unsafe) | `strcpy(dst, src)` | Copies src to dst. Buffer overflow risk — no size check. |
| strncpy | `strncpy(dst, src, n)` | Copies at most n characters. Does NOT append null if n < strlen(src) — be careful. |
| strcat (unsafe) | `strcat(dst, src)` | Appends src to end of dst. No size check. |
| strncat | `strncat(dst, src, n)` | Appends at most n characters, appends null. Safer than strcat. |
| strcmp | `strcmp(a, b)` | Lexicographic comparison. 0 = equal, <0 = a less, >0 = a greater. |
| strncmp | `strncmp(a, b, n)` | Compares first n characters. |
| strchr | `strchr(s, c)` | Searches for character c in s. Returns pointer, null if not found. |
| strrchr | `strrchr(s, c)` | Searches from the end. |
| strstr | `strstr(haystack, needle)` | Searches for substring. Returns pointer. |
| strtok (unsafe) | `strtok(s, delim)` | Splits string into tokens. Holds static state — thread-unsafe. Use strtok_r. |
| strtol / strtod | `strtol(s, &end, base)` | String → number. End pointer shows where parse stopped. More controlled than stoi. |
| sprintf (unsafe) | `sprintf(buf, "%d", x)` | Write formatted output to buffer. Use snprintf instead. |
| snprintf (preferred) | `snprintf(buf, n, "%d", x)` | Writes at most n characters. Safe. Return value is the length that would have been written. |

> In modern C++ code, use `std::string` or `std::string_view` instead of str* functions. Use `c_str()` when passing to C APIs.
