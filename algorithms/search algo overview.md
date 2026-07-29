# Algorithms Guide (C++): Search, Traversal & Related Techniques

Covers everything except sorting: Linear/Binary Search, BFS, DFS, Dijkstra, A*, Two Pointers, Sliding Window, Union-Find, Topological Sort, MST (Kruskal/Prim), Bellman-Ford, Floyd-Warshall, DP, Greedy, Heaps, Trie — when to use each, why, TC/SC, code, and the "visited" question.

---

## 1. Linear Search

### When to use
- Unsorted data, small datasets, or you need *all* occurrences.

### Complexity
- **TC:** O(n) | **SC:** O(1)

```cpp
int linearSearch(vector<int>& arr, int target) {
    for (int i = 0; i < (int)arr.size(); i++) {
        if (arr[i] == target) return i;
    }
    return -1;
}
```

---

## 2. Binary Search

### When to use
- Data is sorted.
- "Search on answer" pattern: minimizing/maximizing a value under a monotonic feasibility condition.

### Complexity
- **TC:** O(log n) | **SC:** O(1)

```cpp
int binarySearch(vector<int>& arr, int target) {
    int lo = 0, hi = (int)arr.size() - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}
```

### Search-on-answer template
```cpp
int binarySearchOnAnswer(int lo, int hi, function<bool(int)> feasible) {
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (feasible(mid)) hi = mid;      // mid could be the answer
        else lo = mid + 1;
    }
    return lo;
}
```

---

## 3. BFS (Breadth-First Search)

### When to use
- **Shortest path in an unweighted graph/grid** — main signal: "minimum steps/moves".
- Level-order tree traversal, connected components, multi-source problems (rotting oranges style).

### Why visited matters
Graphs/grids can have cycles or multiple paths to the same node. Without `visited`, you get infinite/redundant work, and lose the shortest-path guarantee (first visit = shortest distance in unweighted BFS).

### Complexity
- **TC:** O(V + E) for graphs, O(rows×cols) for grids | **SC:** O(V)

```cpp
#include <queue>
#include <vector>
#include <unordered_set>
using namespace std;

vector<int> bfs(vector<vector<int>>& graph, int start) {
    unordered_set<int> visited = {start};
    queue<int> q;
    q.push(start);
    vector<int> order;

    while (!q.empty()) {
        int node = q.front(); q.pop();
        order.push_back(node);
        for (int neighbor : graph[node]) {
            if (!visited.count(neighbor)) {
                visited.insert(neighbor);   // mark when ENQUEUED, not when popped
                q.push(neighbor);
            }
        }
    }
    return order;
}
```

### BFS shortest path / distance
```cpp
int bfsShortestPath(vector<vector<int>>& graph, int start, int target) {
    unordered_set<int> visited = {start};
    queue<pair<int,int>> q; // {node, dist}
    q.push({start, 0});

    while (!q.empty()) {
        auto [node, dist] = q.front(); q.pop();
        if (node == target) return dist;
        for (int neighbor : graph[node]) {
            if (!visited.count(neighbor)) {
                visited.insert(neighbor);
                q.push({neighbor, dist + 1});
            }
        }
    }
    return -1; // unreachable
}
```

**Key habit:** mark visited the moment you push into the queue, not when you pop — otherwise the same node can be queued multiple times before processing.

---

## 4. DFS (Depth-First Search)

### When to use
- Cycle detection, topological sort, counting connected components/islands.
- Backtracking (permutations, combinations, subsets, N-Queens, Sudoku).
- Path existence (not shortest path), tree traversals.

### Visited rules
- **Trees:** usually no `visited` needed — just don't go back to the parent you came from.
- **Graphs/grids:** always need `visited`, marked permanently (never unmark).
- **Backtracking:** need a `used` tracker, but **unmark it after the recursive call returns** — a different branch may need to reuse that element.

```
Graph/grid traversal  → mark visited once, never unmark
Backtracking          → mark "used", then UNMARK after recursion returns
```

### Complexity
- **TC:** O(V+E) graphs/grids, O(b^d) backtracking | **SC:** O(V) visited + O(H) recursion stack

### DFS on a graph (recursive)
```cpp
void dfs(vector<vector<int>>& graph, int node, unordered_set<int>& visited) {
    visited.insert(node);
    for (int neighbor : graph[node]) {
        if (!visited.count(neighbor)) {
            dfs(graph, neighbor, visited);
        }
    }
}
```

