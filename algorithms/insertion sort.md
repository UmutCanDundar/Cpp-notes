# INSERTION SORT

Always start with the second item and compare it to the previous one: 
	
		 if it is smaller, swap them and continue until array is sorted. 

Example: 

		 Array {2,6,9,1,7,3}

	     When the process(inner loop) is complete for the first time: {1,2,6,9,7,3}
		 
 	     When the process(inner loop) is complete again: {1,2,6,9,7,3}
		 
This means that the smallest item among unsorted items goes to the beginning every time the process is repeated. 

Time complexity – O(n^2)

For almost sorted arrays – Ω(n) (Best Case)

Space complexity – O(1)
