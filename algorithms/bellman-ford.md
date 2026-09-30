# Bellman-Ford Algorithm

## When to use
- Ağırlıklı graf, **negatif ağırlıklar olabilir**, tek bir başlangıçtan tüm düğümlere en kısa mesafe lazım.
- Negatif **döngü** var mı, onu da tespit eder.
- Negatif ağırlık yoksa → Dijkstra daha hızlı. Ağırlıklar hep 1 ise → BFS yeter.
- "En fazla k kenar / k aktarma" kısıtı varsa doğal seçim budur.

## Mantık (tek fonksiyon)
Dijkstra **açgözlü**: "en küçük mesafeliyi çıkar, kesinleşti say". Negatif kenar varsa bu varsayım çöker, çünkü kesinleştirdiğin düğüme sonradan negatif kenarla daha ucuz gelinebilir.

Bellman-Ford seçim yapmaz, **her turda bütün kenarları gevşetir (relax)**. Bunu en fazla **n tur** tekrarlar.

- **Neden n-1 tur?** Döngüsüz en kısa yol en fazla n-1 kenar içerir. **i. turun sonunda, en fazla i kenarlı tüm en kısa yollar doğru bulunmuş olur.**
- **n. turda hâlâ bir şey değişiyorsa** negatif döngü vardır (döngü her dönüşte mesafeyi düşürür, hiç durmaz).

Ne heap var, ne kuyruk, ne `visited`: sadece `dist` dizisi ve kenar listesi. `dist` hem "en iyi mesafe" hem de "buraya ulaşıldı mı" bilgisini taşır (`INT_MAX` = henüz ulaşılmadı).

```cpp
// edges: {u, v, w}, düğümler 0..n-1
// dönüş: {dist, negatifDöngüVarMı}
pair<vector<int>, bool> bellmanFord(int n, vector<array<int,3>>& edges, int start) {
    vector<int> dist(n, INT_MAX);
    dist[start] = 0;

    for (int i = 0; i < n; ++i) {                    // n tur (n-1 + döngü kontrolü)
        bool changed = false;
        for (auto& [u, v, w] : edges)
            if (dist[u] != INT_MAX && dist[u] + w < dist[v]) {   // gevşet
                dist[v] = dist[u] + w;
                changed = true;
            }
        if (!changed) return {dist, false};          // oturdu, negatif döngü yok
    }
    return {dist, true};                             // n. turda bile değişti → negatif döngü
}
```

### Trickler (sadeleştirmeler)
- **Ayrı "n. tur kontrolü" döngüsü yok.** Tek döngüyü n kez çalıştırıp "hiç değişmeyen tur" görürsek erken çıkıyoruz. n turun hepsinde değişiklik olduysa negatif döngü var. Klasik anlatımdaki ikinci döngüye gerek kalmıyor.
- **`changed` bayrağı hem hız hem tespit sağlar.** Kenar sırası iyiyse 1-2 turda biter.
- **`dist[u] != INT_MAX` şart.** Yoksa `INT_MAX + w` taşar. Dijkstra'daki "yoksa ekle" kontrolünün karşılığı, ayrı `visited` gerekmez.
- Negatif döngü tespiti sadece **start'tan ulaşılabilen** döngüler için geçerli.

### Adım adım örnek
n=4, start=0, kenarlar bu sırayla: `(1,3,2), (2,1,-3), (0,2,5), (0,1,4)`

```
başlangıç: dist=[0, INF, INF, INF]

Tur 1:
  (1→3, 2):  dist[1]=INF → atla
  (2→1,-3):  dist[2]=INF → atla
  (0→2, 5):  0+5=5 < INF → dist[2]=5
  (0→1, 4):  0+4=4 < INF → dist[1]=4
  dist=[0, 4, 5, INF]   changed=true

Tur 2:
  (1→3, 2):  4+2=6 < INF → dist[3]=6
  (2→1,-3):  5-3=2 < 4   → dist[1]=2      ← negatif kenar, Dijkstra burada yanılırdı
  (0→2, 5):  5 < 5 değil
  (0→1, 4):  4 < 2 değil
  dist=[0, 2, 5, 6]     changed=true

Tur 3:
  (1→3, 2):  2+2=4 < 6   → dist[3]=4
  dist=[0, 2, 5, 4]     changed=true

Tur 4 (n. tur):
  hiçbir kenar gevşemiyor → changed=false → return {dist, false}

Sonuç: dist = [0, 2, 5, 4], negatif döngü yok
```

