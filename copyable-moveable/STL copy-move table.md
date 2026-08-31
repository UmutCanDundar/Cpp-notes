# C++ Standard Library — Copy / Move Table

✅ = supported | ❌ = deleted | ⚠️ = depends on template argument(s)

## Containers

| Type | Copyable | Movable | Move noexcept | Notes |
|---|---|---|---|---|
| `vector` | ⚠️ | ⚠️ | ✅ | |
| `deque` | ⚠️ | ⚠️ | ✅ | |
| `list` / `forward_list` | ⚠️ | ⚠️ | ✅ | |
| `array<T,N>` | ⚠️ | ⚠️ | ⚠️ | move is O(n), not O(1) |
| `map` / `set` / `multimap` / `multiset` | ⚠️ | ⚠️ | ✅ | |
| `unordered_map` / `unordered_set` | ⚠️ | ⚠️ | ✅ | |
| `stack` / `queue` / `priority_queue` | ⚠️ | ⚠️ | depends on underlying container | |
| `span` / `initializer_list` | ✅ | ✅ | ✅ | non-owning |

## Smart Pointers

| Type | Copyable | Movable | Move noexcept | Notes |
|---|---|---|---|---|
| `unique_ptr` | ❌ | ✅ | ✅ | |
| `shared_ptr` | ✅ | ✅ | ✅ | |
| `weak_ptr` | ✅ | ✅ | ✅ | |

## Concurrency

| Type | Copyable | Movable | Move noexcept | Notes |
|---|---|---|---|---|
| `thread` / `jthread` | ❌ | ✅ | ✅ | |
| `mutex` / `recursive_mutex` / `timed_mutex` / `shared_mutex` | ❌ | ❌ | — | |
| `lock_guard` | ❌ | ❌ | — | |
| `unique_lock` / `shared_lock` | ❌ | ✅ | ✅ | |
| `condition_variable` | ❌ | ❌ | — | |
| `atomic<T>` | ❌ | ❌ | — | |
| `future` / `promise` / `packaged_task` | ❌ | ✅ | ✅ | |
| `shared_future` | ✅ | ✅ | ✅ | |

## Utility / Vocabulary Types

| Type | Copyable | Movable | Move noexcept | Notes |
|---|---|---|---|---|
| `optional` / `variant` / `expected` | ⚠️ | ⚠️ | ⚠️ | |
| `pair` / `tuple` | ⚠️ | ⚠️ | ⚠️ | |
| `any` | ✅ (if held type copyable) | ✅ | ✅ | |
| `function` | ✅ (if callable copyable) | ✅ | ⚠️ | **not guaranteed noexcept** |
| `reference_wrapper` | ✅ | ✅ | ✅ | |
| `bitset` | ✅ | ✅ | ✅ | |
| `chrono::duration` / `time_point` | ✅ | ✅ | ✅ | |

## Strings

| Type | Copyable | Movable | Move noexcept | Notes |
|---|---|---|---|---|
| `string` / `wstring` / `u16string` / `u32string` | ✅ | ✅ | ✅ | |
| `string_view` | ✅ | ✅ | ✅ | |
| `pmr::string` | ✅ | ✅ | ⚠️ | **not always noexcept**, allocator-dependent |

## Streams

| Type | Copyable | Movable | Move noexcept | Notes |
|---|---|---|---|---|
| `ifstream` / `ofstream` / `fstream` | ❌ | ✅ | ⚠️ | usually not noexcept |
| `istringstream` / `ostringstream` / `stringstream` | ❌ | ✅ | ⚠️ | usually not noexcept |
| `cin` / `cout` / `cerr` | ❌ | ❌ | — | |

## Miscellaneous

| Type | Copyable | Movable | Move noexcept | Notes |
|---|---|---|---|---|
| `filesystem::path` | ✅ | ✅ | ✅ | |
| `regex` | ✅ | ✅ | ⚠️ | |
| `locale` | ✅ | ✅ | ✅ | |
| `exception` and derived | ✅ | ✅ | ✅ | |
| `type_info` | ❌ | ❌ | — | |
| `coroutine_handle<>` | ✅ | ✅ | ✅ | |
| `mdspan` | ✅ | ✅ | ✅ | |
