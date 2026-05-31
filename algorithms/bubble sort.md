# BUBBLE SORT

Always start with the first item and compare it to the next one: if it is greater, swap them and continue until array is sorted. 

Example:  Array {2,6,9,1,7,3}

	     When the process(inner loop) is complete for the first time: {2,6,1,7,3,9}
		 
 	     When the process(inner loop) is complete again: {2,6,1,3,7,9}
		 
This means that the greatest item among unsorted items goes to the end every time the process(inner loop) is repeated. 

Time complexity – O(n^2)

For almost sorted arrays – Ω(n) (Best Case)

Space complexity – O(1)