Kenar sırası kötü olduğu için doğru sonuca 3 turda ulaştı. Sıra iyi olsaydı (`0→1, 0→2, 2→1, 1→3`) tek turda biterdi.

### Negatif döngü örneği
n=4, start=0, kenarlar: `(0,1,1), (1,2,1), (2,3,-3), (3,1,1)`. Döngü 1→2→3→1'in toplamı 1-3+1 = **-1**.

```
Tur 1: dist[1]=1, dist[2]=2, dist[3]=-1, sonra 3→1: -1+1=0 < 1 → dist[1]=0   changed
Tur 2: dist[2]=1, dist[3]=-2, dist[1]=-1                                       changed
Tur 3: her şey 1 daha düştü                                                    changed
Tur 4: yine düştü                                                              changed

4 turun hepsinde değişti → return {dist, true}
```

Her dönüşte mesafe 1 azalıyor, yani "en kısa yol" tanımsız.

---

## Dijkstra ile fark (ezber için)

| | Dijkstra | Bellman-Ford |
|---|---|---|
| Strateji | Açgözlü, en küçüğü seç ve kesinleştir | Her turda tüm kenarları gevşet |
| Veri yapısı | Min-heap | Yok, sadece kenar listesi |
| Negatif kenar | ✗ | ✓ |
| Negatif döngü tespiti | ✗ | ✓ (son tur hâlâ değişiyorsa) |
| `visited` | Gerekmez (`d > dist[node]` yeter) | Yok |
| TC | O((V+E) log V) | **O(V·E)** |

Aklında tutacağın tek cümle: **"Tüm kenarları gevşet, en fazla n tur, hâlâ değişiyorsa negatif döngü."**

**SPFA** bunun kuyruklu optimizasyonu: sadece mesafesi değişen düğümlerin kenarlarını tekrar gevşetir. Ortalamada hızlı, en kötü durumu yine O(V·E).

---

## Örnek Problem: Cheapest Flights Within K Stops

`src`'den `dst`'ye **en fazla k aktarmayla** en ucuz fiyatı bul.

Burada "i. tur = en fazla i kenar" özelliği doğrudan kullanılıyor. k aktarma = **k+1 kenar** = k+1 tur.

```cpp
int findCheapestPrice(int n, vector<vector<int>>& flights, int src, int dst, int k) {
    vector<int> dist(n, INT_MAX);
    dist[src] = 0;

    for (int i = 0; i <= k; ++i) {           // k+1 tur
        vector<int> tmp = dist;              // önceki turun kopyası
        for (auto& f : flights) {
            int u = f[0], v = f[1], w = f[2];
            if (dist[u] != INT_MAX && dist[u] + w < tmp[v])
                tmp[v] = dist[u] + w;        // oku: dist, yaz: tmp
        }
        dist = tmp;
    }
    return dist[dst] == INT_MAX ? -1 : dist[dst];
}
```

**Neden `tmp` kopyası?** Aynı turda güncellenen değeri aynı turda tekrar kullanırsan bir turda birden fazla kenar ilerlemiş olursun ve aktarma sınırı bozulur. Okumayı eski `dist`'ten, yazmayı `tmp`'ye yapınca her tur tam **1 kenar** ilerletir. Normal Bellman-Ford'da kopya gerekmez, çünkü orada fazla ilerlemek zararsız (sadece daha hızlı yakınsar).

### Adım adım: n=3, flights=[[0,1,100],[1,2,100],[0,2,500]], src=0, dst=2, k=1

```
dist=[0, INF, INF]

Tur 1 (i=0), tmp=[0,INF,INF]:
  0→1 (100): tmp[1]=100
  1→2 (100): dist[1]=INF → atla
  0→2 (500): tmp[2]=500
  dist=[0, 100, 500]

Tur 2 (i=1), tmp=[0,100,500]:
  0→1 (100): 100 < 100 değil
  1→2 (100): dist[1]+100=200 < 500 → tmp[2]=200
  0→2 (500): 500 < 200 değil
  dist=[0, 100, 200]

Sonuç: 200
```

`k=0` olsaydı tek tur yapılır, sonuç `500` kalırdı. Kopya kullanmasaydık ve kenar sırası uygunsa tek turda 0→1→2 ile 200 bulurduk, ki bu k=0 için yanlış.

**Complexity:** TC `O(V·E)` (burada O(k·E)), SC `O(V)`

## Bu mantıkla çözülen benzer problemler
Network Delay Time (negatif yok ama Bellman-Ford ile de çözülür), Negative Cycle Detection, Arbitrage (döviz kurları, `-log` ile negatif döngü aramaya çevrilir).
