## Instructions

### Data Movement

| Instruction | Örnek | src | dst | Açıklama |
|-------------|-------|-----|-----|----------|
| `mov` | `mov rax, rbx` | rbx | rax | rbx'i rax'a kopyala. En sık kullanılan instruction. |
| `mov` | `mov rax, [rbp-8]` | memory | rax | `rbp-8` adresinden 8 byte oku → rax. |
| `mov` | `mov [rbp-8], rax` | rax | memory | rax'ı `rbp-8` adresine yaz. |
| `mov` | `mov rax, 42` | immediate | rax | Sabit değer yaz. |
| `movzx` | `movzx rax, bl` | bl (8-bit) | rax (64-bit) | Küçük → büyük register. Üst bitler **sıfırlanır**. `bl=0xFF` → `rax=0x00...FF`. |
| `movsx` | `movsx rax, ecx` | ecx (32-bit) | rax (64-bit) | Sign-extend. Negatif sayı korunur. `ecx=-1` → `rax=-1` (tüm bitler 1). |
| `movaps` | `movaps xmm0, [ptr]` | memory | xmm0 | 128-bit aligned load. ptr 16-byte aligned olmalı. |
| `movups` | `movups xmm0, [ptr]` | memory | xmm0 | 128-bit unaligned load. |
| `vmovaps` | `vmovaps ymm0, [ptr]` | memory | ymm0 | 256-bit aligned load (AVX). |
| `vmovdqu` | `vmovdqu ymm0, [ptr]` | memory | ymm0 | 256-bit unaligned load (AVX). |
| `lea` | `lea rax, [rbx + rcx*4]` | address expr | rax | Adresi **hesapla**, memory'e erişme. `rax = rbx + rcx*4`. Hızlı multiply/add trick. |
| `lea` | `lea rax, [rax + 5]` | — | rax | `add rax, 5` yerine — flag set etmez. |
| `push` | `push rax` | rax | stack | `rsp -= 8`, sonra `[rsp] = rax`. |
| `pop` | `pop rax` | stack | rax | `rax = [rsp]`, sonra `rsp += 8`. |
| `xchg` | `xchg rax, rbx` | rax↔rbx | — | İki register'ı swap et. Memory operand ile **implicit LOCK** — atomic. |
| `rep movsb` | `rep movsb` | rsi | rdi | `rcx` byte'ı rsi'den rdi'ye kopyala. Modern CPU'da `memcpy` optimize edilmiş hali. |

### Arithmetic

| Instruction | Örnek | Açıklama |
|-------------|-------|----------|
| `add` | `add rax, rbx` | `rax = rax + rbx`. Flags set edilir (ZF, CF, OF, SF). |
| `add` | `add rax, 5` | `rax = rax + 5`. Immediate. |
| `sub` | `sub rax, rbx` | `rax = rax - rbx`. |
| `sub` | `sub rsp, 24` | Stack'te 24 byte yer aç (3 local variable için). |
| `inc` | `inc rax` | `rax++`. CF set etmez — dikkat. |
| `dec` | `dec rax` | `rax--`. |
| `imul` | `imul rax, rbx` | `rax = rax * rbx`. Signed. |
| `imul` | `imul rax, rbx, 5` | `rax = rbx * 5`. 3 operand. |
| `idiv` | `idiv rcx` | `rax = rdx:rax / rcx`. Quotient→rax, Remainder→rdx. Önce `cqo` ile rdx sign-extend et. |
| `mul` | `mul rbx` | `rdx:rax = rax * rbx`. Unsigned. |
| `neg` | `neg rax` | `rax = -rax`. Two's complement negate. |
| `cqo` | `cqo` | `rax`'ı `rdx:rax`'a sign-extend et. `idiv` öncesi kullanılır. |

### Bitwise & Shift

