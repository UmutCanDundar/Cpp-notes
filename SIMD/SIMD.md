# SIMD Intrinsics (SSE / SSE2 / AVX / AVX2)

A categorized reference for x86 SIMD intrinsics: data types, load/store,
arithmetic, comparison, set, and move/mask operations — plus a naming-suffix
guide (`_ps`, `_pd`, `_epi32`, `_si128`, etc.) 
## Required headers

```c
#include <immintrin.h>   // Everything below (SSE -> AVX-512). Safe to just use this one.

// Or, if you want the narrowest header per feature set:
#include <xmmintrin.h>   // SSE    (__m128,  float ops)
#include <emmintrin.h>   // SSE2   (__m128d, __m128i, double/int ops)
#include <pmmintrin.h>   // SSE3
#include <tmmintrin.h>   // SSSE3
#include <smmintrin.h>   // SSE4.1
#include <nmmintrin.h>   // SSE4.2
#include <immintrin.h>   // AVX, AVX2, AVX-512, FMA
```

Compile flags: `-msse2`, `-mavx`, `-mavx2`, `-mfma`, `-mavx512f` (GCC/Clang) or
`/arch:AVX2` (MSVC).

---

## 1. Types & Register Widths

| Type      | Width  | Lanes                              | Feature set |
|-----------|--------|-------------------------------------|-------------|
| `__m128`  | 128-bit | 4 × float                          | SSE   |
| `__m128d` | 128-bit | 2 × double                         | SSE2  |
| `__m128i` | 128-bit | integer (4×i32, 8×i16, 16×i8, 2×i64) | SSE2  |
| `__m256`  | 256-bit | 8 × float                          | AVX   |
| `__m256d` | 256-bit | 4 × double                         | AVX   |
| `__m256i` | 256-bit | integer (8×i32, 16×i16, 32×i8, 4×i64) | AVX2  |
| `__m512`  | 512-bit | 16 × float                         | AVX-512 |
| `__m512d` | 512-bit | 8 × double                         | AVX-512 |
| `__m512i` | 512-bit | integer (16×i32, 32×i16, 64×i8, 8×i64) | AVX-512 |

```c
__m128  v0;   // 4 floats,  uninitialized
__m128d v1;   // 2 doubles, uninitialized
__m128i v2;   // 128-bit integer register, lane layout decided by the intrinsic you use on it
__m256  v3;   // 8 floats
__m256i v4;   // 256-bit integer register
```

> `__m128i` / `__m256i` / `__m512i` are "generic" integer registers — the
> intrinsic name (`_epi32`, `_epi16`, ...) tells you how the lanes are sliced,
> not the type itself.

---

## 2. Load / Store

| Intrinsic | Meaning | Alignment |
|---|---|---|
| `_mm_load_ps(p)`      | aligned load, 4×float   | 16-byte aligned (crashes if not) |
| `_mm_loadu_ps(p)`     | unaligned load, 4×float | none required |
| `_mm_store_ps(p,v)`   | aligned store           | 16-byte aligned |
| `_mm_storeu_ps(p,v)`  | unaligned store         | none required |
| `_mm_load_pd(p)` / `_mm_storeu_pd(p,v)` | same, for `double` | 16-byte |
| `_mm_load_si128(p)`   | aligned load, integer   | 16-byte |
| `_mm_loadu_si128(p)`  | unaligned load, integer | none |
| `_mm256_load_ps(p)` / `_mm256_load_pd(p)`   | aligned load (AVX)      | 32-byte |
| `_mm256_loadu_ps(p)` / `_mm256_loadu_pd(p)` | unaligned load (AVX)    | none |
| `_mm256_load_si256(p)`   | aligned load, integer (AVX2) | 32-byte |
| `_mm256_loadu_si256(p)`  | unaligned load, integer (AVX2) | none |
| `_mm_maskload_ps(p,mask)` / `_mm_maskload_epi32(p,mask)` | conditional load per-lane (mask) | none |
| `_mm_stream_ps(p,v)` / `_mm_stream_si128(p,v)` | non-temporal store (bypasses cache) | aligned |

