# mem* — Raw Memory Operations `<cstring>`

| Function | Signature | Description |
|----------|-----------|-------------|
| memcpy | `memcpy(dst, src, n)` | Copies n bytes. src and dst must not overlap. Fast for trivially copyable structs. |
| memmove | `memmove(dst, src, n)` | Copies n bytes. Safe for overlapping regions. Slightly slower than memcpy. |
| memset | `memset(ptr, val, n)` | Fills n bytes with val. val is int but written as a byte. Use memset(p,0,n) to zero out. |
| memcmp | `memcmp(a, b, n)` | Compares n bytes. 0 means equal, <0 means a is less, >0 means a is greater. Unlike strcmp: does not stop at null. |
| memchr | `memchr(ptr, val, n)` | Searches for byte val within the first n bytes. Returns void*, null if not found. |

**Rule:** If src and dst overlap, use memmove; if they don't, use memcpy. The compiler usually optimizes memcpy with SIMD.

## Safe Versions (C11 / MSVC)

| Function | Signature | Description |
|----------|-----------|-------------|
| memcpy_s | `memcpy_s(dst, dstsz, src, n)` | Also takes dst size to prevent overflow. Not available by default on Linux, available in MSVC. |
