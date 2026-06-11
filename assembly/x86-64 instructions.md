## Instructions

### Data Movement

| Instruction | Example | src | dst | Description |
|-------------|---------|-----|-----|-------------|
| `mov` | `mov rax, rbx` | rbx | rax | Copy rbx into rax. Most common instruction. |
| `mov` | `mov rax, [rbp-8]` | memory | rax | Read 8 bytes from address `rbp-8` → rax. |
| `mov` | `mov [rbp-8], rax` | rax | memory | Write rax to address `rbp-8`. |
| `mov` | `mov rax, 42` | immediate | rax | Load constant value. |
| `movzx` | `movzx rax, bl` | bl (8-bit) | rax (64-bit) | Zero-extend small → large register. Upper bits cleared. `bl=0xFF` → `rax=0x00...FF`. |
| `movsx` | `movsx rax, ecx` | ecx (32-bit) | rax (64-bit) | Sign-extend. Negative preserved. `ecx=-1` → `rax=-1` (all bits 1). |
| `movaps` | `movaps xmm0, [ptr]` | memory | xmm0 | 128-bit aligned load. ptr must be 16-byte aligned. |
| `movups` | `movups xmm0, [ptr]` | memory | xmm0 | 128-bit unaligned load. |
| `vmovaps` | `vmovaps ymm0, [ptr]` | memory | ymm0 | 256-bit aligned load (AVX). ptr must be 32-byte aligned. |
| `vmovdqu` | `vmovdqu ymm0, [ptr]` | memory | ymm0 | 256-bit unaligned load (AVX). |
| `lea` | `lea rax, [rbx + rcx*4]` | address expr | rax | Compute address, do NOT access memory. `rax = rbx + rcx*4`. Fast multiply/add trick. |
| `lea` | `lea rax, [rax + 5]` | — | rax | Same as `add rax, 5` but does not set flags. |
| `push` | `push rax` | rax | stack | `rsp -= 8`, then `[rsp] = rax`. |
| `pop` | `pop rax` | stack | rax | `rax = [rsp]`, then `rsp += 8`. |
| `xchg` | `xchg rax, rbx` | rax↔rbx | — | Swap two registers. With a memory operand: **implicit LOCK** — atomic. |
| `rep movsb` | `rep movsb` | rsi | rdi | Copy `rcx` bytes from rsi to rdi. Optimized memcpy on modern CPUs. |

### Arithmetic

| Instruction | Example | Description |
|-------------|---------|-------------|
| `add` | `add rax, rbx` | `rax = rax + rbx`. Sets ZF, CF, OF, SF. |
| `add` | `add rax, 5` | `rax = rax + 5`. Immediate. |
| `sub` | `sub rax, rbx` | `rax = rax - rbx`. |
| `sub` | `sub rsp, 24` | Reserve 24 bytes on stack for local variables. |
| `inc` | `inc rax` | `rax++`. Does NOT set CF — careful. |
| `dec` | `dec rax` | `rax--`. |
| `imul` | `imul rax, rbx` | `rax = rax * rbx`. Signed multiply. |
| `imul` | `imul rax, rbx, 5` | `rax = rbx * 5`. 3-operand form. |
| `idiv` | `idiv rcx` | `rax = rdx:rax / rcx`. Quotient→rax, Remainder→rdx. Use `cqo` first to sign-extend rdx. |
| `mul` | `mul rbx` | `rdx:rax = rax * rbx`. Unsigned multiply. |
| `neg` | `neg rax` | `rax = -rax`. Two's complement negate. |
| `cqo` | `cqo` | Sign-extend `rax` into `rdx:rax`. Required before `idiv`. |

### Bitwise & Shift

| Instruction | Example | Description |
|-------------|---------|-------------|
| `and` | `and rax, rbx` | `rax = rax & rbx`. Bitwise AND. |
| `and` | `and rax, 0xFF` | Masking — keep only the low 8 bits. |
| `or` | `or rax, rbx` | `rax = rax \| rbx`. |
| `xor` | `xor rax, rax` | `rax = 0`. Standard way to zero a register. Shorter than `mov rax, 0`. |
| `xor` | `xor rax, rbx` | `rax = rax ^ rbx`. |
| `not` | `not rax` | `rax = ~rax`. Flip all bits. |
| `shl` | `shl rax, 3` | `rax <<= 3` → `rax *= 8`. Logical shift left. |
| `shr` | `shr rax, 1` | `rax >>= 1`. Logical shift right. Upper bit filled with **0**. |
| `sar` | `sar rax, 1` | Arithmetic shift right. Upper bit filled with **sign bit**. Preserves sign of negative numbers. |
| `rol` | `rol rax, 1` | Rotate left. Shifted-out bit wraps to the other end. |
| `ror` | `ror rax, 1` | Rotate right. |
| `bsr` | `bsr rax, rbx` | Bit scan reverse. Index of highest set bit → rax. |
| `bsf` | `bsf rax, rbx` | Bit scan forward. Index of lowest set bit → rax. |
| `popcnt` | `popcnt rax, rbx` | Count of set bits in rbx → rax. |
| `tzcnt` | `tzcnt rax, rbx` | Count trailing zeros (zeros below the lowest set bit) → rax. |