| Instruction | Örnek | Açıklama |
|-------------|-------|----------|
| `and` | `and rax, rbx` | `rax = rax & rbx`. Bitwise AND. |
| `and` | `and rax, 0xFF` | Masking — sadece alt 8 bit kalsın. |
| `or` | `or rax, rbx` | `rax = rax \| rbx`. |
| `xor` | `xor rax, rax` | `rax = 0`. Register sıfırlamanın standart yolu. `mov rax, 0`'dan hızlı. |
| `xor` | `xor rax, rbx` | `rax = rax ^ rbx`. |
| `not` | `not rax` | `rax = ~rax`. Tüm bitleri çevir. |
| `shl` | `shl rax, 3` | `rax <<= 3` → `rax *= 8`. Logical shift left. |
| `shr` | `shr rax, 1` | `rax >>= 1`. Logical shift right. Üst bit **0** ile dolar. |
| `sar` | `sar rax, 1` | Arithmetic shift right. Üst bit **sign bit** ile dolar. Negatif sayı korunur. |
| `rol` | `rol rax, 1` | Rotate left. Çıkan bit diğer uca girer. |
| `ror` | `ror rax, 1` | Rotate right. |
| `bsr` | `bsr rax, rbx` | Bit scan reverse. En yüksek set bit'in indexi → rax. |
| `bsf` | `bsf rax, rbx` | Bit scan forward. En düşük set bit'in indexi → rax. |
| `popcnt` | `popcnt rax, rbx` | rbx'teki 1 bitlerin sayısı → rax. |
| `tzcnt` | `tzcnt rax, rbx` | Trailing zero count. En düşük 1 bitten önce kaç 0 var. |

### Comparison & Test

| Instruction | Örnek | Açıklama |
|-------------|-------|----------|
| `cmp` | `cmp rax, rbx` | `rax - rbx` hesapla, sonucu at, sadece flags set et. |
| `cmp` | `cmp rax, 0` | rax == 0 mu? ZF set. |
| `test` | `test rax, rax` | `rax & rax` hesapla, sonucu at. rax==0 ise ZF=1. `cmp rax,0`'dan 1 byte kısa. |
| `test` | `test rax, 1` | rax'ın bit 0'ı set mi? (odd/even check) |
| `test` | `test al, al` | sadece alt 8 bit kontrol et. |

### Conditional Set (SETcc) — flag'e göre 1 byte yaz

| Instruction | Koşul | Örnek |
|-------------|-------|-------|
| `sete` / `setz` | ZF=1 (equal / zero) | `sete al` |
| `setne` / `setnz` | ZF=0 | `setne al` |
| `setl` / `setnge` | SF≠OF (signed less) | `setl al` → `al = (a < b) ? 1 : 0` |
| `setg` / `setnle` | ZF=0 && SF=OF (signed greater) | `setg al` |
| `setle` / `setng` | ZF=1 \|\| SF≠OF | `setle al` |
| `setge` / `setnl` | SF=OF | `setge al` |
| `setb` / `setc` | CF=1 (unsigned below) | `setb al` |
| `seta` | CF=0 && ZF=0 (unsigned above) | `seta al` |
| `sets` | SF=1 (negative) | `sets al` |
| `setns` | SF=0 | `setns al` |
| `seto` | OF=1 (overflow) | `seto al` |

> `setl al` → al = 1 byte. `movzx rax, al` ile tam register'a taşı.

### Conditional Move (CMOVcc) — branchless if

| Instruction | Koşul | Örnek |
|-------------|-------|-------|
| `cmove` | ZF=1 | `cmove rax, rbx` → rax = rbx eğer equal |
| `cmovne` | ZF=0 | |
| `cmovl` | signed less | `cmovl rax, rbx` → `if (a<b) rax=rbx` |
| `cmovg` | signed greater | |
| `cmovle` | signed ≤ | |
| `cmovge` | signed ≥ | |
| `cmovb` | unsigned below | |
| `cmova` | unsigned above | |

> Branch predictor miss yok. `std::min/max` compiler tarafından cmov'a çevrilir.

### Jumps

| Instruction | Koşul | Açıklama |
|-------------|-------|----------|
| `jmp` | — | Unconditional. `jmp label` veya `jmp rax` (indirect). |
| `je` / `jz` | ZF=1 | Equal / Zero. |
| `jne` / `jnz` | ZF=0 | Not equal. |
| `jl` / `jnge` | SF≠OF | Signed less than. |
| `jg` / `jnle` | ZF=0 && SF=OF | Signed greater than. |
| `jle` / `jng` | ZF=1 \|\| SF≠OF | Signed less or equal. |
| `jge` / `jnl` | SF=OF | Signed greater or equal. |
| `jb` / `jc` | CF=1 | Unsigned below. |
| `ja` | CF=0 && ZF=0 | Unsigned above. |
| `js` | SF=1 | Negative. |
| `jns` | SF=0 | Non-negative. |
| `jo` | OF=1 | Overflow. |
| `jz` | ZF=1 | Zero (je ile aynı). |
| `jnz` | ZF=0 | Non-zero. |
| `loop` | rcx-- != 0 | rcx'i azalt, sıfır değilse jump. |

