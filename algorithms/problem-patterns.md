# DSA Patterns (C++)

Coding interview problemlerinin çoğu birkaç tekrar eden **kalıba (pattern)** oturur. Bu dosyada her kalıp için: **ne zaman kullanılır**, **mantığı**, **şablon kod**, **küçük örnek** ve **çözülecek problemler** var.

> **Nasıl çalışılır:** Bir pattern seç → mantığını oku → 3-5 problem çöz → ertesi gün bakmadan tekrar çöz → çözemediklerini birkaç gün sonra tekrar dene.
> Problem çözerken önce şunu sor: *"Bu hangi kalıp?"*

## İçindekiler

1. [Two Pointers](#1-two-pointers)
2. [Sliding Window](#2-sliding-window)
3. [Frequency Counting (Hash Map / Set)](#3-frequency-counting-hash-map--set)
4. [Prefix Sum](#4-prefix-sum)
5. [Stack ve Monotonic Stack](#5-stack-ve-monotonic-stack)
6. [Binary Search](#6-binary-search)
7. [Binary Tree Traversal](#7-binary-tree-traversal)
8. [Heap / Top K Elements](#8-heap--top-k-elements)
9. [Overlapping Intervals](#9-overlapping-intervals)
10. [Matrix Traversal (Grid DFS/BFS)](#10-matrix-traversal-grid-dfsbfs)
11. [Backtracking](#11-backtracking)
12. [Greedy](#12-greedy)
13. [Union-Find](#13-union-find)
14. [Topological Sort](#14-topological-sort)
15. [Shortest Path (Dijkstra / Bellman-Ford)](#15-shortest-path-dijkstra--bellman-ford)
16. [Dynamic Programming](#16-dynamic-programming)
17. [Pattern Tanıma Tablosu](#17-pattern-tanıma-tablosu)
18. [Sort Hakkında](#18-sort-hakkında)

---

## 1. Two Pointers

**Ne zaman:** Sıralı dizi, palindrom, "çift bulma", dizide yerinde (in-place) düzenleme.

**Mantık:** İki indeks kullanıp O(n²) olacak çift taramayı O(n)'e indirirsin. İki ana türü var:

- **Karşılıklı (uçlardan ortaya):** `l = 0`, `r = n-1`. Sıralı dizide toplam çok büyükse `r--`, küçükse `l++`.
- **Aynı yönde (slow/fast):** Biri okur, diğeri yazar. Örn. diziden elemanları silmek.

**Şablon (karşılıklı) - Two Sum II (sıralı dizi):**
```cpp
vector<int> twoSum(vector<int>& a, int target) {
    int l = 0, r = a.size() - 1;
    while (l < r) {
        int sum = a[l] + a[r];
        if (sum == target) return {l + 1, r + 1};
        if (sum < target) l++;   // toplam küçük -> sol'u büyüt
        else r--;                // toplam büyük -> sağ'ı küçült
    }
    return {};
}
```

**Şablon (aynı yön) - Move Zeroes:**
```cpp
void moveZeroes(vector<int>& a) {
    int w = 0;                       // yazma indeksi
    for (int r = 0; r < a.size(); r++)
        if (a[r] != 0) swap(a[w++], a[r]);
}
```

**Mini örnek:** `[1,2,4,7,11]`, hedef 9 → `l=0(1), r=4(11)` toplam 12 büyük → `r--` → `1+7=8` küçük → `l++` → `2+7=9` ✔.

**Neden çalışır:** Sıralı olduğu için bir tarafı oynatmak toplamı tahmin edilebilir şekilde değiştirir; kaçırdığın çözüm olmaz.

**Problemler:** 125, 167, 283, 11, 15, 42

---

## 2. Sliding Window

**Ne zaman:** "En uzun / en kısa **ardışık** alt dizi / alt string", belli koşulu sağlayan pencere.

**Mantık:** `[l, r]` aralığı bir pencere. `r` sürekli sağa ilerler, pencere bozulursa `l`'yi ilerleterek düzeltirsin. Her eleman en fazla bir kez eklenir, bir kez çıkar → O(n).

**Şablon (değişken pencere) - Longest Substring Without Repeating Characters:**
```cpp
int lengthOfLongestSubstring(string s) {
    unordered_set<char> seen;
    int l = 0, best = 0;
    for (int r = 0; r < s.size(); r++) {
        while (seen.count(s[r])) {       // pencere bozuldu (tekrar var)
            seen.erase(s[l]);            // soldan küçült
            l++;
        }
        seen.insert(s[r]);
        best = max(best, r - l + 1);
    }
    return best;
}
```

**Mini örnek:** `"abcabcbb"` → pencere `abc` (3). `r` ikinci `a`'ya gelince `a` tekrar → `l` ilerler, pencere `bca`... en uzun 3.

**İskelet:**
```
for r in 0..n-1:
    pencereye s[r]'yi ekle
    while pencere geçersiz:
        pencereden s[l]'yi çıkar, l++
    cevabı güncelle
```

**Sabit boyutlu pencere** (örn. "uzunluğu k olan alt dizinin max toplamı"): ilk k elemanı topla, sonra her adımda yeni gireni ekle, çıkanı çıkar.

**Problemler:** 121, 3, 424, 567, 76

---

## 3. Frequency Counting (Hash Map / Set)

**Ne zaman:** "Daha önce gördüm mü?", "kaç tane var?", anagram, tekrar, "x'in tamamlayıcısı var mı?".

**Mantık:** Arama maliyetini O(n)'den O(1)'e düşürmek için hash map/set kullanırsın.

**Şablon - Two Sum (tamamlayıcı arama):**
```cpp
vector<int> twoSum(vector<int>& a, int target) {
    unordered_map<int, int> idx;          // değer -> indeks
    for (int i = 0; i < a.size(); i++) {
        int need = target - a[i];
        if (idx.count(need)) return {idx[need], i};
        idx[a[i]] = i;
    }
    return {};
}
```

**Şablon - Valid Anagram (harf sayma):**
```cpp
bool isAnagram(string s, string t) {
    if (s.size() != t.size()) return false;
    int cnt[26] = {};
    for (char c : s) cnt[c - 'a']++;
    for (char c : t) if (--cnt[c - 'a'] < 0) return false;
    return true;
}
```

**Group Anagrams fikri:** Her kelimeyi sıralayıp (`"eat"` → `"aet"`) anahtar yap, aynı anahtara düşenleri grupla.

**Longest Consecutive Sequence fikri:** Hepsini `unordered_set`'e at. Sadece `x-1` set'te **olmayan** `x`'lerden (dizinin başlangıcı) sayarak ilerle → O(n).

**Problemler:** 217, 242, 1, 49, 347, 128

---

## 4. Prefix Sum

**Ne zaman:** Çok sayıda "i'den j'ye toplam kaç?" sorusu, "toplamı k olan alt dizi".

**Mantık:** `prefix[i]` = ilk i elemanın toplamı. O zaman `sum(l..r) = prefix[r+1] - prefix[l]`. Hazırlık O(n), her sorgu O(1).

**Şablon:**
```cpp
vector<int> prefix(n + 1, 0);
for (int i = 0; i < n; i++) prefix[i + 1] = prefix[i] + a[i];
// sum(l..r) = prefix[r + 1] - prefix[l]
```

**Mini örnek:** `a = [2, 4, 1, 3]` → `prefix = [0, 2, 6, 7, 10]`. `sum(1..2) = prefix[3] - prefix[1] = 7 - 2 = 5` (4+1 ✔).

**Güçlü varyasyon - Subarray Sum Equals K (prefix + hash map):**

Şu ana kadarki toplam `sum` olsun. `sum - k` değerinde bir prefix daha önce görüldüyse, arasındaki alt dizinin toplamı `k`'dır.
```cpp
int subarraySum(vector<int>& a, int k) {
    unordered_map<int, int> cnt = {{0, 1}};   // boş prefix
    int sum = 0, res = 0;
    for (int x : a) {
        sum += x;
        if (cnt.count(sum - k)) res += cnt[sum - k];
        cnt[sum]++;
    }
    return res;
}
```

**Product of Array Except Self fikri:** Aynı mantık çarpma ile: her indeks için "solundakilerin çarpımı" × "sağındakilerin çarpımı".

**Problemler:** 303, 724, 238, 560

---

## 5. Stack ve Monotonic Stack

**Ne zaman:**
- Stack: parantez eşleştirme, iç içe yapılar, "son giren ilk çıkar".
- Monotonic stack: **"bir sonraki daha büyük / daha küçük eleman"** soruları.

**Valid Parentheses:**
```cpp
bool isValid(string s) {
    stack<char> st;
    for (char c : s) {
        if (c == '(' || c == '[' || c == '{') st.push(c);
        else {
            if (st.empty()) return false;
            char t = st.top(); st.pop();
            if ((c == ')' && t != '(') || (c == ']' && t != '[') || (c == '}' && t != '{'))
                return false;
        }
    }
    return st.empty();
}
```

**Monotonic Stack mantığı:** Stack'te indeksleri tut, değerler hep azalan sırada olsun. Yeni eleman stack'in tepesinden büyükse, tepedekilerin "bir sonraki büyüğü" bu elemandır → hepsini çıkar ve cevabı yaz. Her eleman bir kez girer bir kez çıkar → O(n).

**Daily Temperatures (kaç gün sonra daha sıcak?):**
```cpp
vector<int> dailyTemperatures(vector<int>& t) {
    int n = t.size();
    vector<int> res(n, 0);
    stack<int> st;                         // indeksler
    for (int i = 0; i < n; i++) {
        while (!st.empty() && t[i] > t[st.top()]) {
            int j = st.top(); st.pop();
            res[j] = i - j;                // j için cevap bulundu
        }
        st.push(i);
    }
    return res;
}
```

**Mini örnek:** `[73, 74, 75, 71, 69, 72, 76]` → `[1, 1, 4, 2, 1, 1, 0]`.

**Problemler:** 20, 155, 150, 739, 496, 853, 84

---

## 6. Binary Search

**Ne zaman:** Sıralı dizi, **veya** cevabın "sağlıyor / sağlamıyor" diye ikiye bölündüğü her durum.

**Klasik şablon:**
```cpp
int binarySearch(vector<int>& a, int target) {
    int l = 0, r = a.size() - 1;
    while (l <= r) {
        int mid = l + (r - l) / 2;         // taşmayı önler
        if (a[mid] == target) return mid;
        if (a[mid] < target) l = mid + 1;
        else r = mid - 1;
    }
    return -1;
}
```

**Asıl güçlü hali: Cevap üzerinde binary search**

Cevap bir sayı aralığında ve fonksiyon monotonsa (`false false false true true true`), ilk `true`'yu bul.

**Koko Eating Bananas (en küçük hız k, öyle ki h saatte bitsin):**
```cpp
int minEatingSpeed(vector<int>& piles, int h) {
    int l = 1, r = *max_element(piles.begin(), piles.end());
    auto canFinish = [&](int k) {
        long long hours = 0;
        for (int p : piles) hours += (p + k - 1) / k;   // yukarı yuvarla
        return hours <= h;
    };
    while (l < r) {
        int mid = l + (r - l) / 2;
        if (canFinish(mid)) r = mid;       // mid çalışıyor, daha küçüğünü dene
        else l = mid + 1;
    }
    return l;
}
```

**Tanıma ipucu:** "En küçük ... öyle ki", "en büyük ... öyle ki" cümleleri.

**Rotated sorted array:** Her adımda `mid`'in hangi yarıda olduğuna (hangi yarı sıralı?) bakıp hedef o yarıdaysa oraya git.

**Problemler:** 704, 74, 875, 153, 33, 981

---

## 7. Binary Tree Traversal

**Ne zaman:** Ağaçla ilgili her şey.

**Mantık:** Üç DFS sırası, "düğümü işle" satırının yeri dışında aynıdır. BFS (level order) ise queue ile seviye seviye gider.

```cpp
void preorder(TreeNode* n)  { if (!n) return; visit(n); preorder(n->left); preorder(n->right); }
void inorder(TreeNode* n)   { if (!n) return; inorder(n->left); visit(n); inorder(n->right); }
void postorder(TreeNode* n) { if (!n) return; postorder(n->left); postorder(n->right); visit(n); }
```

Örnek ağaç:
```
          1
        /   \
       2     3
      / \     \
     4   5     6
        /     / \
       7     8   9
```
- Preorder: `1 2 4 5 7 3 6 8 9`
- Inorder: `4 2 7 5 1 3 8 6 9` (BST'de sıralı çıkar)
- Postorder: `4 7 5 2 8 9 6 3 1`

**Hangisi ne zaman:**
- **Preorder:** Önce düğümü görmem lazım (kopyalama, serialize, path).
- **Inorder:** BST'de sıralı değerler (Kth Smallest, Validate BST).
- **Postorder:** Çocuklardan bilgi toplayıp yukarı taşıma (yükseklik, diameter).

**Iterative Inorder (stack ile) - BST Iterator'ın mantığı:**
```cpp
stack<TreeNode*> st;
auto cur = root;
while (cur || !st.empty()) {
    while (cur) { st.push(cur); cur = cur->left; }   // full sola in
    cur = st.top(); st.pop();
    visit(cur);                                      // en soldaki = en küçük
    cur = cur->right;                                // sağın full soluna geç
}
```

**Iterative Preorder:**
```cpp
stack<TreeNode*> st; st.push(root);
while (!st.empty()) {
    auto n = st.top(); st.pop();
    if (!n) continue;
    visit(n);
    st.push(n->right);   // sağ önce, sol üstte kalsın
    st.push(n->left);
}
```

**Iterative Postorder (hile):** "düğüm, sağ, sol" sırasıyla gez (preorder'ın aynası), sonucu ters çevir.
```cpp
vector<int> res; stack<TreeNode*> st; st.push(root);
while (!st.empty()) {
    auto n = st.top(); st.pop();
    if (!n) continue;
    res.push_back(n->val);
    st.push(n->left);    // sol önce, sağ üstte kalsın
    st.push(n->right);
}
reverse(res.begin(), res.end());
```

**Level Order (BFS):**
```cpp
vector<vector<int>> levelOrder(TreeNode* root) {
    vector<vector<int>> res;
    if (!root) return res;
    queue<TreeNode*> q; q.push(root);
    while (!q.empty()) {
        int sz = q.size();                 // bu seviyedeki düğüm sayısı
        vector<int> level;
        while (sz--) {
            auto n = q.front(); q.pop();
            level.push_back(n->val);
            if (n->left) q.push(n->left);
            if (n->right) q.push(n->right);
        }
        res.push_back(level);
    }
    return res;
}
```
Right Side View = her seviyenin son elemanı. Zigzag = her ikinci seviyeyi ters çevir.

**"Alttan yukarı bilgi" kalıbı (postorder mantığı) - Max Depth & Diameter:**
```cpp
int maxDepth(TreeNode* n) {
    if (!n) return 0;
    return 1 + max(maxDepth(n->left), maxDepth(n->right));
}

int best = 0;
int height(TreeNode* n) {
    if (!n) return 0;
    int l = height(n->left), r = height(n->right);
    best = max(best, l + r);               // bu düğümden geçen en uzun yol
    return 1 + max(l, r);                  // yukarıya sadece tek kol döner
}
```
**İpucu:** Fonksiyon yukarıya **bir şey döndürür**, ama cevabı **global/yan değişkende** güncelleyebilir. Diameter, Max Path Sum gibi problemlerde bu ayrım kilit.

**Validate BST:** Her düğüm için izinli `(min, max)` aralığı taşı; sola giderken `max` düğümün değeri olur, sağa giderken `min`.

**Problemler:** 94, 144, 145, 102, 104, 199, 543, 98, 230, 173

---

## 8. Heap / Top K Elements

**Ne zaman:** "K'ıncı en büyük/küçük", "en sık K eleman", "sürekli en küçüğü/büyüğü al".

**Mantık:** `priority_queue` ile en büyük/küçüğe O(log n)'de ulaşırsın. **Top K büyük** için **boyutu K olan min-heap** tut: boyut K'yı aşarsa en küçüğü at. Sonunda heap'in tepesi K'ıncı en büyük eleman.

```cpp
int findKthLargest(vector<int>& a, int k) {
    priority_queue<int, vector<int>, greater<int>> pq;   // min-heap
    for (int x : a) {
        pq.push(x);
        if (pq.size() > k) pq.pop();       // en küçüğü at
    }
    return pq.top();
}
```

**Neden min-heap?** K büyükten fazlasını tutmak istemiyorsun; her seferinde en küçük aday elenmeli. Bellek O(k), süre O(n log k).

**Top K Frequent:** Önce frekansı hash map ile say, sonra `(frekans, değer)` çiftlerini aynı şekilde heap'le (veya bucket sort ile O(n)).

**Not:** C++ `priority_queue` varsayılan olarak **max-heap**'tir. Min-heap için `greater<int>`.

**Problemler:** 703, 1046, 973, 215, 621

---

## 9. Overlapping Intervals

**Ne zaman:** Aralıklar, takvim, "çakışan", "birleştir", "en az kaç tane çıkar".

**Mantık:** **Başlangıca göre sırala**, sonra tek geçişte son eklenen aralıkla karşılaştır. Çakışma koşulu: `son.end >= yeni.start`.

**Merge Intervals:**
```cpp
vector<vector<int>> merge(vector<vector<int>>& iv) {
    sort(iv.begin(), iv.end());
    vector<vector<int>> res;
    for (auto& cur : iv) {
        if (res.empty() || res.back()[1] < cur[0])
            res.push_back(cur);                        // çakışmıyor
        else
            res.back()[1] = max(res.back()[1], cur[1]); // birleştir
    }
    return res;
}
```

**Mini örnek:** `[[1,3],[2,6],[8,10]]` → `[1,3]` ve `[2,6]` çakışır → `[1,6]`, `[8,10]` ayrı → `[[1,6],[8,10]]`.

**Non-overlapping Intervals / Arrows (greedy varyasyonu):** **Bitişe göre** sırala; her zaman en erken biteni tut, çakışanları say.

**Problemler:** 56, 57, 435, 452

---

## 10. Matrix Traversal (Grid DFS/BFS)

**Ne zaman:** Grid, ada sayma, labirent, "bölge", "en kısa yol (ağırlıksız)".

**Mantık:** Grid bir graph'tır: her hücre bir düğüm, 4 komşu kenar. **Bağlı bölge/ada sayma → DFS veya BFS**, **en kısa yol → BFS**.

**Number of Islands (DFS):**
```cpp
int dx[] = {1, -1, 0, 0}, dy[] = {0, 0, 1, -1};

void dfs(vector<vector<char>>& g, int x, int y) {
    if (x < 0 || y < 0 || x >= g.size() || y >= g[0].size() || g[x][y] != '1') return;
    g[x][y] = '0';                               // ziyaret edildi
    for (int d = 0; d < 4; d++) dfs(g, x + dx[d], y + dy[d]);
}

int numIslands(vector<vector<char>>& g) {
    int cnt = 0;
    for (int i = 0; i < g.size(); i++)
        for (int j = 0; j < g[0].size(); j++)
            if (g[i][j] == '1') { cnt++; dfs(g, i, j); }
    return cnt;
}
```

**Multi-source BFS (Rotting Oranges):** Tüm başlangıç noktalarını **aynı anda** queue'ya at, seviye seviye ilerle; her seviye = 1 dakika.
```cpp
queue<pair<int,int>> q;
// tüm çürük portakalları q'ya at
int minutes = 0;
while (!q.empty()) {
    int sz = q.size();
    bool spread = false;
    while (sz--) {
        auto [x, y] = q.front(); q.pop();
        for (int d = 0; d < 4; d++) {
            int nx = x + dx[d], ny = y + dy[d];
            if (nx >= 0 && ny >= 0 && nx < n && ny < m && g[nx][ny] == 1) {
                g[nx][ny] = 2; q.push({nx, ny}); spread = true;
            }
        }
    }
    if (spread) minutes++;
}
```

**Ters düşünme (Pacific Atlantic):** Her hücreden okyanusa gitmek yerine, **okyanustan başlayıp** yukarı doğru gez.

**En kısa yol:** Ağırlıksız grid → BFS. Ağırlıklı (örn. Path With Minimum Effort) → Dijkstra.

**Problemler:** 200, 695, 994, 417, 130, 1091

---

## 11. Backtracking

**Ne zaman:** "Tüm olası kombinasyonlar / permütasyonlar / alt kümeler / yerleşimler".

**Mantık:** Karar ağacında DFS: **seç → ilerle → geri al (undo)**. Her seviyede bir karar verirsin, bitince eski haline dönersin.

**Genel iskelet:**
```
void bt(durum):
    if (bitiş koşulu): sonucu kaydet; return
    for (seçenek in seçenekler):
        if (geçersiz) continue
        seç
        bt(yeni durum)
        geri al
```

**Subsets:**
```cpp
vector<vector<int>> res;
vector<int> cur;

void bt(vector<int>& a, int start) {
    res.push_back(cur);                    // her durum bir alt küme
    for (int i = start; i < a.size(); i++) {
        cur.push_back(a[i]);               // seç
        bt(a, i + 1);                      // sonraki elemanlardan devam
        cur.pop_back();                    // geri al
    }
}
```
`[1,2,3]` → `[] [1] [1,2] [1,2,3] [1,3] [2] [2,3] [3]`

**Permutations (kullanıldı dizisiyle):**
```cpp
void bt(vector<int>& a, vector<bool>& used) {
    if (cur.size() == a.size()) { res.push_back(cur); return; }
    for (int i = 0; i < a.size(); i++) {
        if (used[i]) continue;
        used[i] = true; cur.push_back(a[i]);
        bt(a, used);
        cur.pop_back(); used[i] = false;
    }
}
```

**Kombinasyon vs permütasyon farkı:** Kombinasyonda sıra önemsiz → `start` indeksiyle sadece ileri bak. Permütasyonda sıra önemli → her seferinde baştan tara, `used` ile kullanılanı atla.

**Combination Sum (aynı eleman tekrar kullanılabilir):** Özyinelemede `i + 1` yerine `i` ver; kalan hedef `< 0` ise dur.

**Tekrarlı elemanlar (Subsets II):** Önce sırala; aynı seviyede `a[i] == a[i-1]` ise (`i > start`) atla.

**Problemler:** 78, 46, 39, 90, 79, 131, 51

---

## 12. Greedy

**Ne zaman:** Her adımda **yerel olarak en iyi** seçimin global en iyiyi verdiği problemler. Çoğu zaman sıralama gerektirir.

**Mantık:** Geri dönmeden, her adımda en mantıklı seçimi yaparsın. Zor kısım **doğru seçim kuralını** bulmak ve doğruluğunu ikna edici şekilde savunmak.

**Jump Game (en uzağa ulaşılabilen indeksi takip et):**
```cpp
bool canJump(vector<int>& a) {
    int reach = 0;
    for (int i = 0; i < a.size(); i++) {
        if (i > reach) return false;       // oraya hiç ulaşılamıyor
        reach = max(reach, i + a[i]);
    }
    return true;
}
```

**Maximum Subarray (Kadane):**
```cpp
int maxSubArray(vector<int>& a) {
    int cur = a[0], best = a[0];
    for (int i = 1; i < a.size(); i++) {
        cur = max(a[i], cur + a[i]);       // önceki toplam faydasızsa yeniden başla
        best = max(best, cur);
    }
    return best;
}
```

**Tanıma ipuçları:** "Minimum sayıda ...", "maksimum sayıda ...", aralık/zaman çizelgesi → **sırala**, sonra tek geçişle seç.

**Dikkat:** Greedy her problemde çalışmaz. Çalışmayan örnek: farklı coin değerleriyle Coin Change (DP gerekir).

**Problemler:** 53, 55, 45, 134, 763, 846

---

## 13. Union-Find

**Ne zaman:** "Bağlı bileşen sayısı", "iki eleman aynı grupta mı?", dinamik gruplama, döngü tespiti (yönsüz graph).

**Mantık:** Her grubun bir temsilcisi (root) var. `find` kökü bulur, `unite` iki grubu birleştirir. **Path compression** ile neredeyse O(1).

```cpp
struct DSU {
    vector<int> p, sz;
    DSU(int n) : p(n), sz(n, 1) { iota(p.begin(), p.end(), 0); }
    int find(int x) { return p[x] == x ? x : p[x] = find(p[x]); }   // path compression
    bool unite(int a, int b) {
        a = find(a); b = find(b);
        if (a == b) return false;           // zaten aynı grup (döngü!)
        if (sz[a] < sz[b]) swap(a, b);
        p[b] = a; sz[a] += sz[b];
        return true;
    }
};
```

**Kullanım:**
- **Bileşen sayısı:** Başlangıçta `n`, her başarılı `unite`'ta bir azalt.
- **Redundant Connection:** Kenarları sırayla ekle; `unite` false dönerse o kenar döngü yaratıyor.
- **Min Cost to Connect All Points (Kruskal):** Kenarları ağırlığa göre sırala, `unite` başarılıysa ekle.

**Problemler:** 547, 684, 721, 1584

---

## 14. Topological Sort

**Ne zaman:** Bağımlılık sırası, "önce X sonra Y", ders programı, **yönlü graph'ta döngü var mı?**

**Mantık (Kahn algoritması, BFS):**
1. Her düğümün **gelen kenar sayısını (indegree)** hesapla.
2. Indegree'si 0 olanları queue'ya at.
3. Birini çıkar, sıraya ekle, komşularının indegree'sini azalt; 0 olanı queue'ya at.
4. Sıraya eklenen düğüm sayısı `n`'e eşit değilse **döngü var**.

```cpp
// Course Schedule II
vector<int> findOrder(int n, vector<vector<int>>& pre) {
    vector<vector<int>> adj(n);
    vector<int> indeg(n, 0);
    for (auto& e : pre) {                  // e = {ders, önkoşul}
        adj[e[1]].push_back(e[0]);
        indeg[e[0]]++;
    }
    queue<int> q;
    for (int i = 0; i < n; i++) if (indeg[i] == 0) q.push(i);

    vector<int> order;
    while (!q.empty()) {
        int u = q.front(); q.pop();
        order.push_back(u);
        for (int v : adj[u])
            if (--indeg[v] == 0) q.push(v);
    }
    return order.size() == n ? order : vector<int>{};   // döngü varsa boş
}
```

**Problemler:** 207, 210 (ve Graph genel: 133)

---

## 15. Shortest Path (Dijkstra / Bellman-Ford)

Zaten yazdığın algoritmalar. Ne zaman hangisi:

| Durum | Algoritma |
|---|---|
| Ağırlıksız graph/grid | BFS |
| Pozitif ağırlıklar | Dijkstra |
| Negatif ağırlık ya da "en fazla K kenar" kısıtı | Bellman-Ford |
| Hedef biliniyor, heuristic var | A* (LeetCode'da nadir) |

**Dijkstra iskeleti (min-heap):**
```cpp
vector<int> dist(n, INT_MAX);
priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;   // {mesafe, düğüm}
dist[src] = 0; pq.push({0, src});
while (!pq.empty()) {
    auto [d, u] = pq.top(); pq.pop();
    if (d > dist[u]) continue;             // eski kayıt
    for (auto [v, w] : adj[u])
        if (dist[u] + w < dist[v]) {
            dist[v] = dist[u] + w;
            pq.push({dist[v], v});
        }
}
```

**Cheapest Flights Within K Stops:** Bellman-Ford'u tam `K+1` tur çalıştır; her turda **bir önceki turun** dizisinden güncelle (kopya al), yoksa aynı turda birden fazla kenar kullanılır.

**Problemler:** 743, 787, 1631

---

## 16. Dynamic Programming

**Ne zaman:**
- "Kaç farklı yol var?", "minimum/maksimum maliyet", "mümkün mü?"
- Problem **alt problemlere** bölünebiliyor ve alt problemler **tekrar ediyor** (overlapping subproblems).
- Brute force (recursion) üstel çalışıyor.

**DP çözme reçetesi (her problemde aynı 4 adım):**
1. **Durum (state):** `dp[i]` neyi ifade ediyor? Bunu cümleyle yaz.
2. **Geçiş (transition):** `dp[i]` önceki durumlardan nasıl hesaplanır?
3. **Başlangıç (base case):** En küçük durumlar neler?
4. **Sıra:** Hangi sırayla doldurulur? Cevap nerede?

**İpucu:** Önce brute-force recursion yaz → tekrar eden çağrıları bul → memoization ekle → istersen tabloya (bottom-up) çevir.

### 1D DP

**Climbing Stairs** (`dp[i]` = i. basamağa gelme yolu sayısı, `dp[i] = dp[i-1] + dp[i-2]`):
```cpp
int climbStairs(int n) {
    int a = 1, b = 1;                      // dp[0], dp[1]
    for (int i = 2; i <= n; i++) { int c = a + b; a = b; b = c; }
    return b;
}
```

**House Robber** (`dp[i]` = ilk i evden max para; ya i'yi soy ya soyma):
`dp[i] = max(dp[i-1], dp[i-2] + a[i])`
```cpp
int rob(vector<int>& a) {
    int prev2 = 0, prev1 = 0;
    for (int x : a) {
        int cur = max(prev1, prev2 + x);
        prev2 = prev1; prev1 = cur;
    }
    return prev1;
}
```

**Coin Change** (`dp[a]` = a tutarı için min coin sayısı):
```cpp
int coinChange(vector<int>& coins, int amount) {
    vector<int> dp(amount + 1, amount + 1);   // "sonsuz"
    dp[0] = 0;
    for (int a = 1; a <= amount; a++)
        for (int c : coins)
            if (a >= c) dp[a] = min(dp[a], dp[a - c] + 1);
    return dp[amount] > amount ? -1 : dp[amount];
}
```

**LIS (Longest Increasing Subsequence):** `dp[i]` = i'de biten en uzun artan alt dizi. `dp[i] = 1 + max(dp[j])` (j < i ve a[j] < a[i]). O(n²); binary search ile O(n log n) var.

**Word Break:** `dp[i]` = ilk i karakter sözlük kelimeleriyle bölünebilir mi?

### 2D DP

**Unique Paths** (`dp[i][j]` = (i,j)'ye gelme yolu sayısı, `dp[i][j] = dp[i-1][j] + dp[i][j-1]`):
```cpp
int uniquePaths(int m, int n) {
    vector<vector<int>> dp(m, vector<int>(n, 1));
    for (int i = 1; i < m; i++)
        for (int j = 1; j < n; j++)
            dp[i][j] = dp[i-1][j] + dp[i][j-1];
    return dp[m-1][n-1];
}
```

**Longest Common Subsequence** (`dp[i][j]` = s1'in ilk i, s2'nin ilk j karakterinin LCS'i):
```cpp
int lcs(string a, string b) {
    int n = a.size(), m = b.size();
    vector<vector<int>> dp(n + 1, vector<int>(m + 1, 0));
    for (int i = 1; i <= n; i++)
        for (int j = 1; j <= m; j++)
            dp[i][j] = (a[i-1] == b[j-1])
                       ? dp[i-1][j-1] + 1                  // eşleşti
                       : max(dp[i-1][j], dp[i][j-1]);      // birini atla
    return dp[n][m];
}
```

**0/1 Knapsack (Partition Equal Subset Sum):** "Toplamı `S/2` olan alt küme var mı?" `dp[s]` = s toplamı yapılabilir mi? Her eleman için `s`'yi **büyükten küçüğe** tara (aynı elemanı iki kez kullanmamak için).
```cpp
bool canPartition(vector<int>& a) {
    int total = accumulate(a.begin(), a.end(), 0);
    if (total % 2) return false;
    int target = total / 2;
    vector<bool> dp(target + 1, false);
    dp[0] = true;
    for (int x : a)
        for (int s = target; s >= x; s--)
            dp[s] = dp[s] || dp[s - x];
    return dp[target];
}
```

**Coin Change II (kombinasyon sayısı):** Dış döngü **coin**, iç döngü **tutar** olursa sıra önemsiz sayılır (kombinasyon). Ters olursa permütasyon sayarsın.

**Problemler:** 70, 746, 198, 213, 322, 300, 139, 91, 62, 1143, 518, 416, 494

---

## 17. Pattern Tanıma Tablosu

Problemi okuyunca şu ipuçlarına bak:

| Problemdeki ipucu | Muhtemel pattern |
|---|---|
| Sıralı dizi, çift bul, palindrom | Two Pointers |
| En uzun/kısa **ardışık** alt dizi/string | Sliding Window |
| "Daha önce gördüm mü", sayma, anagram | Hash Map / Set |
| Alt dizi toplamı, aralık toplamı sorguları | Prefix Sum |
| Parantez, iç içe yapı | Stack |
| "Bir sonraki daha büyük/küçük" | Monotonic Stack |
| Sıralı, "en küçük ... öyle ki" | Binary Search |
| Ağaç, seviye seviye | BFS (queue) |
| Ağaç, alttan yukarı bilgi | Postorder DFS |
| K'ıncı en büyük/küçük, en sık K | Heap |
| Aralıklar, takvim | Sort + Intervals |
| Grid, ada, bölge | Grid DFS/BFS |
| Grid/graph'ta en kısa yol (ağırlıksız) | BFS |
| Ağırlıklı en kısa yol | Dijkstra |
| Tüm kombinasyon/permütasyon/alt küme | Backtracking |
| "Minimum sayıda", yerel seçim yeterli | Greedy |
| Gruplama, bağlı bileşen, döngü (yönsüz) | Union-Find |
| Bağımlılık sırası, yönlü döngü | Topological Sort |
| "Kaç yol var", min/max maliyet, alt problemler tekrar ediyor | DP |

---

## 18. Sort Hakkında

- **Çoğu problemde `sort()` yeterli.** Algoritmayı sıfırdan yazman gereken problem neredeyse hiç çıkmaz.
- Ama **fikirleri** bilmen gerekir:
  - **Merge sort:** Linked list sıralama, inversion sayma.
  - **Quickselect:** Kth Largest Element (ortalama O(n)).
  - **Counting / bucket sort:** Top K Frequent, Sort Colors.
  - **Heap:** Top K, K sıralı listeyi birleştirme.
- Özel sıralama için lambda:
```cpp
sort(v.begin(), v.end(), [](auto& a, auto& b) { return a[1] < b[1]; });
```

---

## Son Not

- Hepsini ezberleme; **mantığı anla**, sonra problemle pekiştir.
- Çözemediğin problemi 20-30 dk düşündükten sonra çözüme bak, **ertesi gün bakmadan yeniden çöz.**
- Kendine bir "çözemediklerim" listesi tut ve birkaç gün sonra tekrar dene.
- Amaç problemleri ezberlemek değil, **bilmediğin bir problemde doğru kalıbı tanımak.**