### Comparison & Test

| Instruction | Example | Description |
|-------------|---------|-------------|
| `cmp` | `cmp rax, rbx` | Compute `rax - rbx`, discard result, set flags only. |
| `cmp` | `cmp rax, 0` | Is rax zero? Sets ZF. |
| `test` | `test rax, rax` | Compute `rax & rax`, discard result. Sets ZF=1 if rax==0. 1 byte shorter than `cmp rax, 0`. |
| `test` | `test rax, 1` | Is bit 0 set? (odd/even check) |
| `test` | `test al, al` | Check only the low 8 bits. |

### Conditional Set (SETcc) — write 1 byte based on flags

| Instruction | Condition | Example |
|-------------|-----------|---------|
| `sete` / `setz` | ZF=1 (equal / zero) | `sete al` |
| `setne` / `setnz` | ZF=0 (not equal) | `setne al` |
| `setl` / `setnge` | SF≠OF (signed less than) | `setl al` → `al = (a < b) ? 1 : 0` |
| `setg` / `setnle` | ZF=0 && SF=OF (signed greater than) | `setg al` |
| `setle` / `setng` | ZF=1 \|\| SF≠OF (signed ≤) | `setle al` |
| `setge` / `setnl` | SF=OF (signed ≥) | `setge al` |
| `setb` / `setc` | CF=1 (unsigned below) | `setb al` |
| `seta` | CF=0 && ZF=0 (unsigned above) | `seta al` |
| `sets` | SF=1 (negative) | `sets al` |
| `setns` | SF=0 (non-negative) | `setns al` |
| `seto` | OF=1 (overflow) | `seto al` |

> `setl al` writes 1 byte. Use `movzx rax, al` to widen to a full register.

### Conditional Move (CMOVcc) — branchless if

| Instruction | Condition | Example |
|-------------|-----------|---------|
| `cmove` | ZF=1 | `cmove rax, rbx` → `if (equal) rax = rbx` |
| `cmovne` | ZF=0 | |
| `cmovl` | signed less | `cmovl rax, rbx` → `if (a < b) rax = rbx` |
| `cmovg` | signed greater | |
| `cmovle` | signed ≤ | |
| `cmovge` | signed ≥ | |
| `cmovb` | unsigned below | |
| `cmova` | unsigned above | |

> No branch predictor miss. `std::min`/`std::max` are typically compiled to cmov by the compiler.

### Jumps

| Instruction | Condition | Description |
|-------------|-----------|-------------|
| `jmp` | — | Unconditional. `jmp label` or `jmp rax` (indirect). |
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
| `jz` | ZF=1 | Zero (alias for `je`). |
| `jnz` | ZF=0 | Non-zero (alias for `jne`). |
| `loop` | rcx-- != 0 | Decrement rcx, jump if not zero. |

### Call & Return

| Instruction | Description |
|-------------|-------------|
| `call func` | Push return address (`rip+1`) onto stack, then jump to `func`. Equivalent to `push rip; jmp func`. |
| `ret` | Pop return address from stack and jump to it. Equivalent to `pop rip`. |
| `ret 8` | Return and clean N bytes from stack (Windows stdcall convention). |
| `syscall` | Enter the kernel. `rax` = syscall number. Arguments: rdi, rsi, rdx, r10, r8, r9. |

### Memory Addressing Modes

```asm
[base]                    mov rax, [rbx]             ; *rbx
[base + offset]           mov rax, [rbx + 8]         ; *(rbx + 8)
[base + index*scale]      mov rax, [rbx + rcx*4]     ; rbx[rcx]  — int array
[base + index*scale + d]  mov rax, [rbx + rcx*8 + 16]
[rip + offset]            mov rax, [rip + label]     ; RIP-relative — global variable
```

Scale must be 1, 2, 4, or 8 — matches `sizeof(type)`.

### Memory Size Specifiers

