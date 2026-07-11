# `<random>`

| API | Priority | Description |
|-----|----------|-------------|
| `std::mt19937` / `std::mt19937_64` | memorize | Mersenne Twister PRNG engine. Good quality, fast, but has ~2.5KB state — don't construct it per-call. |
| `std::random_device` | know | True (OS-sourced) entropy, used once to seed an engine. Slow — never use it as the generator itself in a loop. |
| `std::uniform_int_distribution<T>` | memorize | Uniform integers in `[a,b]`. Distribution objects are cheap, reuse the engine. |
| `std::uniform_real_distribution<T>` | memorize | Uniform floating point in `[a,b)`. |
| `std::normal_distribution<T>` | know | Gaussian/normal distribution, for simulating noisy signals or returns. |
| `std::discrete_distribution<T>` | know | Weighted random choice from a set of outcomes. For loot tables / weighted sampling. |
| `std::bernoulli_distribution` | know | Random true/false with given probability p. |
| `std::exponential_distribution<T>` | know | Models time between independent events — used for simulating order arrival times. |
| `std::seed_seq` | know | Combines multiple seed sources for better statistical seeding of an engine. |
| `engine()` reused vs recreated | careful | Recreating an engine or distribution per call is a common perf mistake — construct once, call repeatedly. |
| `std::minstd_rand` | know | Linear congruential engine — much faster/smaller than mt19937, lower quality. Fine for perf-sensitive simulation, not for security. |