### DFS on a grid (iterative, with stack)
```cpp
#include <stack>

set<pair<int,int>> dfsGrid(vector<vector<char>>& grid, pair<int,int> start) {
    int rows = grid.size(), cols = grid[0].size();
    set<pair<int,int>> visited = {start};
    stack<pair<int,int>> st;
    st.push(start);
    int dr[] = {-1, 1, 0, 0};
    int dc[] = {0, 0, -1, 1};

    while (!st.empty()) {
        auto [r, c] = st.top(); st.pop();
        for (int d = 0; d < 4; d++) {
            int nr = r + dr[d], nc = c + dc[d];
            if (nr >= 0 && nr < rows && nc >= 0 && nc < cols &&
                !visited.count({nr, nc}) && grid[nr][nc] != '#') {
                visited.insert({nr, nc});
                st.push({nr, nc});
            }
        }
    }
    return visited;
}
```

### Backtracking (mark/unmark pattern)
```cpp
void backtrack(vector<int>& nums, vector<bool>& used, vector<int>& path,
               vector<vector<int>>& result) {
    if (path.size() == nums.size()) {
        result.push_back(path);
        return;
    }
    for (int i = 0; i < (int)nums.size(); i++) {
        if (used[i]) continue;
        used[i] = true;
        path.push_back(nums[i]);
        backtrack(nums, used, path, result);
        path.pop_back();
        used[i] = false;   // UNMARK — this is what makes it backtracking
    }
}

vector<vector<int>> permutations(vector<int>& nums) {
    vector<vector<int>> result;
    vector<bool> used(nums.size(), false);
    vector<int> path;
    backtrack(nums, used, path, result);
    return result;
}
```

---

## 5. Dijkstra's Algorithm

### When to use
- Weighted shortest path, **all weights ≥ 0**, need distances to all nodes (or no heuristic available).
- If weights can be negative → use Bellman-Ford instead.

### Why it needs a priority queue
BFS treats every edge as cost 1. Dijkstra generalizes this with a **min-heap ordered by current best-known distance**, always expanding the cheapest-to-reach node first.

### Visited / "finalized" logic
Maintain a `dist` map and a min-heap. When popping a node:
- If already finalized (visited), skip it.
- Otherwise finalize it and relax its neighbors.

### Complexity
- **TC:** O((V+E) log V) | **SC:** O(V)

```cpp
#include <queue>
#include <unordered_map>
#include <vector>
using namespace std;

// graph: node -> list of {neighbor, weight}
unordered_map<int,int> dijkstra(unordered_map<int, vector<pair<int,int>>>& graph, int start) {
    unordered_map<int,int> dist;
    dist[start] = 0;
    unordered_set<int> visited;

    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
    pq.push({0, start});

    while (!pq.empty()) {
        auto [d, node] = pq.top(); pq.pop();
        if (visited.count(node)) continue;
        visited.insert(node);

        for (auto& [neighbor, weight] : graph[node]) {
            int newDist = d + weight;
            if (!dist.count(neighbor) || newDist < dist[neighbor]) {
                dist[neighbor] = newDist;
                pq.push({newDist, neighbor});
            }
        }
    }
    return dist;
}
```

---

## 6. A* Search

### When to use
- Shortest path to a **specific target** with a good heuristic (e.g., grid pathfinding with Manhattan/Euclidean distance to the goal).
- Games, robotics, navigation/maps.
- No good heuristic → behaves like Dijkstra, no harm in using it.

### Complexity
- **TC:** O(E) worst case (like Dijkstra), much faster in practice with a good heuristic | **SC:** O(V)

```cpp
#include <queue>
#include <cmath>
using namespace std;

int heuristic(pair<int,int> a, pair<int,int> b) {
    return abs(a.first - b.first) + abs(a.second - b.second); // Manhattan distance
}

int aStar(vector<vector<char>>& grid, pair<int,int> start, pair<int,int> goal) {
    int rows = grid.size(), cols = grid[0].size();
    using State = tuple<int,int,pair<int,int>>; // {f, g, node}
    priority_queue<State, vector<State>, greater<>> openSet;
    openSet.push({heuristic(start, goal), 0, start});

    map<pair<int,int>, int> gScore;
    gScore[start] = 0;
    set<pair<int,int>> visited;

    int dr[] = {-1, 1, 0, 0};
    int dc[] = {0, 0, -1, 1};

    while (!openSet.empty()) {
        auto [f, g, node] = openSet.top(); openSet.pop();
        if (node == goal) return g;
        if (visited.count(node)) continue;
        visited.insert(node);

        int r = node.first, c = node.second;
        for (int d = 0; d < 4; d++) {
            int nr = r + dr[d], nc = c + dc[d];
            pair<int,int> neighbor = {nr, nc};
            if (nr >= 0 && nr < rows && nc >= 0 && nc < cols && grid[nr][nc] != '#') {
                int tentativeG = g + 1;
                if (!gScore.count(neighbor) || tentativeG < gScore[neighbor]) {
                    gScore[neighbor] = tentativeG;
                    int fScore = tentativeG + heuristic(neighbor, goal);
                    openSet.push({fScore, tentativeG, neighbor});
                }
            }
        }
    }
    return -1; // unreachable
}
```

