# std::format (C++20)

`std::format` provides type-safe, Python-style string formatting, replacing error-prone `printf` and verbose `std::stringstream` chains.

## std::format

Formats arguments into a string using `{}` placeholders, checked at compile time.

```cpp
std::string s = std::format("{} is {} years old", "Alice", 30);
```

## std::format_to

Writes formatted output directly into an existing iterator/buffer, avoiding an extra string allocation.

```cpp
std::vector<char> buf;
std::format_to(std::back_inserter(buf), "{}-{}", 1, 2);
```

## std::format_to_n

Like `format_to`, but writes at most `n` characters, useful for fixed-size buffers.

```cpp
char buf[10];
std::format_to_n(buf, 10, "{}", 12345);
```

## std::formatted_size

Computes how many characters a format operation would produce, without actually writing them, useful for pre-allocating buffers.

```cpp
size_t n = std::formatted_size("{}-{}", 1, 2); // e.g. 3
```

## std::print (C++23)

Prints formatted text directly to a stream (like stdout), combining `std::format` and `std::cout` in one efficient call.

```cpp
std::print("Hello, {}!\n", "world");
```
