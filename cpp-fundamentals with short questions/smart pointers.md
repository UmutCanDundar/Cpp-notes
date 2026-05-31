# WHAT ARE SMART POINTERS?

Unlike raw pointers, smart pointers destroy themselves where they are out of scope so that memory leaks do not occur, in other words, programmers do not have to bother freeing memory. 

There are 3 types of smart pointers: 

unique_ptr: It allows only one owner of the object. 

shares_ptr: It allows more than one owner that point to the object. Reference counter, which is accessed by using use_count() method, includes the number of owners. 

weak_ptr: It is used with shares_ptr but not added to reference counter.