```asm
BYTE  PTR [rbp-1]    ; 1 byte  — uint8_t, char
WORD  PTR [rbp-2]    ; 2 bytes — uint16_t, short
DWORD PTR [rbp-4]    ; 4 bytes — uint32_t, int, float
QWORD PTR [rbp-8]    ; 8 bytes — uint64_t, double, pointer
XMMWORD PTR [rsi]    ; 16 bytes — xmm register
YMMWORD PTR [rsi]    ; 32 bytes — ymm register
```

### Atomic & Synchronization

| Instruction | Description |
|-------------|-------------|
| `lock add [ptr], 1` | Atomic increment. Bus lock — no other CPU can access the cache line. |
| `lock cmpxchg [ptr], rbx` | Compare-and-swap. `rax` = expected value, `rbx` = new value. If `[ptr] == rax`: swap and set ZF=1. Otherwise: load current into `rax`, ZF=0. Maps to `std::atomic` CAS. |
| `lock xchg rax, [ptr]` | Atomic swap. `LOCK` prefix is implicit with `xchg` + memory operand. |
| `mfence` | Full memory barrier. No loads or stores may reorder across this point. |
| `sfence` | Store barrier. All prior stores become visible before any later stores. |
| `lfence` | Load barrier. All prior loads complete before any later loads. |
| `pause` | Spin-wait hint. Reduces power and pipeline pressure in busy-wait loops. Maps to `_mm_pause()`. |

### SIMD

| Instruction | Description |
|-------------|-------------|
| `vzeroupper` | Zero upper 128 bits of all ymm registers. Prevents AVX→SSE transition penalty. |
| `vmovaps ymm0, [ptr]` | 256-bit aligned load. ptr must be 32-byte aligned. |
| `vmovdqu ymm0, [ptr]` | 256-bit unaligned load. |
| `vmulps ymm0, ymm1, ymm2` | 8x float multiply. `ymm0 = ymm1 * ymm2`. |
| `vaddps ymm0, ymm1, ymm2` | 8x float add. |
| `vpand ymm0, ymm1, ymm2` | 256-bit bitwise AND. |
| `vpcmpeqd ymm0, ymm1, ymm2` | 8x int32 compare equal. Match lane → `0xFFFFFFFF`, no match → `0`. |

### Misc

| Instruction | Description |
|-------------|-------------|
| `nop` | Do nothing. Used for alignment padding or timing. |
| `nop DWORD PTR [rax]` | Multi-byte nop. Fills alignment gaps without side effects. |
| `ud2` | Undefined instruction — deliberate crash/trap. Sometimes emitted by `__builtin_unreachable()`. |
| `int3` | Breakpoint trap. Triggers the debugger. |
| `cpuid` | Query CPU features. `rax` = leaf number in, results in rax/rbx/rcx/rdx. |
| `rdtsc` | Read timestamp counter → `rdx:rax`. Cycle-accurate timer. |
| `rdtscp` | Like `rdtsc` but also reads the CPU ID. Serializes out-of-order execution. |

---

## objdump Output — Full Walkthrough

### Example C++ source

```cpp
int add(int a, int b) {
    return a + b;
}

int main() {
    int x = add(3, 4);
    return x;
}
```

### Compile and disassemble

```bash
g++ -O0 -g -o example example.cpp
objdump -d -M intel --no-show-raw-insn example   # Intel syntax, no raw bytes
# or with interleaved source:
objdump -d -M intel -S example
```

### objdump output (-O0)

```
0000000000001129 <_Z3addii>:                         ← (1)
    1129:   push   rbp                               ← (2)
    112a:   mov    rbp,rsp
    112d:   mov    DWORD PTR [rbp-0x4],edi           ← (3)
    1130:   mov    DWORD PTR [rbp-0x8],esi
    1133:   mov    edx,DWORD PTR [rbp-0x4]
    1136:   mov    eax,DWORD PTR [rbp-0x8]
    1139:   add    eax,edx                           ← (4)
    113b:   pop    rbp
    113c:   ret

000000000000113d <main>:                             ← (5)
    113d:   push   rbp
    113e:   mov    rbp,rsp
    1141:   sub    rsp,0x10                          ← (6)
    1145:   mov    esi,0x4                           ← (7)
    114a:   mov    edi,0x3
    114f:   call   1129 <_Z3addii>                  ← (8)
    1154:   mov    DWORD PTR [rbp-0x4],eax           ← (9)
    1157:   mov    eax,DWORD PTR [rbp-0x4]
    115a:   leave
    115b:   ret
```

### Column meanings

```
0000000000001129  <_Z3addii>:
^^^^^^^^^^^^^^^^^  ^^^^^^^^^^
       (A)             (B)

    1129:   push   rbp
    ^^^^    ^^^^^^^^^^^^
     (C)        (D)
```

