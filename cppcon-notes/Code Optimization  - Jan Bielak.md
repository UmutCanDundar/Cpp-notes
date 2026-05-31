#(Notes from The Most Important Optimizations to Apply in Your C++ Programs - Jan Bielak - CppCon 2022 https://www.youtube.com/watch?v=qCjEN5XRzHc and more)

    1- Compiler optimization (common flags): 
    
• MSVC: /O1 < /O2 < /Ox 
    
cl /O2 main.cpp
https://learn.microsoft.com/en-us/cpp/build/reference/o-options-optimize-code?view=msvc-170

    2- Set target architecture:
Compilers could be optimized for a specific CPU architecture. Determine the CPU(s) on which your code will be run.

• gcc: 

-march=native(code may not run on other CPUs) , -mtune=native (better performance on your CPU)
https://gcc.gnu.org/onlinedocs/gcc-6.3.0/gcc/Submodel-Options.html#Submodel-Options 

• MSVC: 

/arch options: Target architecture for the program.
https://learn.microsoft.com/en-us/cpp/build/reference/arch-minimum-cpu-architecture?view=msvc-170&redirectedfrom=MSDN

and -MACHINE: Target platform for the program.
learn.microsoft.com/en-us/cpp/build/reference/machine-specify-target-platform?view=msvc-170&redirectedfrom=MSDN

    3- Use fast math:

• MSVC:

Compilers use /fp:precise by default which preserves ordering and rounding properties.
/fp:fast ignores them and makes code faster and smaller. Be careful to use!
https://learn.microsoft.com/en-us/cpp/build/reference/fp-specify-floating-point-behavior?view=msvc-170

    4- Unity builds:
    
Header files are parsed once, which means that the compiler is invoked fewer times. It makes linking faster because of the smaller total size of object files.

    5- Use constexpr, const, noexcept and assume
    
    6- Mark pointers restrict, functions as pure(MSVC does not support it)
    
    7- Function parameters 
    
    8- Avoid allocation and deallocation on the heap (new and delete are slow) (Prefer stack allocation)
    
    9- Avoid unnecessary copying (exceptions, in lambda captures, in structured bindings)
    
    10- Exploit data locality: 
    
Reuse data as much as possible rather than creating new copies.
Loop through an array linearly rather than jumping around randomly. 

    11- Avoid false sharing
    
    12- Avoid indirected calls
    
    13- Make branches predictable
    
    14- Use branchless optimization
    
    15- SIMD intrinsics
    
    16- Recursion is worse than iteration (Call stack problem)
    
    17- Memory alignment: 
• Data structures should be aligned properly to cache lines. For example, int array[16] = 16x4 = 64 byte. 
• The number of elements should be a power of 2. 
• Use ‘’alignas(int constant)’’.

    18- Use cache-friendly structure: 
• Prefer structure of arrays to array of structures.

    19- Lock-free programming: Locking causes latency.
    
    20- Use initializer lists and std::move.
    
    21- Use reserve() method to avoid expensive reallocation.
