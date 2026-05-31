# WHAT IS TEMPLATE-METAPROGRAMMING?

Template-metaprogramming is a technique that the program performs computations at compile-time in other words it runs at compile-time, to write a metaprogram, we use compile-time types and variables (const, constexpr) instead of run-time features (RTTI, virtual).

# WHAT IS A METAFUNCTION?

A metafunction is actually a class/struct that can also return a value just as regular functions do and it is commonly used in the context of template metaprogramming. 
For instance:
template <int num1, int num2>
struct Add
{
    static constexpr int value = num1 + num2;
};

“type_traits” is a standard c++ library that provides many metafunctions.