| Column | Meaning |
|--------|---------|
| **(A)** Function address | Virtual address in the binary. `0x1129` = byte offset where this function starts. |
| **(B)** Symbol name | `_Z3addii` = mangled C++ name. Run `c++filt _Z3addii` → `add(int, int)`. |
| **(C)** Instruction address | Address of this specific instruction. Each instruction is 1–15 bytes, so addresses are not sequential. |
| **(D)** Assembly instruction | Intel syntax: `op dst, src`. |

### Line-by-line explanation

```
1129:  push rbp
```
> Save caller's frame pointer onto the stack. Standard function prologue.

```
112a:  mov rbp, rsp
```
> Set new frame base. From here, `rbp` = bottom of this function's stack frame.

```
112d:  mov DWORD PTR [rbp-0x4], edi
```
> Store 1st argument (`edi` = a = 3) into the stack. `DWORD PTR` = 4 bytes (int). First local slot at `rbp-4`.

```
1130:  mov DWORD PTR [rbp-0x8], esi
```
> Store 2nd argument (`esi` = b = 4). Second local slot at `rbp-8`.

```
1133:  mov edx, DWORD PTR [rbp-0x4]
1136:  mov eax, DWORD PTR [rbp-0x8]
```
> Reload a and b from stack back into registers. Unnecessary round-trip only present at `-O0`.

```
1139:  add eax, edx
```
> `eax = a + b`. Result in `eax` = return value (low 32 bits of `rax`).

```
113b:  pop rbp
113c:  ret
```
> Restore caller's frame pointer, return.

```
1141:  sub rsp, 0x10
```
> Reserve 16 bytes on the stack for local variable `x`. Only 4 bytes needed but rsp must stay 16-byte aligned.

```
1145:  mov esi, 0x4
114a:  mov edi, 0x3
```
> Set up arguments for `add(3, 4)`. 1st arg = rdi = 3, 2nd arg = rsi = 4.

```
114f:  call 1129 <_Z3addii>
```
> Push return address onto stack, jump to `0x1129`.

```
1154:  mov DWORD PTR [rbp-0x4], eax
```
> `eax` holds the return value (7). Store `x = add(3, 4)` onto the stack.

### Same code at -O2

```bash
g++ -O2 -o example_opt example.cpp
objdump -d -M intel example_opt
```

```
0000000000001110 <_Z3addii>:
    1110:   lea    eax,[rdi+rsi]    ← single instruction, no stack frame
    1113:   ret

0000000000001120 <main>:
    1120:   mov    eax,0x7          ← 3+4 evaluated at compile time
    1125:   ret                        no function call at all
```

> `lea eax, [rdi+rsi]` — compute `rdi + rsi`, store in `eax`. No push/pop/stack.
> `mov eax, 0x7` — constant folding: `3+4=7` computed at compile time. `add` was inlined, `call` eliminated entirely.

### Common patterns in the wild

```asm
; NULL check
test   rax, rax
je     .null_handler         ; jump if rax == 0

; loop counter increment
add    DWORD PTR [rbp-4], 1

; array element access: arr[i] where arr = int*, i = rcx
mov    eax, DWORD PTR [rax + rcx*4]
;                              ^^^^ sizeof(int) = 4

; pointer dereference: *ptr
mov    rax, QWORD PTR [rbp-8]   ; load the pointer value (an address)
mov    eax, DWORD PTR [rax]     ; dereference: read int at that address

; branchless min (cmov)
cmp    edi, esi
cmovle eax, edi                  ; if (a <= b) return a, else return b
```

---

## Quick Reference — Common Patterns

| When you see | It means |
|--------------|----------|
| `sub rsp, N` | Reserve N bytes for local variables |
| `mov [rbp-N], reg` | Write to a local variable |
| `mov reg, [rbp-N]` | Read from a local variable |
| `xor eax, eax` | rax = 0 (return 0) |
| `test rax, rax` + `je` | if (ptr == null) |
| `lea rax, [rdi+rsi*4]` | Pointer arithmetic: address of `arr[i]` |
| `call` + `mov [rbp-N], eax` | Call function and store result |
| `movzx eax, byte/word` | Widen small type to register, zero-fill |
| `movsx rax, dword` | int → long, sign-extend |
| `rep movsb` | memcpy |
| `lock cmpxchg` | Atomic CAS |
| `vmulps ymm0, ymm1, ymm2` | Multiply 8 floats at once (SIMD) |
| `vzeroupper` | Done with AVX, switching back to SSE |
