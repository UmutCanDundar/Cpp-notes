# Kombinasyon (Combinations)

## When to use
- Problem "**seçim/alt küme**" diyor: [1,2] ve [2,1] AYNI sayılıyor (sıra önemsiz).
- "n elemandan k tanesini seç", "tüm alt kümeler", "toplamı X olan grupları bul" tarzı sorular.

## Mantık (tek fonksiyon)
Her eleman için sadece 2 seçenek var: **onu alıyor musun almıyor musun**. Recursion her elemanda
bu kararı verir, sona gelince elde olan listeyi kaydeder. Permütasyondan farkı: burada "hangi
sırayla" sorusu yok, sadece "al mı, alma mı" var — bu yüzden aynı eleman kümesi tek yoldan üretilir.

```cpp
void solve(vector<int>& nums, int index, vector<int>& current, vector<vector<int>>& result) {
    if (index == nums.size()) {         // tüm elemanları gezdik
        result.push_back(current);       // elde ettiğimiz kombinasyonu kaydet
        return;
    }

    // Seçenek 1: bu elemanı ALMA
    solve(nums, index + 1, current, result);

    // Seçenek 2: bu elemanı AL
    current.push_back(nums[index]);
    solve(nums, index + 1, current, result);
    current.pop_back();   // geri al
}
```

### Adım adım örnek: nums = [1, 2]

```
solve(index=0, current=[])
│
├── AL-MA (1'i almadan devam)
│   solve(index=1, current=[])
│   │
│   ├── AL-MA (2'yi almadan devam)
│   │   solve(index=2, current=[])  → index==size → kaydet: []
│   │
│   └── AL (2'yi al)
│       current=[2]
│       solve(index=2, current=[2]) → kaydet: [2]
│       geri al: current=[]
│
└── AL (1'i al)
    current=[1]
    solve(index=1, current=[1])
    │
    ├── AL-MA (2'yi almadan devam)
    │   solve(index=2, current=[1]) → kaydet: [1]
    │
    └── AL (2'yi al)
        current=[1,2]
        solve(index=2, current=[1,2]) → kaydet: [1,2]
        geri al: current=[1]
    geri al: current=[]

Sonuç: [], [2], [1], [1,2]  → [1,2]'nin tüm alt kümeleri (power set)
```

---

## Sadece k boyutunda kombinasyon istiyorsan

Yukarıdaki fonksiyona tek şart ekle: `current.size() == k` olunca kaydet, `k`'den büyükse dala hiç girme.

```cpp
void solve(vector<int>& nums, int index, int k, vector<int>& current, vector<vector<int>>& result) {
    if (current.size() == k) { result.push_back(current); return; }   // k'ya ulaştık, kaydet
    if (index == nums.size()) return;                                   // eleman bitti ama k'ya ulaşamadık

    solve(nums, index + 1, k, current, result);          // AL-MA

    current.push_back(nums[index]);
    solve(nums, index + 1, k, current, result);          // AL
    current.pop_back();
}
```

## Aynı elemanı birden fazla kez kullanabiliyorsan (örn. sınırsız bozuk para)

"AL" dalında bir sonraki elemana değil, **aynı elemana** tekrar bakılır (`index` yerine `index` kalır, `index+1` olmaz):

```cpp
// AL seçeneğinde: solve(nums, index, ...)   ← index+1 değil, aynı index tekrar denenebilir
```

---

## Örnek Problem: Combination Sum (LeetCode 39)

Toplamı `target`'a eşit olan grupları bul, aynı sayı tekrar kullanılabilir.

```cpp
void solve(vector<int>& candidates, int index, int remaining,
           vector<int>& current, vector<vector<int>>& result) {
    if (remaining == 0) { result.push_back(current); return; }   // tam hedefe ulaşıldı
    if (remaining < 0 || index == candidates.size()) return;      // aştı ya da eleman bitti

    // AL-MA: bu elemanı hiç kullanma, sıradakine geç
    solve(candidates, index + 1, remaining, current, result);

    // AL: bu elemanı kullan, aynı elemanı tekrar deneyebilmek için index AYNI kalıyor
    current.push_back(candidates[index]);
    solve(candidates, index, remaining - candidates[index], current, result);
    current.pop_back();
}

vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
    vector<vector<int>> result;
    vector<int> current;
    solve(candidates, 0, target, current, result);
    return result;
}
```

### Adım adım (kısaltılmış): candidates = [2,3], target = 5

```
solve(index=0, remaining=5, current=[])
│
├── AL-MA (2'yi almadan devam)
│   solve(index=1, remaining=5, current=[])
│   │
│   ├── AL-MA (3'ü almadan) → solve(index=2, remaining=5) → index==size, remaining≠0 → çık
│   └── AL (3'ü al) → current=[3]
│       solve(index=1, remaining=2, current=[3])
│       ├── AL-MA → solve(index=2, remaining=2) → çık
│       └── AL (3'ü tekrar al) → current=[3,3]
│           solve(index=1, remaining=-1) → remaining<0 → çık
│       geri al: current=[3]
│   geri al: current=[]
│
└── AL (2'yi al) → current=[2]
    solve(index=0, remaining=3, current=[2])   ← index AYNI kaldı, 2 tekrar denenebilir
    │
    ├── AL-MA (2'yi bir daha almadan devam) → solve(index=1, remaining=3, current=[2])
    │   ├── AL-MA (3'ü almadan) → çık
    │   └── AL (3'ü al) → current=[2,3]
    │       solve(index=1, remaining=0) → remaining==0 → kaydet: [2,3]
    │       geri al: current=[2]
    │   geri al: current=[2]
    │
    └── AL (2'yi tekrar al) → current=[2,2]
        solve(index=0, remaining=1, current=[2,2])
        ├── AL-MA → ... sonunda remaining hep >0 kalır, sonuç yok
        └── AL (2'yi tekrar al) → current=[2,2,2]
            solve(index=0, remaining=-1) → çık
        geri al: current=[2,2]
    geri al: current=[2]
geri al: current=[]

Sonuç: [[2,3]]
```

**Complexity:** en kötü durumda üstel, `SC = O(target / min_candidate)` recursion derinliği.

## Bu mantıkla çözülen benzer problemler
Subsets / Subsets II, Combinations (LeetCode 77), Combination Sum II (tekrar yok), Palindrome Partitioning.
