# WHAT ARE THE DIFFERENCES BETWEEN POINTER AND REFERENCE?

Unlike pointers(*p), a reference(&r) 
    • must be declared and initialised at the same time,
    • is not allowed to re-assign,
    • does not need extra space on the stack, it shares its original variable memory location,
    • cannot be NULL, it has to be a valid value,
    • cannot be used with the arithmetic operators,
    • is just an alias of an existing variable, not its memory address,