### Call & Return

| Instruction | Açıklama |
|-------------|----------|
| `call func` | `rsp -= 8`, `[rsp] = rip+1` (return addr), `jmp func`. |
| `ret` | `rip = [rsp]`, `rsp += 8`. |
| `ret 8` | Return + stack'ten N byte temizle (Windows calling convention). |
| `syscall` | Kernel'a geç. `rax` = syscall number. Argümanlar: rdi,rsi,rdx,r10,r8,r9. |

### Memory Addressing Modes

```
[base]                    mov rax, [rbx]           ; *rbx
[base + offset]           mov rax, [rbx + 8]       ; *(rbx+8)
[base + index*scale]      mov rax, [rbx + rcx*4]   ; rbx[rcx]  (int array)
[base + index*scale + d]  mov rax, [rbx + rcx*8 + 16]
[rip + offset]            mov rax, [rip + label]   ; RIP-relative (global var)
```

Scale sadece 1, 2, 4, 8 olabilir — sizeof(type) için.

### Memory Size Specifiers

```asm
BYTE  PTR [rbp-1]   ; 1 byte  (uint8_t, char)
WORD  PTR [rbp-2]   ; 2 byte  (uint16_t, short)
DWORD PTR [rbp-4]   ; 4 byte  (uint32_t, int, float)
QWORD PTR [rbp-8]   ; 8 byte  (uint64_t, double, pointer)
XMMWORD PTR [rsi]   ; 16 byte (xmm register)
YMMWORD PTR [rsi]   ; 32 byte (ymm register)
```

### Atomic & Synchronization

| Instruction | Açıklama |
|-------------|----------|
| `lock add [ptr], 1` | Atomic increment. Bus lock — başka CPU erişemez. |
| `lock cmpxchg [ptr], rbx` | Compare-and-swap. `rax` = expected, `rbx` = new. Equal ise swap, değilse `rax` = current. `std::atomic CAS` karşılığı. |
| `lock xchg rax, [ptr]` | Atomic swap. Implicit LOCK — prefix gereksiz. |
| `mfence` | Full memory barrier. Store ve load reorder edilemez. |
| `sfence` | Store barrier. Önceki store'lar görünür olmadan geçilmez. |
| `lfence` | Load barrier. Önceki load'lar tamamlanmadan geçilmez. |
| `pause` | Spin-wait hint. Pipeline'ı boşaltır, power azaltır. `_mm_pause()` karşılığı. |

### SIMD

| Instruction | Açıklama |
|-------------|----------|
| `vzeroupper` | Tüm `ymm[255:128]` bitlerini sıfırla. AVX→SSE geçişinde penalty önler. |
| `vmovaps ymm0, [ptr]` | 256-bit aligned load. ptr 32-byte aligned olmalı. |
| `vmovdqu ymm0, [ptr]` | 256-bit unaligned load. |
| `vmulps ymm0, ymm1, ymm2` | 8x float multiply. `ymm0 = ymm1 * ymm2`. |
| `vaddps ymm0, ymm1, ymm2` | 8x float add. |
| `vpand ymm0, ymm1, ymm2` | 256-bit bitwise AND. |
| `vpcmpeqd ymm0, ymm1, ymm2` | 8x int32 compare equal. Match → 0xFFFFFFFF, no match → 0. |

### Misc

| Instruction | Açıklama |
|-------------|----------|
| `nop` | Hiçbir şey yapma. Alignment için veya timing pad. |
| `nop DWORD PTR [rax]` | Multi-byte nop. Alignment gap doldurmak için. |
| `ud2` | Undefined instruction — kasıtlı crash. `__builtin_unreachable()` bazen buna çevrilir. |
| `int3` | Breakpoint. Debugger trap. |
| `cpuid` | CPU özelliklerini sorgula. `rax` = leaf. |
| `rdtsc` | Timestamp counter oku → `rdx:rax`. Cycle-accurate timer. |
| `rdtscp` | `rdtsc` + CPU ID. Out-of-order execution'ı önler. |

