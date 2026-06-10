## Registers

### General Purpose Registers

| 64-bit | 32-bit | 16-bit | 8-bit high | 8-bit low | Role |
|--------|--------|--------|------------|-----------|------|
| `rax` | `eax` | `ax` | `ah` | `al` | Return value. Division quotient. After `idiv rcx`: quotient → rax, remainder → rdx. |
| `rbx` | `ebx` | `bx` | `bh` | `bl` | Callee-saved. Guaranteed to hold its value across a function call. |
| `rcx` | `ecx` | `cx` | `ch` | `cl` | 4th argument. Loop counter for `rep` instructions. `cl` = shift amount for `shl rax, cl`. |
| `rdx` | `edx` | `dx` | `dh` | `dl` | 3rd argument. High half of division (`rdx:rax` pair). Must be zeroed/sign-extended before `idiv`. |
| `rsi` | `esi` | `si` | — | `sil` | 2nd argument. Source index for `rep movsb`. |
| `rdi` | `edi` | `di` | — | `dil` | 1st argument. Destination index for `rep movsb`. |
| `rsp` | `esp` | `sp` | — | `spl` | Stack pointer. Always points to the current top of stack. Updated automatically by `push`/`pop`. Do not use directly. |
| `rbp` | `ebp` | `bp` | — | `bpl` | Frame pointer. Points to the base of the current stack frame. Local variables addressed as `rbp-N`. Callee-saved. |
| `r8` | `r8d` | `r8w` | — | `r8b` | 5th argument. Caller-saved. |
| `r9` | `r9d` | `r9w` | — | `r9b` | 6th argument. Caller-saved. |
| `r10` | `r10d` | `r10w` | — | `r10b` | Caller-saved. General purpose. |
| `r11` | `r11d` | `r11w` | — | `r11b` | Caller-saved. General purpose. |
| `r12` | `r12d` | `r12w` | — | `r12b` | Callee-saved. General purpose. |
| `r13` | `r13d` | `r13w` | — | `r13b` | Callee-saved. General purpose. |
| `r14` | `r14d` | `r14w` | — | `r14b` | Callee-saved. General purpose. |
| `r15` | `r15d` | `r15w` | — | `r15b` | Callee-saved. General purpose. |

> **Caller-saved:** You must save these before calling a function — the callee may overwrite them.
> **Callee-saved:** If a function uses these, it must save and restore them — the value is preserved across calls.

### Special Registers

| Register | Bits | Description |
|----------|------|-------------|
| `rip` | 64 | Instruction pointer. Address of the currently executing instruction. Cannot be written directly — only changed via `jmp`/`call`/`ret`. |
| `rflags` | 64 | Flag register. Automatically set by arithmetic and comparison instructions. |

### RFLAGS — Key Flags

| Flag | Bit | Set when | Used by |
|------|-----|----------|---------|
| `ZF` | 6 | Result == 0 | `je`, `jne`, `test reg,reg` |
| `SF` | 7 | Result < 0 (sign bit = 1) | `js`, `jns` |
| `CF` | 0 | Unsigned overflow or borrow | `jb`, `jae`, `jc` |
| `OF` | 11 | Signed overflow | `jo`, `jno` |
| `PF` | 2 | Low 8 bits have even parity | Rarely used |
| `AF` | 4 | BCD arithmetic carry | Only for BCD |
| `DF` | 10 | Direction: 0 = forward, 1 = backward | Controls `rep movsb` direction |

### SIMD Registers

```
zmm0 [511                                                              0]
      |                                                                |
ymm0 [255                            0]
      |                              |
xmm0 [127              0]
```

| Register | Bits | ISA | Description |
|----------|------|-----|-------------|
| `xmm0–xmm15` | 128 | SSE/SSE2 | Float/double arguments passed here (`xmm0–xmm7`). Return value in `xmm0`. |
| `ymm0–ymm15` | 256 | AVX/AVX2 | Upper 128 bits added on top of `xmm`. `xmm0` == `ymm0[127:0]`. |
| `zmm0–zmm31` | 512 | AVX-512 | Extends `ymm0–ymm15`. |

> `xmm0` and `ymm0` are the same physical register — just different-width views.
> An SSE instruction using `xmm` leaves `ymm`'s upper bits [255:128] dirty → `vzeroupper` required before switching back to SSE.

### Calling Convention (Linux — System V AMD64)

```
Integer arguments : rdi, rsi, rdx, rcx, r8, r9  (remaining on stack)
Float arguments   : xmm0–xmm7
Return (integer)  : rax  (128-bit: rdx:rax)
Return (float)    : xmm0
Stack alignment   : rsp must be 16-byte aligned before call
```

---

## Stack Frame — How It Works

```
Stack grows downward — from high addresses to low addresses.
This is why local variables use rbp-N (negative offsets).
```

```cpp
void foo(int a, int b) {
    int x = a + b;
    int y = x * 2;
}
```

```asm
foo:
    push   rbp               ; save caller's frame pointer
    mov    rbp, rsp          ; set new frame base
    sub    rsp, 16           ; allocate 16 bytes for locals (16-byte aligned)

    mov    DWORD PTR [rbp-4],  edi   ; a = 1st arg (edi)
    mov    DWORD PTR [rbp-8],  esi   ; b = 2nd arg (esi)

    mov    eax, DWORD PTR [rbp-4]    ; eax = a
    add    eax, DWORD PTR [rbp-8]    ; eax = a + b
    mov    DWORD PTR [rbp-12], eax   ; x = eax

    mov    eax, DWORD PTR [rbp-12]   ; eax = x
    add    eax, eax                   ; eax = x * 2
    mov    DWORD PTR [rbp-16], eax   ; y = eax

    leave                    ; mov rsp,rbp + pop rbp
    ret

Stack layout (assuming rbp = 0x7fff1000):
┌──────────────┬────────────────────────────────┐
│ Address      │ Contents                       │
├──────────────┼────────────────────────────────┤
│ 0x7fff1008   │ return address (8 bytes)       │
│ 0x7fff1000   │ old rbp (8 bytes)  ← rbp       │
│ 0x7ffe0ffc   │ int a (4 bytes)    rbp-4        │
│ 0x7ffe0ff8   │ int b (4 bytes)    rbp-8        │
│ 0x7ffe0ff4   │ int x (4 bytes)    rbp-12       │
│ 0x7ffe0ff0   │ int y (4 bytes)    rbp-16       │  ← rsp
└──────────────┴────────────────────────────────┘
```

> `rbp-4` = the address where `int a` starts.
> `old rbp` is 8 bytes (a 64-bit pointer), but the first int slot is `rbp-4` not `rbp-8`
> because the compiler assigns slots by variable size, starting right below rbp.
> Stack grows downward → going toward locals means subtracting → negative offsets.
