## Registers

### General Purpose Registers

| 64-bit | 32-bit | 16-bit | 8-bit high | 8-bit low | Role |
|--------|--------|--------|------------|-----------|------|
| `rax` | `eax` | `ax` | `ah` | `al` | Return value. Division result (quotient). `rax = rax / rcx` sonrası quotient burada. |
| `rbx` | `ebx` | `bx` | `bh` | `bl` | Callee-saved. Fonksiyon çağrısından önce ne ise çağrı sonrasında da o olması garantili. |
| `rcx` | `ecx` | `cx` | `ch` | `cl` | 4. argüman. `rep` ile kullanılan loop counter. `cl` shift miktarı için kullanılır (`shl rax, cl`). |
| `rdx` | `edx` | `dx` | `dh` | `dl` | 3. argüman. Division'da high half (`rdx:rax` pair). `idiv` öncesi `rdx` sıfırlanmalı. |
| `rsi` | `esi` | `si` | — | `sil` | 2. argüman. `rep movsb` için source index. |
| `rdi` | `edi` | `di` | — | `dil` | 1. argüman. `rep movsb` için destination index. |
| `rsp` | `esp` | `sp` | — | `spl` | Stack pointer. Her zaman geçerli stack top'ını gösterir. `push/pop` otomatik günceller. Direkt kullanma. |
| `rbp` | `ebp` | `bp` | — | `bpl` | Frame pointer. Stack frame tabanını gösterir. Local variable'lar `rbp-N` ile adreslenir. Callee-saved. |
| `r8` | `r8d` | `r8w` | — | `r8b` | 5. argüman. Caller-saved. |
| `r9` | `r9d` | `r9w` | — | `r9b` | 6. argüman. Caller-saved. |
| `r10` | `r10d` | `r10w` | — | `r10b` | Caller-saved. Genel amaç. |
| `r11` | `r11d` | `r11w` | — | `r11b` | Caller-saved. Genel amaç. |
| `r12` | `r12d` | `r12w` | — | `r12b` | Callee-saved. Genel amaç. |
| `r13` | `r13d` | `r13w` | — | `r13b` | Callee-saved. Genel amaç. |
| `r14` | `r14d` | `r14w` | — | `r14b` | Callee-saved. Genel amaç. |
| `r15` | `r15d` | `r15w` | — | `r15b` | Callee-saved. Genel amaç. |

> **Caller-saved:** Fonksiyon çağırmadan önce sen kaydetmelisin — callee ezebilir.
> **Callee-saved:** Fonksiyon içinde kullanacaksan sen kaydetmelisin — çağrı sonrası restore et.

### Special Registers

| Register | Bit | Açıklama |
|----------|-----|----------|
| `rip` | 64 | Instruction pointer. Şu an execute edilen instruction'ın adresi. Direkt yazılamaz — `jmp/call` ile değişir. |
| `rflags` | 64 | Flag register. Aritmetik sonuçlardan otomatik set edilir. |

### RFLAGS — Önemli Flaglar

| Flag | Bit | Set koşulu | Kullanım |
|------|-----|------------|----------|
| `ZF` | 6 | Sonuç == 0 | `je`, `jne`, `test reg,reg` |
| `SF` | 7 | Sonuç < 0 (sign bit 1) | `js`, `jns` |
| `CF` | 0 | Unsigned overflow / borrow | `jb`, `jae`, `jc` |
| `OF` | 11 | Signed overflow | `jo`, `jno` |
| `PF` | 2 | Düşük 8 bitin parity'si çift | Nadir kullanılır |
| `AF` | 4 | BCD aritmetik carry | BCD dışında kullanılmaz |
| `DF` | 10 | Direction flag. 0=forward, 1=backward | `rep movsb` yönü |

### SIMD Registers

```
zmm0 [511                                                              0]
      |                                                                |
ymm0 [255                            0]
      |                              |
xmm0 [127              0]
```

| Register | Bit | ISA | Açıklama |
|----------|-----|-----|----------|
| `xmm0–xmm15` | 128 | SSE/SSE2 | Float/double argümanlar buradan geçer (`xmm0–xmm7`). Return da `xmm0`. |
| `ymm0–ymm15` | 256 | AVX/AVX2 | `xmm`'in upper 128 biti eklenmesi. `xmm0` = `ymm0[127:0]`. |
| `zmm0–zmm31` | 512 | AVX-512 | `ymm0–ymm15`'i kapsar. |

> `xmm0` ve `ymm0` aynı fiziksel register — farklı genişlikte view.
> SSE instruction'ı `xmm` kullanır → `ymm`'nin upper [255:128] dirty kalır → `vzeroupper` gerekir.

### Calling Convention (Linux — System V AMD64)

```
Integer argümanlar : rdi, rsi, rdx, rcx, r8, r9  (sonrası stack)
Float argümanlar   : xmm0–xmm7
Return (integer)   : rax  (128-bit: rdx:rax)
Return (float)     : xmm0
Stack alignment    : call öncesi rsp 16-byte aligned olmalı
```

> **Linux calling convention (System V AMD64):** Arguments in order: rdi, rsi, rdx, rcx, r8, r9. Rest on stack. Return in rax. Floats in xmm0–xmm7.
