# WHAT IS PASSING BY (VALUE, REFERENCE, POINTER, CONST REFERENCE, CONST POINTER)?

Passing by value means that a copy of the actual variable is passed into the function as a parameter, the actual variable does not get affected by the function. 

Passing by reference means that the actual variable is passed into the function as a parameter and this variable can be modified when the function is called because we deliberately refer to the actual one by using reference. If we use “const” reference, the function cannot change the variable's value. 

Passing by pointer is similar to passing by reference, so the pointer’s value can be modified permanently in the function. Unlike reference, we also can re-assign our pointer in the function but it is passed by value, which means that our pointer is copied before passing into the function, so the actual one, which is outside of the function, does not get affected. If we use “const” pointer, the function cannot change the memory address that the pointer points to.