---

## Stack Frame — Nasıl Çalışır

```
void foo(int a, int b) {
    int x = a + b;
    int y = x * 2;
}
```

```asm
foo:
    push   rbp              ; caller'ın rbp'sini kaydet
    mov    rbp, rsp         ; yeni frame base kur
    sub    rsp, 16          ; 2x int için yer aç (16-byte aligned)
    
    mov    DWORD PTR [rbp-4],  edi   ; a = rdi (1. arg)
    mov    DWORD PTR [rbp-8],  esi   ; b = rsi (2. arg)
    
    mov    eax, DWORD PTR [rbp-4]    ; eax = a
    add    eax, DWORD PTR [rbp-8]    ; eax = a + b
    mov    DWORD PTR [rbp-12], eax   ; x = eax
    
    mov    eax, DWORD PTR [rbp-12]   ; eax = x
    add    eax, eax                   ; eax = x * 2  (add yerine shl 1 de olur)
    mov    DWORD PTR [rbp-16], eax   ; y = eax
    
    leave                   ; mov rsp,rbp + pop rbp
    ret

Stack layout (rbp = 0x7fff1000):
┌──────────────┬───────────────────┐
│ Address      │ İçerik            │
├──────────────┼───────────────────┤
│ 0x7fff1008   │ return address    │
│ 0x7fff1000   │ old rbp  ← rbp   │
│ 0x7ffe0ffc   │ a (int)  rbp-4   │
│ 0x7ffe0ff8   │ b (int)  rbp-8   │
│ 0x7ffe0ff4   │ x (int)  rbp-12  │
│ 0x7ffe0ff0   │ y (int)  rbp-16  │ ← rsp
└──────────────┴───────────────────┘
```

---

## objdump Çıktısı — Tam Analiz

### Örnek C++ kodu

```cpp
int add(int a, int b) {
    return a + b;
}

int main() {
    int x = add(3, 4);
    return x;
}
```

### Derleme ve objdump

```bash
g++ -O0 -g -o example example.cpp          # debug, optimizasyon yok
objdump -d -M intel --no-show-raw-insn example   # Intel syntax, temiz
# veya daha detaylı:
objdump -d -M intel -S example             # C++ kaynak ile interleaved
```

### objdump Çıktısı

```
0000000000001129 <_Z3addii>:                          ← (1)
    1129:   push   rbp                                ← (2)
    112a:   mov    rbp,rsp
    112d:   mov    DWORD PTR [rbp-0x4],edi            ← (3)
    1130:   mov    DWORD PTR [rbp-0x8],esi
    1133:   mov    edx,DWORD PTR [rbp-0x4]
    1136:   mov    eax,DWORD PTR [rbp-0x8]
    1139:   add    eax,edx                            ← (4)
    113b:   pop    rbp
    113c:   ret

000000000000113d <main>:                              ← (5)
    113d:   push   rbp
    113e:   mov    rbp,rsp
    1141:   sub    rsp,0x10                           ← (6)
    1145:   mov    esi,0x4                            ← (7)
    114a:   mov    edi,0x3
    114f:   call   1129 <_Z3addii>                   ← (8)
    1154:   mov    DWORD PTR [rbp-0x4],eax            ← (9)
    1157:   mov    eax,DWORD PTR [rbp-0x4]
    115a:   leave
    115b:   ret
```

### Sütunların Anlamı

```
0000000000001129  <_Z3addii>:
^^^^^^^^^^^^^^^^^  ^^^^^^^^^^
      (A)              (B)

    1129:   push   rbp
    ^^^^    ^^^^^^^^^^^^
     (C)       (D)
```

| Kolon | Açıklama |
|-------|----------|
| **(A)** Fonksiyon adresi | Binary'deki virtual address. Link sonrası gerçek adres. `0x1129` = fonksiyonun başladığı offset. |
| **(B)** Sembol adı | `_Z3addii` = mangled name. `c++filt _Z3addii` → `add(int, int)`. |
| **(C)** Instruction adresi | O instruction'ın adresi. `1129`, `112a`, `112d`... Her instruction 1-15 byte. |
| **(D)** Assembly instruction | Intel syntax. `mov dst, src` sırası. |

### Satır Satır Açıklama

