# WHEN SHOULD WE DEFINE A DESTRUCTOR?

When we need to perform an action other than destructing class members, we define our destructor. For example, freeing memory. 
When we destruct an object through a base class pointer, the destructor must be defined as virtual.
