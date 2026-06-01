# HASH TABLE - class

We can use a fixed-size (node*) array - each index has a linked-list of key-value pair(s) (collision – Separate Chaining Method) 

We use hash function to determine the address of a pair in hash table 

[If the address is not empty, we move through the indexes until we find an empty address(Linear Probing - Open Addressing Method)]

Set, Get – O(1)

Space Complexity – O(n)

We can also use std::underored_map to implement hash table
