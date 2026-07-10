# std::regex

`std::regex` provides built-in regular expression support for pattern matching, searching, and replacing in text, avoiding external regex libraries.

## Regex Grammar

`std::regex` supports multiple grammars (ECMAScript by default, also basic/extended POSIX), controlling the pattern syntax accepted.

```cpp
std::regex r("[0-9]+", std::regex::ECMAScript); // default grammar
```

## std::basic_regex

The template class representing a compiled regular expression; `std::regex` is `std::basic_regex<char>`.

```cpp
std::regex pattern(R"(\d{3}-\d{4})"); // compiled pattern
```

## std::sub_match

Represents a matched subexpression (capture group), convertible to a string, with position info.

```cpp
std::smatch m;
std::regex_search(s, m, pattern);
std::ssub_match group = m[1]; // first capture group
```

## std::match_results

Holds the full results of a match, including the whole match and all capture groups.

```cpp
std::smatch match;
if (std::regex_search(text, match, pattern)) {
    std::cout << match[0]; // full match
}
```

## regex_match

Checks whether the *entire* string matches the pattern exactly.

```cpp
bool ok = std::regex_match("12345", std::regex(R"(\d+)"));
```

## regex_search

Checks whether the pattern matches *anywhere* within the string.

```cpp
std::smatch m;
std::regex_search("abc123def", m, std::regex(R"(\d+)")); // finds "123"
```

## regex_replace

Replaces matched substrings with a replacement pattern.

```cpp
std::string result = std::regex_replace("hello world", std::regex("o"), "0");
// "hell0 w0rld"
```

## std::regex_iterator

Iterates over all matches of a pattern within a string, useful for finding every occurrence.

```cpp
std::sregex_iterator it(text.begin(), text.end(), pattern), end;
for (; it != end; ++it) std::cout << it->str() << "\n";
```

## std::regex_token_iterator

Iterates over specific capture groups (or the text between matches) across all matches, useful for tokenizing/splitting strings.

```cpp
std::sregex_token_iterator it(text.begin(), text.end(), pattern, -1); // split by pattern
```