```c
alignas(16) float buf[4] = {1,2,3,4};
__m128 va = _mm_load_ps(buf);            // buf must be 16-byte aligned

float ubuf[4] = {1,2,3,4};
__m128 vb = _mm_loadu_ps(ubuf);          // works regardless of alignment

_mm_store_ps(buf, va);                   // write va back to aligned buf
_mm_storeu_ps(ubuf, vb);                 // write vb back to unaligned ubuf

alignas(16) int ibuf[4] = {1,2,3,4};
__m128i vi = _mm_load_si128((__m128i*)ibuf);   // integer aligned load

alignas(32) float abuf[8] = {0,1,2,3,4,5,6,7};
__m256 va256 = _mm256_load_ps(abuf);     // AVX, needs 32-byte alignment

__m128i mask = _mm_set_epi32(-1, 0, -1, 0);          // high bit set = load, 0 = skip
__m128  vm   = _mm_maskload_ps(buf, mask);           // conditional load per lane
```

**Rule of thumb:** `load` = must be aligned (fast, can segfault) · `loadu` =
works on any address (slightly slower, always safe).

---

## 3. Arithmetic

| Intrinsic | Meaning |
|---|---|
| `_mm_add_ps` / `_mm_add_pd`     | packed add, float / double |
| `_mm_sub_ps` / `_mm_sub_pd`     | packed subtract |
| `_mm_mul_ps` / `_mm_mul_pd`     | packed multiply |
| `_mm_div_ps` / `_mm_div_pd`     | packed divide |
| `_mm_sqrt_ps` / `_mm_rsqrt_ps`  | square root / reciprocal sqrt (approx) |
| `_mm_min_ps` / `_mm_max_ps`     | packed min / max |
| `_mm_add_epi32` / `_epi16` / `_epi8` | packed integer add, per lane width |
| `_mm_sub_epi32`                 | packed integer subtract |
| `_mm_mullo_epi32`               | integer multiply, low bits kept |
| `_mm_mulhi_epi16`               | integer multiply, high bits kept |
| `_mm_abs_epi32` / `_epi16` / `_epi8`  | absolute value |
| `_mm_and_ps` / `_mm_or_ps` / `_mm_xor_ps` | bitwise ops (float regs) |
| `_mm_and_si128` / `_mm_or_si128` / `_mm_xor_si128` | bitwise ops (int regs) |
| `_mm_fmadd_ps` (FMA)            | fused multiply-add: `a*b + c` |
| `_mm256_add_ps` / `_mm256_mul_ps` | same ops, 256-bit AVX versions |

```c
__m128 a = _mm_set1_ps(2.0f), b = _mm_set1_ps(3.0f);
__m128 sum  = _mm_add_ps(a, b);      // {5,5,5,5}
__m128 diff = _mm_sub_ps(a, b);      // {-1,-1,-1,-1}
__m128 prod = _mm_mul_ps(a, b);      // {6,6,6,6}
__m128 quot = _mm_div_ps(a, b);      // {0.666...} x4
__m128 root = _mm_sqrt_ps(a);        // sqrt of each lane

__m128i ia = _mm_set1_epi32(10), ib = _mm_set1_epi32(3);
__m128i isum = _mm_add_epi32(ia, ib);    // {13,13,13,13}
__m128i imul = _mm_mullo_epi32(ia, ib);  // {30,30,30,30}, low 32 bits kept

__m128 c = _mm_set1_ps(4.0f);
__m128 fma = _mm_fmadd_ps(a, b, c);  // a*b + c = {10,10,10,10}  (needs -mfma)

__m256 a8 = _mm256_set1_ps(1.0f), b8 = _mm256_set1_ps(2.0f);
__m256 sum8 = _mm256_add_ps(a8, b8); // 8-wide version
```

---

## 4. Set / Broadcast / Init

| Intrinsic | Meaning |
|---|---|
| `_mm_set_ps(d,c,b,a)`   | set 4 floats, **highest arg = lane 0** (reverse order!) |
| `_mm_setr_ps(a,b,c,d)`  | set 4 floats, natural order |
| `_mm_set1_ps(x)`        | broadcast one value to all lanes |
| `_mm_set_epi32(...)`    | set 4 int32 lanes |
| `_mm_setzero_ps` / `_pd` / `_si128` | all-zero register |
| `_mm256_set1_epi32(x)`  | broadcast int32 to all 8 lanes (AVX2) |
| `_mm256_broadcast_ss(p)`| broadcast a scalar from memory (AVX) |
| `_mm_undefined_ps()`    | uninitialized register (perf hint, no init cost) |

