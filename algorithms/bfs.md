# DFS (Depth-First Search)

## When to use
- Bir düğümden gidebildiğin kadar derine in, çıkmaz sokağa girince geri dön.
- Tüm yolları bulma, bağlantılılık kontrolü, ada/bölge sayma, ağaç gezme.
- **Not:** en kısa yol istiyorsan DFS değil BFS kullan.

## Mantık (tek fonksiyon)
Bir düğüme gir → ziyaret edildi işaretle → komşularına aynı fonksiyonu çağır → hepsi bitince geri dön.
Ağaç/graph/grid'de fark sadece **"komşu kimdir"** ve **"ziyaret nasıl işaretlenir"** kısmında.

---

## Graph versiyonu

```cpp
void dfs(unordered_map<int, vector<int>>& graph, int node, unordered_set<int>& visited) {
    if (visited.count(node)) return;   // zaten gezildi, çık
    visited.insert(node);              // gezildi işaretle
    // process(node);

    for (int neighbor : graph[node])   // her komşuya git
        dfs(graph, neighbor, visited);
}
```

### Adım adım örnek: graph = {0:[1,2], 1:[2], 2:[0,3], 3:[3]}, start=0

```
dfs(0)  visited={0}
 ├── dfs(1)  visited={0,1}
 │    └── dfs(2)  visited={0,1,2}
 │         ├── dfs(0) → zaten visited, çık
 │         └── dfs(3)  visited={0,1,2,3}
 │              └── dfs(3) → zaten visited, çık
 └── dfs(2) → zaten visited, çık
```
Ziyaret sırası: 0 → 1 → 2 → 3

---

## Grid versiyonu (row/col + yön vektörleri)

```cpp
void dfs(vector<vector<char>>& grid, int r, int c, vector<vector<bool>>& visited) {
    if (r < 0 || r >= rows || c < 0 || c >= cols) return; // sınır dışı, çık
    if (visited[r][c] || grid[r][c] == '#') return;        // gezildi/duvar, çık

    visited[r][c] = true;
    // process(r, c);

    dfs(grid, r-1, c, visited); dfs(grid, r+1, c, visited);
    dfs(grid, r, c-1, visited); dfs(grid, r, c+1, visited);
}
```
Fark: komşu bir liste yerine **yön ekleyerek** (`r±1, c±1`) hesaplanıyor, "var mı" kontrolü sınır kontrolüne dönüşüyor.

---

## Tree versiyonu (left/right)

```cpp
void dfs(TreeNode* node) {
    if (node == nullptr) return; // taban durum, ziyaret işaretine gerek yok (cycle yok)
    // process(node);
    dfs(node->left);
    dfs(node->right);
}
```
Fark: ağaçta cycle olmadığı için `visited` seti hiç gerekmiyor, `nullptr` doğal olarak durduruyor.

---

## Örnek Problem: Number of Islands (Grid)

Grid'de kaç tane bağlı `'1'` (kara) bölgesi var?

```cpp
int numIslands(vector<vector<char>>& grid) {
    int rows = grid.size(), cols = grid[0].size(), islands = 0;

    for (int r = 0; r < rows; r++)
        for (int c = 0; c < cols; c++)
            if (grid[r][c] == '1') {       // gezilmemiş kara bulundu
                islands++;                  // yeni bir ada
                sink(grid, r, c);            // bu adayı komple '0' yap
            }
    return islands;
}

void sink(vector<vector<char>>& grid, int r, int c) {
    if (r < 0 || r >= (int)grid.size() || c < 0 || c >= (int)grid[0].size()) return;
    if (grid[r][c] != '1') return;   // su ya da zaten gezildi

    grid[r][c] = '0';                // gezildi işaretle (batır)

    sink(grid, r+1, c); sink(grid, r-1, c);
    sink(grid, r, c+1); sink(grid, r, c-1);
}
```

### Adım adım: grid = [["1","1","0"],["0","1","0"],["0","0","1"]]

```
(0,0)='1' → islands=1, sink(0,0)
   sink(0,0): '0' yap → komşulara git: (1,0)='0' dur, (0,1)='1' → sink(0,1)
   sink(0,1): '0' yap → komşulara git: (1,1)='1' → sink(1,1)
   sink(1,1): '0' yap → komşuların hepsi '0' veya sınır dışı → dur
   → (0,0)-(0,1)-(1,1) hepsi '0' oldu, 1. ada bitti

(0,2)='0' → atla
(1,0)='0' → atla
(1,2)='0' → atla
(2,0)='0','2,1'='0' → atla
(2,2)='1' → islands=2, sink(2,2) → komşuların hepsi sınır/su → dur

Sonuç: islands = 2
```

**Complexity:** TC `O(rows*cols)`, SC `O(rows*cols)`

## Bu mantıkla çözülen benzer problemler
Max Area of Island, Flood Fill, Surrounded Regions, connected components sayma, iki düğüm arası yol var mı kontrolü.
