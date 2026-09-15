# Backtracking (DFS'in "tüm yolları bul" versiyonu)

## When to use
- "**Tüm** yolları/kombinasyonları/çözümleri bul" diyen problemler (sadece biri değil, hepsi).
- N-Queens, Sudoku, tüm geçerli parantez dizilimleri, iki nokta arası tüm yollar.

## Mantık (tek fonksiyon)
DFS ile birebir aynı: bir düğüme gir → gezildi işaretle → komşulara git → geri dön.
**Tek fark:** DFS'te bir düğüm gezilince sonsuza kadar gezilmiş sayılır. Burada ise sadece
**şu anki yol boyunca** gezilmiş sayılır — fonksiyondan çıkarken (geri dönerken) işareti kaldırıyoruz,
çünkü o düğüm başka bir yolda tekrar kullanılabilir olmalı.

```cpp
void backtrack(unordered_map<int, vector<int>>& graph, int node, int target,
               unordered_set<int>& visited, vector<int>& path, vector<vector<int>>& allPaths) {

    visited.insert(node);       // bu yol için gezildi işaretle
    path.push_back(node);       // yola ekle

    if (node == target) {
        allPaths.push_back(path);   // hedefe ulaşıldı, bu yolu kaydet
    } else {
        for (int neighbor : graph[node])
            if (!visited.count(neighbor))
                backtrack(graph, neighbor, target, visited, path, allPaths);
    }

    path.pop_back();      // geri al: bu düğümü yoldan çıkar
    visited.erase(node);  // geri al: bu düğüm artık başka yolda tekrar kullanılabilir
}
```

### Adım adım örnek: graph = {0:[1,2], 1:[3], 2:[3], 3:[]}, start=0, target=3

```
backtrack(0)  visited={0}  path=[0]
 ├── backtrack(1)  visited={0,1}  path=[0,1]
 │    └── backtrack(3)  visited={0,1,3}  path=[0,1,3]
 │         3==target → allPaths += [0,1,3]
 │         geri dön: path=[0,1], visited={0,1}
 │    geri dön: path=[0], visited={0}
 └── backtrack(2)  visited={0,2}  path=[0,2]
      └── backtrack(3)  visited={0,2,3}  path=[0,2,3]
           3==target → allPaths += [0,2,3]
           geri dön: path=[0,2], visited={0,2}
      geri dön: path=[0], visited={0}
 geri dön: path=[], visited={}

Sonuç: allPaths = [[0,1,3], [0,2,3]]
```

DFS'ten farkı burada net görünüyor: `3` düğümü iki farklı yolda (`0→1→3` ve `0→2→3`) kullanıldı, çünkü her seferinde geri dönerken `visited.erase(node)` ile temizledik. Normal DFS'te `3` bir kere gezilince kilitli kalırdı ve ikinci yol asla bulunamazdı.

---

## Grid versiyonu (yol geziliyor, `visited` yerine grid'in kendisi işaretleniyor)

```cpp
void backtrack(vector<vector<int>>& grid, int r, int c,
               vector<pair<int,int>>& path, vector<vector<pair<int,int>>>& allPaths) {
    if (!inBounds(r,c) || grid[r][c] == 1) return;   // duvar/sınır dışı, çık

    grid[r][c] = 1;          // bu yol için gezildi işaretle
    path.push_back({r,c});

    if (r == rows-1 && c == cols-1) {
        allPaths.push_back(path);   // hedefe ulaşıldı, kaydet
    } else {
        backtrack(grid, r+1, c, path, allPaths);
        backtrack(grid, r-1, c, path, allPaths);
        backtrack(grid, r, c+1, path, allPaths);
        backtrack(grid, r, c-1, path, allPaths);
    }

    path.pop_back();
    grid[r][c] = 0;   // geri al: bu hücre başka yolda tekrar kullanılabilir
}
```

## Tree versiyonu (visited hiç gerekmiyor, sadece path geri alınıyor)

```cpp
void backtrack(TreeNode* node, vector<int>& path, vector<vector<int>>& allPaths) {
    if (!node) return;
    path.push_back(node->val);

    if (!node->left && !node->right) allPaths.push_back(path);  // yaprak, kaydet
    else {
        backtrack(node->left, path, allPaths);
        backtrack(node->right, path, allPaths);
    }

    path.pop_back();  // geri al
}
```

---

## Örnek Problem: All Paths From Source to Target (DAG)

`0`'dan `n-1`'e kadar olan tüm yolları bul.

```cpp
void backtrack(vector<vector<int>>& graph, int node, vector<int>& path, vector<vector<int>>& allPaths) {
    int target = graph.size() - 1;

    if (node == target) {
        allPaths.push_back(path);
        return;
    }

    for (int neighbor : graph[node]) {
        path.push_back(neighbor);                    // seç
        backtrack(graph, neighbor, path, allPaths);   // devam et
        path.pop_back();                               // geri al
    }
}

vector<vector<int>> allPathsSourceTarget(vector<vector<int>>& graph) {
    vector<vector<int>> allPaths;
    vector<int> path = {0};
    backtrack(graph, 0, path, allPaths);
    return allPaths;
}
```

### Adım adım: graph = [[1,2],[3],[3],[]]  (0→1,2 / 1→3 / 2→3 / 3→hiçbir yer)

```
backtrack(0, path=[0])
 ├── neighbor=1: path=[0,1]
 │    backtrack(1, path=[0,1])
 │     └── neighbor=3: path=[0,1,3]
 │          backtrack(3, path=[0,1,3]) → 3==target → allPaths += [0,1,3]
 │          geri al: path=[0,1]
 │    geri al: path=[0]
 └── neighbor=2: path=[0,2]
      backtrack(2, path=[0,2])
       └── neighbor=3: path=[0,2,3]
            backtrack(3, path=[0,2,3]) → 3==target → allPaths += [0,2,3]
            geri al: path=[0,2]
      geri al: path=[0]

Sonuç: allPaths = [[0,1,3],[0,2,3]]
```

DAG olduğu için `visited` seti bile gerekmedi — graf yapısı zaten cycle'a izin vermiyor.

**Complexity:** en kötü durumda `O(2^n * n)`

## Bu mantıkla çözülen benzer problemler
Word Search (grid), N-Queens, Sudoku Solver, Generate Parentheses.
