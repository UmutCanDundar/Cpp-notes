# Basic x86-64 Instructions

| Instruction | Syntax (Intel) | What It Does |
|-------------|----------------|--------------|
| mov | `mov rax, rbx` | Copies rbx into rax. Most common instruction. |
| movzx | `movzx rax, byte [ptr]` | Copies small value to larger register, zero-extends upper bits. |
| movsx | `movsx rax, dword [ptr]` | Copies with sign-extension. For negative numbers. |
| lea | `lea rax, [rbx + rcx*4]` | Computes address but does NOT access memory. Fast multiply/add trick. |
| push / pop | `push rax` / `pop rax` | Write to / read from stack. rsp updated automatically. |
| add / sub | `add rax, 5` | Addition / subtraction. Flags updated. |
| imul / idiv | `imul rax, rbx` | Signed multiply/divide. idiv uses rdx:rax pair. |
| inc / dec | `inc rax` | Increment / decrement by 1. |
| and / or / xor | `xor rax, rax` | Bitwise operations. `xor reg,reg` is the standard way to zero a register. |
| not | `not rax` | Bitwise NOT. |
| shl / shr | `shl rax, 3` | Logical shift left/right. shl 3 = ×8. |
| sar | `sar rax, 1` | Arithmetic shift right. Sign bit is preserved. |
| cmp | `cmp rax, rbx` | Subtracts but discards result, only updates flags. |
| test | `test rax, rax` | ANDs but discards result. Use `test reg,reg` to check for zero. |
| jmp | `jmp label` | Unconditional jump. |
| je/jne/jl/jg/jle/jge | `je label` | Conditional jump based on flags. je=equal, jne=not equal, jl=less than... |
| call / ret | `call func` | call: push return address to stack, then jmp. ret: pop + jmp. |
| nop | `nop` | Does nothing. Used for alignment or timing. |
| xchg | `xchg rax, rbx` | Swaps two registers. xchg with memory is implicitly LOCKed. |
| lock prefix | `lock add [ptr], 1` | Atomic operation. Bus lock. Maps to `std::atomic fetch_add`. |
| cmpxchg | `lock cmpxchg [ptr], rbx` | Compare-and-swap. Assembly equivalent of atomic CAS. |
| pause | `pause` | Hints to CPU in spin-wait loops. Maps to `_mm_pause()`. |
| mfence/lfence/sfence | `mfence` | Memory barrier. mfence=full, lfence=load, sfence=store. |
| vzeroupper | `vzeroupper` | Zeros upper 128 bits of ymm registers. Prevents state transition penalty on SSE↔AVX switch. Compiler inserts after AVX functions. |
| vmovdqu/vmovaps | `vmovdqu ymm0, [ptr]` | SIMD load. vmovaps=aligned, vmovdqu=unaligned. |
| rep movsb | `rep movsb` | Copies rcx bytes from rsi to rdi. Optimized form of memcpy on modern CPUs. |