```c
__m128 v1 = _mm_set_ps(4,3,2,1);     // lanes = {1,2,3,4} -> arg order is reversed!
__m128 v2 = _mm_setr_ps(1,2,3,4);    // lanes = {1,2,3,4} -> natural order, easier to read
__m128 v3 = _mm_set1_ps(7.0f);       // lanes = {7,7,7,7}
__m128i v4 = _mm_set_epi32(4,3,2,1); // int lanes = {1,2,3,4}, same reversed order rule
__m128  z1 = _mm_setzero_ps();       // {0,0,0,0}
__m128i z2 = _mm_setzero_si128();    // all-zero integer register

__m256i v5 = _mm256_set1_epi32(9);   // 8 lanes all = 9

float scalar = 3.14f;
__m256 v6 = _mm256_broadcast_ss(&scalar); // load scalar from memory, broadcast to 8 lanes
```

---

## 5. Compare

| Intrinsic | Meaning | Result |
|---|---|---|
| `_mm_cmpeq_ps` / `_pd`     | equal, per lane (float/double)      | all-1s or all-0s mask per lane |
| `_mm_cmplt_ps`           | less-than | same |
| `_mm_cmpgt_ps`           | greater-than | same |
| `_mm_cmple_ps` / `_mm_cmpge_ps` | less/greater-or-equal | same |
| `_mm_cmpeq_epi32` / `_epi16` / `_epi8` | integer equality per lane | 0/-1 mask |
| `_mm_cmpgt_epi32`        | integer greater-than per lane | 0/-1 mask |
| `_mm256_cmp_ps(a,b,imm)` | AVX generalized compare (imm8 selects predicate, e.g. `_CMP_EQ_OQ`) | mask |

```c
__m128 a = _mm_set1_ps(5.0f), b = _mm_set1_ps(3.0f);
__m128 eq = _mm_cmpeq_ps(a, b);   // all lanes 0 (false), since 5 != 3
__m128 gt = _mm_cmpgt_ps(a, b);   // all lanes 0xFFFFFFFF (true), since 5 > 3

__m128i ia = _mm_set1_epi32(5), ib = _mm_set1_epi32(5);
__m128i ieq = _mm_cmpeq_epi32(ia, ib);  // all lanes -1 (true)

__m256 a8 = _mm256_set1_ps(1.0f), b8 = _mm256_set1_ps(2.0f);
__m256 lt8 = _mm256_cmp_ps(a8, b8, _CMP_LT_OQ);  // AVX needs an imm8 predicate constant
```

> SSE/AVX2 **integer** compares (`cmpeq_epi*`, `cmpgt_epi*`) use dedicated
> instructions; **float** compares need the `imm8` predicate form once you're
> in AVX (`_mm256_cmp_ps`), since AVX dropped the separate float-compare
> opcodes SSE had.

---

## 6. Move / Mask

| Intrinsic | Meaning |
|---|---|
| `_mm_movemask_ps(v)`     | pack the sign bit of each float lane into an int (4 bits used) |
| `_mm_movemask_epi8(v)`   | pack the sign bit (MSB) of each byte lane into an int (16 bits used) |
| `_mm256_movemask_ps(v)`  | same, 8 lanes (AVX) |
| `_mm_move_ss` / `_mm_move_sd` | move only the lowest lane, keep rest of dest |
| `_mm_blendv_ps(a,b,mask)`| per-lane select: mask bit decides `a` or `b` |
| `_mm_blend_ps(a,b,imm)`  | per-lane select via compile-time immediate mask |
| `_mm_shuffle_ps(a,b,imm)`| arbitrary lane permutation/shuffle |
| `_mm_unpacklo_ps` / `_mm_unpackhi_ps` | interleave low/high halves of two registers |

```c
__m128 v = _mm_set_ps(-1.0f, 2.0f, -3.0f, 4.0f);
int bits = _mm_movemask_ps(v);       // sign bits packed into an int, e.g. 0b1010

__m128i bv = _mm_set_epi8(-1,0,-1,0, -1,0,-1,0, -1,0,-1,0, -1,0,-1,0);
int bbits = _mm_movemask_epi8(bv);   // 16-bit mask, one bit per byte lane

__m128 lo = _mm_set1_ps(1.0f), hi = _mm_set1_ps(2.0f);
__m128 moved = _mm_move_ss(lo, hi);  // lane0 = hi's lane0, lanes 1-3 stay from lo

__m128 x = _mm_set1_ps(1.0f), y = _mm_set1_ps(2.0f);
__m128 mask = _mm_cmpgt_ps(y, x);            // mask = all-true here
__m128 blended = _mm_blendv_ps(x, y, mask);  // picks y where mask bit is set

__m128 shuf = _mm_shuffle_ps(x, y, _MM_SHUFFLE(0,1,2,3)); // reorder lanes from x and y
```

