# WHAT ARE THE ADVANTAGES AND DISADVANTAGES OF USING POINTERS AS A CLASS MEMBER?

The advantages of using a pointer as a class member are lazy initialisation (to delay the creation of an object until it is used), reduction of header dependencies and giving more control to programmers.

The disadvantage of using a pointer as a class member is having to write our copy constructor and operators and that means more work, using a pointer can easily cause memory leak and accessing a pointer data member is slower than a non-pointer.