---

## 7. Two Pointers

### When to use
- Sorted array problems: pair sum, triplet sum, removing duplicates.
- Palindrome checks.
- Merging two sorted arrays/lists.
- "Opposite direction" pattern: shrink a window from both ends based on a condition.

### Complexity
- **TC:** O(n) (vs O(n²) brute force) | **SC:** O(1)

```cpp
pair<int,int> twoSumSorted(vector<int>& arr, int target) {
    int left = 0, right = arr.size() - 1;
    while (left < right) {
        int sum = arr[left] + arr[right];
        if (sum == target) return {left, right};
        else if (sum < target) left++;
        else right--;
    }
    return {-1, -1};
}
```

---

## 8. Sliding Window

### When to use
- "Longest/shortest substring with condition X" (no repeating chars, at most K distinct, etc.)
- "Max sum subarray of size K" or "subarray sum ≤/≥ target".
- Any "contiguous" + constraint problem.

### State tracking
Keep a hash map/array counting frequencies inside the window; shrink the window (move `left`) when the condition is violated. Window bounds *are* your state — no separate visited set needed.

### Complexity
- **TC:** O(n) | **SC:** O(k) for the window's frequency map

```cpp
int longestUniqueSubstring(string s) {
    unordered_map<char,int> lastSeen;
    int left = 0, best = 0;
    for (int right = 0; right < (int)s.size(); right++) {
        if (lastSeen.count(s[right]) && lastSeen[s[right]] >= left) {
            left = lastSeen[s[right]] + 1;   // shrink window
        }
        lastSeen[s[right]] = right;
        best = max(best, right - left + 1);
    }
    return best;
}
```

---

## 9. Union-Find / Disjoint Set Union (DSU)

### When to use
- Detecting cycles in an undirected graph.
- Dynamically counting/merging connected components.
- **Kruskal's MST algorithm** (below).
- "Are these two nodes connected after adding this edge?" style queries.

### Complexity
- **TC:** ~O(α(n)) per op (practically constant) | **SC:** O(n)

```cpp
class DSU {
public:
    vector<int> parent, rank_;
    DSU(int n) : parent(n), rank_(n, 0) {
        for (int i = 0; i < n; i++) parent[i] = i;
    }
    int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]); // path compression
        return parent[x];
    }
    bool unite(int a, int b) {
        int ra = find(a), rb = find(b);
        if (ra == rb) return false; // already connected -> would form a cycle
        if (rank_[ra] < rank_[rb]) swap(ra, rb);
        parent[rb] = ra;
        if (rank_[ra] == rank_[rb]) rank_[ra]++;
        return true;
    }
};
```

---

## 10. Topological Sort (Kahn's Algorithm)

### When to use
- Task scheduling with dependencies ("course prerequisites").
- Build systems, compilation order.
- Detecting cycles in a directed graph (if not all nodes get ordered, a cycle exists).

### Complexity
- **TC:** O(V + E) | **SC:** O(V)

```cpp
vector<int> topologicalSort(int n, vector<vector<int>>& graph) {
    vector<int> indegree(n, 0);
    for (int u = 0; u < n; u++)
        for (int v : graph[u]) indegree[v]++;

    queue<int> q;
    for (int i = 0; i < n; i++)
        if (indegree[i] == 0) q.push(i);

    vector<int> order;
    while (!q.empty()) {
        int node = q.front(); q.pop();
        order.push_back(node);
        for (int neighbor : graph[node]) {
            if (--indegree[neighbor] == 0) q.push(neighbor);
        }
    }
    // order.size() != n -> cycle exists
    return order;
}
```

---

## 11. Minimum Spanning Tree: Kruskal & Prim