```
1129:  push rbp
```
> Caller'ın frame pointer'ını stack'e kaydet. Her fonksiyon başlangıcı.

```
112a:  mov rbp, rsp
```
> Yeni frame base'i kur. Artık `rbp` bu fonksiyonun stack tabanı.

```
112d:  mov DWORD PTR [rbp-0x4], edi
```
> 1. argüman `edi` (a=3) → stack'e yaz. `DWORD PTR` = 4 byte (int). `[rbp-4]` = ilk local slot.

```
1130:  mov DWORD PTR [rbp-0x8], esi
```
> 2. argüman `esi` (b=4) → stack'e yaz. `[rbp-8]` = ikinci slot.

```
1133:  mov edx, DWORD PTR [rbp-0x4]
1136:  mov eax, DWORD PTR [rbp-0x8]
```
> Stack'ten a ve b'yi register'lara geri yükle. (`-O0` yüzünden gereksiz roundtrip var.)

```
1139:  add eax, edx
```
> `eax = eax + edx` = `a + b`. Sonuç `eax`'ta = return value (`rax`).

```
113b:  pop rbp
113c:  ret
```
> Frame kapat, return.

```
1141:  sub rsp, 0x10
```
> Stack'te 16 byte yer aç. `x` için 4 byte lazım ama alignment 16-byte olmalı.

```
1145:  mov esi, 0x4
114a:  mov edi, 0x3
```
> `add(3, 4)` çağrısı için argümanları hazırla. 1.arg=rdi=3, 2.arg=rsi=4.

```
114f:  call 1129 <_Z3addii>
```
> Return address'i stack'e push et, `0x1129`'a jump.

```
1154:  mov DWORD PTR [rbp-0x4], eax
```
> Fonksiyonun return value'su `eax`'ta. `x = add(3, 4)` → stack'e kaydet.

### -O2 ile aynı kod — optimized

```bash
g++ -O2 -o example_opt example.cpp
objdump -d -M intel example_opt
```

```
0000000000001110 <_Z3addii>:
    1110:   lea    eax,[rdi+rsi]    ← (1) tek instruction, stack yok
    1113:   ret

0000000000001120 <main>:
    1120:   mov    eax,0x7           ← (2) compile-time 3+4=7 hesaplandı
    1125:   ret                          fonksiyon çağrısı bile yok
```

> **(1)** `lea eax, [rdi+rsi]` → `eax = rdi + rsi`. Stack frame yok, push/pop yok.
> **(2)** `3+4=7` compile-time constant fold edildi. `add` fonksiyonu inline edildi, `call` bile yok.

### Yaygın Patterns

```asm
; NULL check
test   rax, rax
je     .null_handler         ; rax == 0 ise jump

; i++ (loop counter)
add    DWORD PTR [rbp-4], 1

; array index: arr[i] where arr=int*, i=rcx
mov    eax, DWORD PTR [rax + rcx*4]
;                              ^^^^ sizeof(int)=4

; pointer dereference: *ptr
mov    rax, QWORD PTR [rbp-8]   ; ptr değerini al (adres)
mov    eax, DWORD PTR [rax]     ; o adresteki int'i oku

; branch → cmov (branchless min)
cmp    edi, esi
cmovle eax, edi                  ; if (a <= b) return a, else return b
```

---

## Hızlı Referans — Sık Karşılaşılanlar

| Gördüğünde | Anlamı |
|-----------|--------|
| `sub rsp, N` | N byte local variable için yer aç |
| `mov [rbp-N], reg` | Local variable'a yaz |
| `mov reg, [rbp-N]` | Local variable'ı oku |
| `xor eax, eax` | rax = 0 (return 0) |
| `test rax, rax` + `je` | if (ptr == null) |
| `lea rax, [rdi+rsi*4]` | pointer arithmetic, `arr[i]` adresi |
| `call` + `mov [rbp-N], eax` | fonksiyon çağır, sonucu sakla |
| `movzx eax, byte/word` | küçük tipten büyüğe, sıfırla |
| `movsx rax, dword` | int → long, sign-extend |
| `rep movsb` | memcpy |
| `lock cmpxchg` | atomic CAS |
| `vmulps ymm0, ymm1, ymm2` | 8x float çarp (SIMD) |
| `vzeroupper` | AVX bitti, SSE'ye dönüyor |
