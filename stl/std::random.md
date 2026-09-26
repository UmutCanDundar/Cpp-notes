# `<random>`

### Notation
* `a`, `b`: the inclusive bounds of a uniform distribution's range (`[a, b]` for ints, `[a, b)` for reals).
* `p`: a probability (`0.0`–`1.0`). `mean`, `stddev`: parameters of a normal distribution. `lambda`: rate parameter of an exponential distribution.
* `eng`: an instance of a random engine (e.g. `mt19937`), the source of randomness a distribution consumes.
* `dist`: an instance of a distribution object.

---

### Constructors

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| *(constructor)* | engine, default | `mt19937 eng;` | O(state size) | Seeds with a fixed default seed — **deterministic**, same sequence every run (fine for reproducible tests, not for real randomness). |
| *(constructor)* | engine, seeded | `mt19937 eng(seed)`<br>`mt19937 eng(seed_seq)` | O(state size) | Seeds from a single integer, or from a `seed_seq` for better statistical seeding across multiple engines. |
| *(constructor)* | uniform_int_distribution | `uniform_int_distribution<T> dist(a, b)` | O(1) | Distribution objects are cheap to construct — the expensive part is the engine, not the distribution. |
| *(constructor)* | uniform_real_distribution | `uniform_real_distribution<T> dist(a, b)` | O(1) | Same cost profile as above. |
| *(constructor)* | normal_distribution | `normal_distribution<T> dist(mean, stddev)` | O(1) | Gaussian distribution parameters. |
| *(constructor)* | seed_seq | `seed_seq sseq(sources...)` | O(k), k = number of sources | Combines multiple seed sources (e.g. several `random_device` calls) for better statistical seeding of an engine. |

---

### Methods

| Returns | Method | Usage | Complexity | Note |
|---------|--------|-------|------------|------|
| `result_type` | operator() | `eng()`<br>`dist(eng)` | O(1) | Advances the engine and returns its next raw value · draws one value from `dist`, consuming one or more calls to `eng`. |
| `void` | seed() | `eng.seed(s)` | O(state size) | Re-seeds an already-constructed engine. |
| `void` | discard(n) | `eng.discard(n)` | O(n) | Advances the engine's state by `n` steps without returning the values — used to skip ahead in a sequence. |
| `result_type` | min() / max() | `eng.min()` / `eng.max()` | O(1) | The inclusive range of raw values the engine can produce (not the distribution's range). |
| `param_type` | param() | `dist.param()`<br>`dist.param(new_params)` | O(1) | Gets/sets a distribution's parameters after construction, without rebuilding the object. |
| `void` | reset() | `dist.reset()` | O(1) | Clears any internal cached state a distribution might keep between calls (e.g. some `normal_distribution` implementations generate two values per call and cache one). |
| *(engine type)* | mt19937 / mt19937_64 | `std::mt19937 eng(seed)` | O(1) per draw, O(2.5KB state) | Mersenne Twister PRNG. Good statistical quality, fast, but has ~2.5KB of state — don't construct it per-call. |
| *(engine type)* | minstd_rand | `std::minstd_rand eng(seed)` | O(1) per draw, tiny state | Linear congruential engine — much faster/smaller than `mt19937`, lower statistical quality. Fine for perf-sensitive simulation, not for security. |
| *(entropy source)* | random_device | `std::random_device rd;`<br>`rd()` | O(1) per call, but slow (syscall) | True OS-sourced entropy, used once to seed an engine. Slow — never use it as the generator itself in a loop. |
| *(distribution type)* | uniform_int_distribution\<T\> | `uniform_int_distribution<int> dist(a, b)` | O(1) | Uniform integers in `[a, b]`. |
| *(distribution type)* | uniform_real_distribution\<T\> | `uniform_real_distribution<double> dist(a, b)` | O(1) | Uniform floating point in `[a, b)`. |
| *(distribution type)* | normal_distribution\<T\> | `normal_distribution<double> dist(mean, stddev)` | O(1) | Gaussian/normal distribution — for simulating noisy signals or returns. |
| *(distribution type)* | discrete_distribution\<T\> | `discrete_distribution<int> dist(weights.begin(), weights.end())` | O(1), built in O(k) from k weights | Weighted random choice from a fixed set of outcomes. For loot tables / weighted sampling. |
| *(distribution type)* | bernoulli_distribution | `bernoulli_distribution dist(p)` | O(1) | Random `true`/`false` with probability `p` of `true`. |
| *(distribution type)* | exponential_distribution\<T\> | `exponential_distribution<double> dist(lambda)` | O(1) | Models the time between independent events — used for simulating order arrival times. |
| *(distribution type)* | poisson_distribution\<T\> | `poisson_distribution<int> dist(mean)` | O(1) | Number of events in a fixed interval, given an average rate `mean` — the discrete counterpart of `exponential_distribution`. |
| *(distribution type)* | geometric_distribution\<T\> | `geometric_distribution<int> dist(p)` | O(1) | Number of Bernoulli trials needed to get one success — e.g. "how many retries until this succeeds." |

> **Reuse the engine, not recreate it.** Constructing an engine or distribution per call is a common perf mistake — construct once, call repeatedly. `mt19937`'s ~2.5KB state makes per-call construction especially costly.