### When to use
- "Connect all cities with minimum cable/road cost" type problems.
- **Kruskal:** sort edges, add greedily if no cycle (uses DSU). Better for sparse graphs.
- **Prim:** grow the tree from a start node, always adding the cheapest edge to a new node (uses a min-heap). Better for dense graphs.

### Complexity
- **Kruskal TC:** O(E log E) | **Prim TC:** O(E log V)

```cpp
int kruskalMST(int n, vector<array<int,3>>& edges) { // {weight, u, v}
    sort(edges.begin(), edges.end());
    DSU dsu(n);
    int totalWeight = 0;
    for (auto& [w, u, v] : edges) {
        if (dsu.unite(u, v)) totalWeight += w;
    }
    return totalWeight;
}
```

---

## 12. Bellman-Ford

### When to use
- Weighted shortest path where **negative weights** are possible.
- Need to detect a negative cycle (e.g., arbitrage detection).

### Complexity
- **TC:** O(V × E) | **SC:** O(V)

```cpp
vector<long long> bellmanFord(int n, vector<array<int,3>>& edges, int src) {
    vector<long long> dist(n, LLONG_MAX);
    dist[src] = 0;
    for (int i = 0; i < n - 1; i++) {
        for (auto& [u, v, w] : edges) {
            if (dist[u] != LLONG_MAX && dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
            }
        }
    }
    // one more pass: if any distance still improves -> negative cycle exists
    return dist;
}
```

---

## 13. Floyd-Warshall

### When to use
- Need shortest paths between *every* pair of nodes, not just from one source.
- Small graphs (V ≲ 400–500), since it's O(V³).
- Works with negative weights (but not negative cycles).

### Complexity
- **TC:** O(V³) | **SC:** O(V²)

```cpp
void floydWarshall(vector<vector<long long>>& dist, int n) {
    for (int k = 0; k < n; k++)
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                if (dist[i][k] + dist[k][j] < dist[i][j])
                    dist[i][j] = dist[i][k] + dist[k][j];
}
```

---

## 14. Dynamic Programming (Memoization & Tabulation)

### When to use
- Problem has **optimal substructure** + **overlapping subproblems**.
- Signals: "count the number of ways", "min/max cost to reach", "can you reach exact value X" (knapsack family), "longest common/increasing subsequence", "edit distance".
- No overlap between subproblems → plain recursion/backtracking is enough, DP won't help.

### Complexity
- **TC:** O(states × transitions) | **SC:** O(states) (often reducible with rolling arrays)

```cpp
// 0/1 Knapsack - bottom-up
int knapsack(vector<int>& weights, vector<int>& values, int capacity) {
    int n = weights.size();
    vector<vector<int>> dp(n + 1, vector<int>(capacity + 1, 0));
    for (int i = 1; i <= n; i++) {
        for (int w = 0; w <= capacity; w++) {
            dp[i][w] = dp[i-1][w];
            if (weights[i-1] <= w) {
                dp[i][w] = max(dp[i][w], dp[i-1][w - weights[i-1]] + values[i-1]);
            }
        }
    }
    return dp[n][capacity];
}

// Fibonacci - top-down memoization
unordered_map<int, long long> memo;
long long fib(int n) {
    if (n <= 1) return n;
    if (memo.count(n)) return memo[n];
    return memo[n] = fib(n-1) + fib(n-2);
}
```

---

## 15. Greedy Algorithms

### When to use
- Interval scheduling (activity selection — sort by end time, pick greedily).
- Huffman coding.
- Coin change **only with canonical denominations** — otherwise use DP.
- Jump game, gas station problems.

### DP vs Greedy
- Local best never needs revisiting → **Greedy**.
- Local best can be wrong later, need to compare combinations → **DP**.

### Complexity
- **TC:** usually O(n log n) (dominated by sorting) | **SC:** O(1)–O(n)

```cpp
int maxActivities(vector<pair<int,int>>& intervals) {
    sort(intervals.begin(), intervals.end(), [](auto& a, auto& b) {
        return a.second < b.second;
    });
    int count = 0, lastEnd = INT_MIN;
    for (auto& [start, end] : intervals) {
        if (start >= lastEnd) {
            count++;
            lastEnd = end;
        }
    }
    return count;
}
```

---

## 16. Heap / Priority Queue Patterns

### When to use
- "Top K elements", "K closest points", "median of a stream" (two heaps).
- Any greedy/graph problem needing repeated min/max access (Dijkstra, Prim, Huffman all rely on this).

