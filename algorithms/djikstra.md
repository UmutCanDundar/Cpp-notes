# Dijkstra's Algorithm

## When to use
- Ağırlıklı graf, **tüm ağırlıklar ≥ 0**, bir başlangıçtan tüm düğümlere en kısa mesafe lazım.
- Negatif ağırlık varsa → Bellman-Ford kullan (bu geçerli değil).
- Ağırlıklar hep aynıysa (1) → BFS yeter, heap'e gerek yok.

## Mantık (tek fonksiyon)
BFS'in aynısı, **tek fark**: kuyruk yerine "en küçük mesafeliyi önce çıkar" diyen bir **min-heap**
kullanılıyor, çünkü artık her adımın maliyeti farklı (1 değil, ağırlık kadar).
Bir düğümü kuyruktan çıkardığında zaten "gezildi" ise atla (bayat kayıt), değilse gezildi yap ve komşularını gevşet (relax): oraya gitmenin daha ucuz bir yolu varsa mesafesini güncelle.

```cpp
unordered_map<int,int> dijkstra(unordered_map<int, vector<pair<int,int>>>& graph, int start) {
    unordered_map<int,int> dist;
    dist[start] = 0;
    unordered_set<int> visited;

    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq; // {mesafe, düğüm}
    pq.push({0, start});

    while (!pq.empty()) {
        auto [d, node] = pq.top(); pq.pop();
        if (visited.count(node)) continue;  // bayat kayıt, atla
        visited.insert(node);                // artık kesinleşti

        for (auto& [neighbor, weight] : graph[node]) {
            int newDist = d + weight;
            if (!dist.count(neighbor) || newDist < dist[neighbor]) {
                dist[neighbor] = newDist;    // daha kısa yol bulundu, güncelle
                pq.push({newDist, neighbor});
            }
        }
    }
    return dist;
}
```

### Adım adım örnek: graph = {0:[(1,4),(2,1)], 1:[(3,1)], 2:[(1,2),(3,5)], 3:[]}, start=0

```
pq=[(0,0)]                    dist={0:0}
pop (0,0) → gezildi={0}
  komşu 1: newDist=4 → dist[1]=4, pq push (4,1)
  komşu 2: newDist=1 → dist[2]=1, pq push (1,2)
pq=[(1,2),(4,1)]               dist={0:0,1:4,2:1}

pop (1,2) → gezildi={0,2}
  komşu 1: newDist=1+2=3 < dist[1]=4 → dist[1]=3, pq push (3,1)
  komşu 3: newDist=1+5=6 → dist[3]=6, pq push (6,3)
pq=[(3,1),(4,1),(6,3)]          dist={0:0,1:3,2:1,3:6}

pop (3,1) → gezildi={0,1,2}
  komşu 3: newDist=3+1=4 < dist[3]=6 → dist[3]=4, pq push (4,3)
pq=[(4,1)-bayat,(4,3),(6,3)-bayat]

pop (4,1) → zaten gezildi ({0,1,2} içinde) → atla (bayat kayıt)

pop (4,3) → gezildi={0,1,2,3}, komşusu yok

Sonuç: dist = {0:0, 1:3, 2:1, 3:4}
```

Dikkat et: `1` düğümüne ilk giren mesafe 4'tü ama sonra 2 üzerinden 3'e düştü — heap sayesinde daha ucuzu bulununca eski kayıt otomatik "bayat" oldu ve atlandı.

---

## Grid versiyonu (hücreye girme maliyeti değişken olduğunda)

```cpp
int dijkstraGrid(vector<vector<int>>& grid) {
    vector<vector<int>> dist(rows, vector<int>(cols, INT_MAX));
    priority_queue<vector<int>, vector<vector<int>>, greater<>> pq; // {mesafe, r, c}
    dist[0][0] = grid[0][0];
    pq.push({dist[0][0], 0, 0});

    while (!pq.empty()) {
        auto top = pq.top(); pq.pop();
        int d = top[0], r = top[1], c = top[2];
        if (d > dist[r][c]) continue;  // bayat kayıt

        for (auto& dir : dirs) {
            int nr = r+dir[0], nc = c+dir[1];
            if (inBounds(nr, nc)) {
                int newDist = d + grid[nr][nc];   // bu hücreye girmenin maliyeti
                if (newDist < dist[nr][nc]) {
                    dist[nr][nc] = newDist;
                    pq.push({newDist, nr, nc});
                }
            }
        }
    }
    return dist[rows-1][cols-1];
}
```
Fark: komşu = yön vektörü, "gezildi" kontrolü yerine "popladığım mesafe, elimdeki en iyisinden kötü mü" kontrolü var — ikisi de aynı işi görüyor.

---

## Örnek Problem: Network Delay Time

`k` düğümünden sinyal gönderiliyor, tüm düğümlere ulaşması için gereken min süreyi bul.

```cpp
int networkDelayTime(vector<vector<int>>& times, int n, int k) {
    unordered_map<int, vector<pair<int,int>>> graph;
    for (auto& t : times) graph[t[0]].push_back({t[1], t[2]});

    unordered_map<int,int> dist;
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
    pq.push({0, k});

    while (!pq.empty()) {
        auto [d, node] = pq.top(); pq.pop();
        if (dist.count(node)) continue;   // zaten kesinleşti
        dist[node] = d;

        for (auto& [neighbor, weight] : graph[node])
            if (!dist.count(neighbor))
                pq.push({d + weight, neighbor});
    }

    if ((int)dist.size() != n) return -1;   // ulaşamayan düğüm var

    int maxTime = 0;
    for (auto& [node, d] : dist) maxTime = max(maxTime, d);
    return maxTime;
}
```

### Adım adım: times=[[2,1,1],[2,3,1],[3,4,1]], n=4, k=2

```
graph: 2→[(1,1),(3,1)], 3→[(4,1)]
pq=[(0,2)]
pop (0,2) → dist={2:0} → komşular: 1(1), 3(1) push
pq=[(1,1),(1,3)]
pop (1,1) → dist={2:0,1:1} → komşusu yok
pq=[(1,3)]
pop (1,3) → dist={2:0,1:1,3:1} → komşu 4(1+1=2) push
pq=[(2,4)]
pop (2,4) → dist={2:0,1:1,3:1,4:2} → komşusu yok

dist.size()=4=n → maxTime = max(0,1,1,2) = 2
```

**Complexity:** TC `O((V+E) log V)`, SC `O(V)`

## Bu mantıkla çözülen benzer problemler
Path With Minimum Effort (grid), Cheapest Flights Within K Stops, minimum cost to reach all nodes.
