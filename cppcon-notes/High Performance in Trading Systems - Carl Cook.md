# (Notes from CppCon 2017: Carl Cook “When a Microsecond Is an Eternity: High Performance Trading Systems in C++” https://www.youtube.com/watch?v=NH1Tta7purM)

The factors that have impacts on the program (apart from language):
Compiler and version 
Machine architecture
Third-party library
Build and link flags

Low latency programming techniques:
1- Use Errorflags to check if there is an error instead of checking for all events:
If(!errorflags) Proceed;
else errorhandle(errorflags);
      Error handling code should not be inlined.
2- Virtual functions make codes slower.
3- Lambda functions are faster.
4- Memory allocation is costly: 
    • Use a pool of preallocated objects.
    • Reuse objects instead of deleting them.
    • If deletion is a necessity then do this from another thread.
5- We can use exceptions because they are zero cost if they do not throw.
     But don’t use exceptions for control flow.
6- Prefer templates to branches (if statements).
7- Multithreading is the best avoided for low latency:
    • Locking is expensive.
    • It is complex to implement parallelism correctly.
When we use multithreading: 
    • Keep shared data between ‘’hotpath’’ and everything else to a minimum. 
(Multiple threads at the same cache line get expensive)
    • Pass copies of data instead of sharing data.
(single writer, single reader lock-free queue)
    • If we must share data, we should not use synchronization.
	
8- std::unordered_map is faster container. 
          (hybrid hash map approach: both chaining and open addressing)
9- Use benchmark and measure your code (and profiling).
10- Don’t share L3:  disable all cores except 1 core. Choose neighbours carefully.
11- Consider inplace_function over std::function because std::function may allocate.
12- Keep the cache hot.
13- std::pow can be slow.
14- Don’t do system calls.
