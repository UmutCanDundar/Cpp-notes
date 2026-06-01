# Readability vs Speed — When to Choose Which

| Situation | Preference | Why |
|-----------|------------|-----|
| Hot path — measured bottleneck | speed | Every cycle matters here. Write manual code, add explanatory comments. |
| Cold path — init, config, error | readability | Runs rarely, latency doesn't matter. Write clearly, make it maintainable. |
| Removing ifs and writing arrays | measure first | Hurts readability AND isn't always faster. Do it if the profiler shows it. |
| Manual loop instead of STL algo | usually STL | STL is readable and usually equally fast. Manual code increases bug risk. Measure on hot path. |
| Short but complex one-liner | avoid | Clever code ≠ good code. Even you won't understand it in 3 months. Write a comment or separate lines. |

## Steps When Asked to "Optimize" 

| # | Step |
|---|------|
| 1 | Ask: "What should I optimize — latency, throughput, or memory?" — clarifying question |
| 2 | Ask: "Is there a profile? Is the bottleneck known?" |
| 3 | Check Big-O — is there an algorithm change possible? |
| 4 | Is there allocation on the hot path? |
| 5 | How is the cache access pattern — sequential or random? |
| 6 | Is there heavy branching? |
| 7 | Check assembly on Godbolt — what is the compiler generating? |
| 8 | Make the change, run benchmarks, compare. |
