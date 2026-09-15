# A* Search

## When to use
- Belirli **tek bir hedefe** en kısa yol lazım ve hedefin nerede olduğunu biliyorsun (grid/harita).
- İyi bir tahmin fonksiyonun (heuristic) varsa Dijkstra'dan çok daha az düğüm gezersin.
- Tahmin fonksiyonun yoksa zaten Dijkstra'ya döner, zararı olmaz.

## Mantık (tek fonksiyon)
Dijkstra'nın aynısı, **tek fark**: heap'i sadece gerçek mesafeye (`g`) göre değil,
`f = g + h`'ye göre sırala. `h` = "buradan hedefe tahmini kalan mesafe" (örn. Manhattan distance).
Böylece heap, hedefe yakın görünen düğümleri önce çıkarır — Dijkstra gibi her yöne eşit yayılmak yerine hedefe doğru yönlenir.

```cpp
int heuristic(pair<int,int> a, pair<int,int> b) {
    return abs(a.first - b.first) + abs(a.second - b.second); // Manhattan distance
}

int aStar(vector<vector<char>>& grid, pair<int,int> start, pair<int,int> goal) {
    priority_queue<tuple<int,int,pair<int,int>>, vector<tuple<int,int,pair<int,int>>>, greater<>> pq; // {f, g, node}
    pq.push({heuristic(start, goal), 0, start});
    map<pair<int,int>, int> gScore;
    gScore[start] = 0;

    while (!pq.empty()) {
        auto [f, g, node] = pq.top(); pq.pop();
        if (node == goal) return g;             // hedefe ulaşıldı, gerçek mesafe g
        if (g > gScore[node]) continue;         // bayat kayıt

        for (auto& d : dirs) {
            pair<int,int> neighbor = {node.first + d.first, node.second + d.second};
            if (valid(neighbor)) {
                int newG = g + 1;
                if (!gScore.count(neighbor) || newG < gScore[neighbor]) {
                    gScore[neighbor] = newG;
                    int newF = newG + heuristic(neighbor, goal);   // f = g + h
                    pq.push({newF, newG, neighbor});
                }
            }
        }
    }
    return -1;
}
```

### Adım adım örnek: 1D koridor, start=0, goal=4, düğümler 0-1-2-3-4, heuristic(x)=|4-x|

```
pq=[(4,0,0)]        gScore={0:0}     (f=0+4=4)
pop (4,0,0) → hedef değil, komşu 1: g=1, h=3, f=4 → push (4,1,1)
pq=[(4,1,1)]        gScore={0:0,1:1}

pop (4,1,1) → hedef değil, komşu 2: g=2, h=2, f=4 → push (4,2,2)
pq=[(4,2,2)]        gScore={0:0,1:1,2:2}

pop (4,2,2) → hedef değil, komşu 3: g=3, h=1, f=4 → push (4,3,3)
pq=[(4,3,3)]

pop (4,3,3) → hedef değil, komşu 4: g=4, h=0, f=4 → push (4,4,4)
pq=[(4,4,4)]

pop (4,4,4) → node==goal → return g=4
```

Dikkat: `f` her adımda sabit 4 kaldı çünkü Manhattan distance düz bir koridorda gerçek mesafeyle birebir örtüşüyor — bu yüzden A* hiç yan yola sapmadan direkt hedefe ilerledi. Dallanma olsaydı (çatal yollar), yanlış yönler daha yüksek `f` alır ve heap'te geriye düşerdi; Dijkstra'da ise tüm yönler eşit önceliklendirilirdi.

---

## Grid ile graph arasındaki fark
Grid'de komşu = yön vektörü, heuristic = Manhattan/Euclidean (satır-sütun farkı).
Graph'ta komşu = adjacency list, heuristic için düğümlerin koordinatı (x,y veya lat/lng) olmalı, yoksa geçerli bir tahmin üretilemez ve Dijkstra'dan farkı kalmaz.

---

## Örnek Problem: Shortest Path in Binary Matrix (A* ile)

BFS örneğiyle aynı problem, bu kez tek hedefe yönlendirilmiş arama ile:

```cpp
int shortestPathBinaryMatrix(vector<vector<int>>& grid) {
    int n = grid.size();
    if (grid[0][0] != 0 || grid[n-1][n-1] != 0) return -1;

    auto heuristic = [&](int r, int c) { return max(n-1-r, n-1-c); }; // Chebyshev (8 yön için doğru)

    priority_queue<tuple<int,int,int,int>, vector<tuple<int,int,int,int>>, greater<>> pq; // {f,g,r,c}
    vector<vector<int>> gScore(n, vector<int>(n, INT_MAX));
    gScore[0][0] = 1;
    pq.push({heuristic(0,0), 1, 0, 0});

    while (!pq.empty()) {
        auto [f, g, r, c] = pq.top(); pq.pop();
        if (r == n-1 && c == n-1) return g;
        if (g > gScore[r][c]) continue;

        for (auto& d : dirs8) {
            int nr = r+d[0], nc = c+d[1];
            if (nr>=0 && nr<n && nc>=0 && nc<n && grid[nr][nc]==0) {
                int newG = g + 1;
                if (newG < gScore[nr][nc]) {
                    gScore[nr][nc] = newG;
                    pq.push({newG + heuristic(nr,nc), newG, nr, nc});
                }
            }
        }
    }
    return -1;
}
```

**Complexity:** TC Dijkstra ile aynı mertebe `O(n^2 log n)`, ama iyi heuristic sayesinde pratikte çok daha az düğüm ziyaret eder.

## Bu mantıkla çözülen benzer problemler
Sliding Puzzle, oyunlarda bot pathfinding, harita üzerinde iki nokta arası en kısa rota (koordinat varken).
