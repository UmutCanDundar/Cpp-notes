# x86-64 Registers — All Names and Roles

| 64-bit | 32-bit | 16-bit | 8-bit | Role / Note |
|--------|--------|--------|-------|-------------|
| rax | eax | ax | al/ah | Return value, division result |
| rbx | ebx | bx | bl/bh | Callee-saved (function preserved) |
| rcx | ecx | cx | cl/ch | 4th argument, loop counter (for rep) |
| rdx | edx | dx | dl/dh | 3rd argument, high half of division |
| rsi | esi | si | sil | 2nd argument, source index |
| rdi | edi | di | dil | 1st argument, destination index |
| rsp | esp | sp | spl | Stack pointer — do not use directly |
| rbp | ebp | bp | bpl | Frame pointer, callee-saved |
| r8–r11 | r8d–r11d | r8w–r11w | r8b–r11b | 5th–8th arguments, caller-saved |
| r12–r15 | r12d–r15d | r12w–r15w | r12b–r15b | Callee-saved, general purpose |
| rip | — | — | — | Instruction pointer — address of currently executing instruction |
| rflags | eflags | — | — | ZF/CF/SF/OF flags — comparison results |
| xmm0–xmm15 | — | — | — | 128-bit SSE registers. Float/double arguments passed here. |
| ymm0–ymm15 | — | — | — | 256-bit AVX. Upper half of xmm. |
| zmm0–zmm31 | — | — | — | 512-bit AVX-512. |

> **Linux calling convention (System V AMD64):** Arguments in order: rdi, rsi, rdx, rcx, r8, r9. Rest on stack. Return in rax. Floats in xmm0–xmm7.
