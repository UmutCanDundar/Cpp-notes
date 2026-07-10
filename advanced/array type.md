# Array Type

C++ arrays have special, sometimes surprising rules around pointers, references, decay, and templates. Understanding these avoids subtle bugs in generic code and low-level programming.

## Pointer to Arrays

A pointer to an array (not to its first element) preserves the array's size in its type, enabling correct pointer arithmetic across whole arrays.

```cpp
int arr[5];
int (*p)[5] = &arr; // pointer to an array of 5 ints
```

## Array References

A reference to an array binds to the whole array, keeping the size information, unlike a plain array parameter which decays to a pointer.

```cpp
void f(int (&arr)[5]) { std::cout << sizeof(arr); } // prints 20 (5*int)
```

## auto Type Deduction & Arrays

`auto` deduces arrays as decayed pointers unless you explicitly deduce a reference.

```cpp
int arr[3] = {1,2,3};
auto a = arr;   // int*, decays
auto& b = arr;  // int(&)[3], preserves array type
```

## Arrays as Template Arguments

Templates can deduce both the element type and the size of an array, which is powerful for generic array-handling code.

```cpp
template <typename T, size_t N>
size_t size(T (&)[N]) { return N; }

int arr[7];
size(arr); // returns 7
```

## Arrays and Explicit Specialization

You can fully specialize a template for a specific array type and size for custom behavior.

```cpp
template <typename T> struct Info;
template <> struct Info<int[10]> { static constexpr int value = 10; };
```

## Arrays and Partial Specialization

Templates can partially specialize on array size or element type, letting you write generic code for "any array of N elements."

```cpp
template <typename T> struct Info;
template <typename T, size_t N> struct Info<T[N]> { static constexpr size_t size = N; };
```

## Array Decay / No Array Decay

"Decay" means an array converts to a pointer to its first element in most contexts (like function parameters). References and templates can avoid this decay.

```cpp
void byPointer(int* p) {}   // arr decays to pointer here
void byRef(int (&arr)[5]) {} // no decay, size preserved
```

## Arrays and Type Alias Declarations

`using` can create clearer aliases for array types, especially useful for function pointers to arrays or multi-dimensional arrays.

```cpp
using IntArray5 = int[5];
IntArray5 arr; // same as int arr[5];
```