### Complexity
- Push/pop: O(log n) | Top-K: O(n log k)

```cpp
vector<int> topKFrequent(vector<int>& nums, int k) {
    unordered_map<int,int> freq;
    for (int n : nums) freq[n]++;

    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> minHeap;
    for (auto& [val, f] : freq) {
        minHeap.push({f, val});
        if ((int)minHeap.size() > k) minHeap.pop();
    }

    vector<int> result;
    while (!minHeap.empty()) {
        result.push_back(minHeap.top().second);
        minHeap.pop();
    }
    return result;
}
```

---

## 17. Trie (Prefix Tree)

### When to use
- Autocomplete, prefix search ("words starting with X").
- Word search / dictionary problems.
- Bit-trie variant: maximum XOR pair problems.

### Complexity
- **TC:** O(L) per insert/search (L = word length) | **SC:** O(total characters inserted)

```cpp
struct TrieNode {
    TrieNode* children[26] = {};
    bool isEnd = false;
};

class Trie {
    TrieNode* root;
public:
    Trie() { root = new TrieNode(); }

    void insert(const string& word) {
        TrieNode* node = root;
        for (char c : word) {
            int idx = c - 'a';
            if (!node->children[idx]) node->children[idx] = new TrieNode();
            node = node->children[idx];
        }
        node->isEnd = true;
    }

    bool search(const string& word) {
        TrieNode* node = root;
        for (char c : word) {
            int idx = c - 'a';
            if (!node->children[idx]) return false;
            node = node->children[idx];
        }
        return node->isEnd;
    }
};
```

---

## Decision Guide

| Signal in the problem | Likely algorithm |
|---|---|
| "sorted array", "answer range" | Binary Search |
| "minimum steps/moves", unweighted graph | BFS |
| "all permutations/combinations/subsets" | Backtracking (DFS + mark/unmark) |
| "connected components", "cycle detection" (undirected) | DFS or Union-Find |
| "prerequisites", "build order", DAG | Topological Sort |
| "cheapest cost", weights ≥ 0 | Dijkstra |
| "cheapest cost", negative weights possible | Bellman-Ford |
| "shortest path between ALL pairs" | Floyd-Warshall |
| "connect all nodes, min total cost" | Kruskal / Prim (MST) |
| "contiguous subarray/substring + constraint" | Sliding Window |
| "sorted array + pair/triplet target" | Two Pointers |
| "count ways" / "min-max cost with choices" / overlapping subproblems | Dynamic Programming |
| "locally best choice always safe" (intervals, canonical coins) | Greedy |
| "top K", "Kth largest", "median of stream" | Heap / Priority Queue |
| "prefix search", "autocomplete" | Trie |

### Cheat sheet: core graph/search algorithms

| Algorithm | Data structure | Shortest path guarantee | Needs weights? | TC |
|---|---|---|---|---|
| Linear Search | array | — | — | O(n) |
| Binary Search | sorted array | — | — | O(log n) |
| BFS | queue | Yes (unweighted) | No | O(V+E) |
| DFS | stack/recursion | No | No | O(V+E) |
| Dijkstra | min-heap | Yes (non-negative weights) | Yes | O((V+E) log V) |
| A* | min-heap + heuristic | Yes (if heuristic admissible) | Yes | O(E) worst, faster in practice |
| Bellman-Ford | edge list | Yes (handles negative weights) | Yes | O(V·E) |
| Floyd-Warshall | adjacency matrix | Yes (all pairs) | Yes | O(V³) |

---

## Visited/State Tracking — Full Recap

| Algorithm | State tracked | Ever "unmarked"? |
|---|---|---|
| BFS / DFS (graph) | `visited` set, marked on enqueue/push | No — permanent |
| Backtracking | `used[]` marker | **Yes** — unmark after recursion returns |
| Dijkstra / A* | `visited`/finalized + `dist`/`gScore` map | No — permanent once finalized |
| Union-Find | `parent[]` array (which set you belong to) | No — merges are permanent |
| DP (memo) | `memo` cache of subproblem results | No — overwritten only if state redefined |
| Sliding Window | window bounds + frequency map | Implicitly — shrinks/expands as window moves |

**Rule of thumb:** if a state can legitimately be revisited from a different valid path (backtracking) → unmark it. If revisiting a state is always wasted work (plain graph traversal, Union-Find, Dijkstra) → mark it permanently, forever.
