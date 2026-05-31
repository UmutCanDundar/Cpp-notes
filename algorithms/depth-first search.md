# DEPTH-FIRST SEARCH

Going to the deepest node (leaf) before backtracking (tree)

We can use recursion to implement preorder, Inorder and postorder DFS (tree).

We use stack<Node*>:  
            
            we visit every node and push it into stack, 
            
            when we reach a node that does not have any unvisited neighbours, 
            
            we return to the previous one by popping to find an unvisited node (graph).

Time complexity - O(V+E)

Space complexity - O(V)
