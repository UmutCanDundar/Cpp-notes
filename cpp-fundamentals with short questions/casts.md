# WHAT ARE STATIC_CAST AND REINTERPRET_CAST?
Static_cast is used to convert one type into another. It is a compile-time conversion.
Reinterpret_cast is used to the conversion of unrelated types (pointer to pointer, pointer to function, pointer to integral types etc.). It should not be used unless you do not have another option as it is inherently an unsafe cast type.
