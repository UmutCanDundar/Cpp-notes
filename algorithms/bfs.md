# BFS (Breadth-First Search)

## When to use
- Ağırlıksız graf/grid'de **en kısa yol** / en az adım sayısı.
- Katman katman (level by level) gezme.
- **Not:** ağırlıklı graf için Dijkstra kullan.

## Mantık (tek fonksiyon)
Başlangıç düğümünü kuyruğa koy → kuyruktan çıkar, komşularını kuyruğa ekle (kuyruğa eklerken hemen "gezildi" işaretle) → kuyruk boşalana kadar devam et.
DFS'ten fark: **stack/recursion yerine queue** kullanılıyor, bu yüzden önce eklenen önce işlenir → katman katman ilerler.

---

## Graph versiyonu

```cpp
unordered_map<int,int> bfs(unordered_map<int, vector<int>>& graph, int start) {
    unordered_map<int,int> dist;
    unordered_set<int> visited;
    queue<int> q;

    q.push(start);
    visited.insert(start);   // kuyruğa girer girmez işaretle
    dist[start] = 0;

    while (!q.empty()) {
        int node = q.front(); q.pop();

        for (int neighbor : graph[node])
            if (!visited.count(neighbor)) {
                visited.insert(neighbor);
                dist[neighbor] = dist[node] + 1;
                q.push(neighbor);
            }
    }
    return dist;
}
```

### Adım adım örnek: graph = {0:[1,2], 1:[3], 2:[3], 3:[]}, start=0

```
q=[0]              visited={0}         dist={0:0}
pop 0 → komşular 1,2 gezilmemiş → ekle
q=[1,2]            visited={0,1,2}     dist={0:0,1:1,2:1}
pop 1 → komşu 3 gezilmemiş → ekle
q=[2,3]            visited={0,1,2,3}   dist={0:0,1:1,2:1,3:2}
pop 2 → komşu 3 zaten gezilmiş → atla
q=[3]
pop 3 → komşu yok
q=[]  → bitti

Sonuç: dist = {0:0, 1:1, 2:1, 3:2}
```

---

## Grid versiyonu (row/col + yön vektörleri, adım sayısı için katman takibi)

```cpp
int shortestPath(vector<vector<int>>& grid, pair<int,int> start, pair<int,int> target) {
    queue<pair<int,int>> q;
    vector<vector<bool>> visited(rows, vector<bool>(cols, false));
    q.push(start);
    visited[start.first][start.second] = true;
    int steps = 0;

    while (!q.empty()) {
        int size = q.size();          // bu katmandaki eleman sayısı
        for (int i = 0; i < size; i++) {
            auto [r, c] = q.front(); q.pop();
            if (r == target.first && c == target.second) return steps;

            for (auto& d : dirs) {
                int nr = r + d[0], nc = c + d[1];
                if (inBounds(nr, nc) && !visited[nr][nc] && grid[nr][nc] != 1) {
                    visited[nr][nc] = true;
                    q.push({nr, nc});
                }
            }
        }
        steps++;   // bir katman bitti, adım sayısını artır
    }
    return -1;
}
```
Fark: `size = q.size()` ile "bir katman"ı diğerinden ayırıyoruz, böylece adım sayısını sayabiliyoruz.

---

## Tree versiyonu (level-order)

```cpp
vector<vector<int>> levelOrder(TreeNode* root) {
    vector<vector<int>> result;
    if (!root) return result;
    queue<TreeNode*> q;
    q.push(root);

    while (!q.empty()) {
        int size = q.size();
        vector<int> level;
        for (int i = 0; i < size; i++) {
            TreeNode* node = q.front(); q.pop();
            level.push_back(node->val);
            if (node->left) q.push(node->left);
            if (node->right) q.push(node->right);
        }
        result.push_back(level);
    }
    return result;
}
```
Fark: `visited` yok (ağaçta cycle yok), komşu = `left`/`right`.

---

## Örnek Problem: Shortest Path in Binary Matrix

`(0,0)`'dan `(n-1,n-1)`'e 8 yönlü hareketle en kısa yolu bul (0=açık, 1=duvar).

```cpp
int shortestPathBinaryMatrix(vector<vector<int>>& grid) {
    int n = grid.size();
    if (grid[0][0] != 0 || grid[n-1][n-1] != 0) return -1;

    vector<vector<int>> dirs = {{-1,-1},{-1,0},{-1,1},{0,-1},{0,1},{1,-1},{1,0},{1,1}};
    queue<pair<int,int>> q;
    q.push({0,0});
    grid[0][0] = 1;          // grid'i visited olarak kullan
    int pathLength = 1;

    while (!q.empty()) {
        int size = q.size();
        for (int i = 0; i < size; i++) {
            auto [r,c] = q.front(); q.pop();
            if (r == n-1 && c == n-1) return pathLength;

            for (auto& d : dirs) {
                int nr = r+d[0], nc = c+d[1];
                if (nr>=0 && nr<n && nc>=0 && nc<n && grid[nr][nc]==0) {
                    grid[nr][nc] = 1;
                    q.push({nr,nc});
                }
            }
        }
        pathLength++;
    }
    return -1;
}
```

### Adım adım: grid = [[0,0,0],[1,1,0],[1,1,0]] (n=3)

```
q=[(0,0)]           pathLength=1
katman işle: (0,0) hedef değil, komşular: (0,1),(1,1)duvar,(1,0)duvar → (0,1) ekle
q=[(0,1)]           pathLength=2
katman işle: (0,1) hedef değil, komşular: (0,2) ekle (diğerleri duvar/gezilmiş)
q=[(0,2)]           pathLength=3
katman işle: (0,2) hedef değil, komşular: (1,2) ekle
q=[(1,2)]           pathLength=4
katman işle: (1,2) hedef değil, komşular: (2,2) ekle
q=[(2,2)]           pathLength=5
katman işle: (2,2) == hedef → return 5
```

**Complexity:** TC `O(n^2)`, SC `O(n^2)`

## Bu mantıkla çözülen benzer problemler
Rotting Oranges (çoklu kaynak BFS), Word Ladder, Level Order Traversal, minimum knight moves.