---

## 7. Naming Suffix Guide (how to decode any intrinsic)

Every intrinsic reads as: `_mm<width>_<operation>_<type_suffix>`

| Suffix | Meaning |
|---|---|
| `_ps`   | **p**acked **s**ingle-precision float (32-bit float lanes) |
| `_pd`   | **p**acked **d**ouble-precision float (64-bit double lanes) |
| `_ss`   | **s**calar **s**ingle — operates only on lane 0, rest untouched |
| `_sd`   | **s**calar **d**ouble — operates only on lane 0 |
| `_epi8` / `_epi16` / `_epi32` / `_epi64` | **e**xtended **p**acked **i**nteger, signed, N-bit lanes |
| `_epu8` / `_epu16` / `_epu32` / `_epu64` | same, but **u**nsigned lanes |
| `_si128` / `_si256` / `_si512` | **s**igned **i**nteger, whole-register op (no defined lane width — used for bitwise ops like AND/OR/XOR that don't care about lane size) |
| `_pi8` / `_pi16` / `_pi32` (legacy MMX) | packed integer, 64-bit MMX register (rare/deprecated) |
| `m` prefix (`_mm`, `_mm256`, `_mm512`) | register width: 128 / 256 / 512-bit |

```c
__m128  a = _mm_add_ps(_mm_set1_ps(1), _mm_set1_ps(2));      // _ps  -> all 4 float lanes
__m128  b = _mm_add_ss(_mm_set1_ps(1), _mm_set1_ps(2));      // _ss  -> only lane0 changes, 1,1,1 kept in rest
__m128i c = _mm_add_epi32(_mm_set1_epi32(1), _mm_set1_epi32(2)); // _epi32 -> 4 signed int32 lanes
__m128i d = _mm_and_si128(_mm_set1_epi32(0xF), _mm_set1_epi32(0x3)); // _si128 -> whole-register bitwise, lane-agnostic
```

**Quick disambiguation — `_epi32` vs `_si128`:**
- `_epi32` → the op *cares* about lane width (add, compare, multiply per 32-bit lane).
- `_si128` → the op is lane-agnostic (bitwise AND/OR/XOR/ANDNOT), so it's just "signed integer register" as a whole, no lane split implied.

**Quick disambiguation — `_ps` vs `_pd` vs `_ss`/`_sd`:**
- `_ps`/`_pd` → whole register, all lanes.
- `_ss`/`_sd` → only lane 0 touched, other lanes pass through from the first operand unchanged.

---

## 8. Putting It Together — Minimal Example

```c
#include <immintrin.h>

void add_arrays(const float* a, const float* b, float* out, int n) {
    int i = 0;
    for (; i + 4 <= n; i += 4) {
        __m128 va = _mm_loadu_ps(&a[i]);
        __m128 vb = _mm_loadu_ps(&b[i]);
        __m128 vc = _mm_add_ps(va, vb);
        _mm_storeu_ps(&out[i], vc);
    }
    for (; i < n; i++) out[i] = a[i] + b[i]; // scalar tail
}
```

---

## Summary Table (one-line cheat sheet)

| Category  | SSE/SSE2 prefix | AVX/AVX2 prefix |
|---|---|---|
| Load/Store | `_mm_load(u)_ps/pd/si128` | `_mm256_load(u)_ps/pd/si256` |
| Arithmetic | `_mm_add/sub/mul/div_ps/pd/epiN` | `_mm256_add/sub/mul/div_ps/pd/epiN` |
| Set        | `_mm_set/set1/setzero_*` | `_mm256_set/set1/setzero_*` |
| Compare    | `_mm_cmpeq/cmpgt_ps/pd/epiN` | `_mm256_cmp_ps/pd` (imm8 predicate) |
| Move/Mask  | `_mm_movemask_ps/epi8`, `_mm_blendv_ps` | `_mm256_movemask_ps`, `_mm256_blendv_ps` |
