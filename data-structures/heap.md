# HEAP

Is a complete binary tree 
Can have duplicates
Parents must be greater/lower than their children (max heap/min heap)
Can be implemented with a vector:  leftChild = 2 * parentIndex + 1
				            rightChild = 2 * parentIndex + 2
				            parentIndex = (childIndex-1) / 2 (for both)	
Can be used to implement priority_queue (more efficient)
Insert, Remove - O(log n)
Space Complexity – O(n)	

##Corresponds to (STL) std::priority_queue:

Is implemented as a vector 
Elements are stored in order (front is the largest)
Iterators are not provided
Insertion and deletion – O(n)
p_q.top(), p_q.pop(), p_q.push(), p_q.size(), p_q.empty(), …
https://en.cppreference.com/w/cpp/container/priority_queue
