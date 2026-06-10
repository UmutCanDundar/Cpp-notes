# BINARY SEARCH

We choose a side in an array to look for the target by checking whether the middle element is smaller or bigger than the target, then do it again until the target is found. 

    Mid = low + (high - low) / 2

Time complexity - O(log n) 

Space complexity - O(1) (iterative) / O(log n) (recursive)

## Interpolation search:

Binary search with different formula

    Mid = Low + (High - Low) / (Arr[High] – Arr[Low]) * (X – Arr[Low])

Time complexity - O(log(log n)) 
