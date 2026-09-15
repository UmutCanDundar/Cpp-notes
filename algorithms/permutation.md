# Permütasyon (Permutations)

## When to use
- Problem "**sıralama/dizilim**" diyor: [1,2] ve [2,1] farklı sayılıyor.
- "Kaç farklı şekilde dizebilirsin" tarzı sorular.

## Mantık (tek fonksiyon)
Her boş yere hangi sayıyı koyacağını sırayla dene. Bir sayıyı kullandığında `used[i]=true` yap (kilitle),
o daldan dönünce `used[i]=false` yap (kilidi aç) ki başka bir yerde tekrar denenebilsin.
Bu da DFS'teki visited mantığının aynısı — sadece "şu anki yol için" geçici işaretleme.

```cpp
void solve(vector<int>& nums, vector<bool>& used, vector<int>& current, vector<vector<int>>& result) {
    if (current.size() == nums.size()) {   // tüm yerler doldu
        result.push_back(current);
        return;
    }

    for (int i = 0; i < nums.size(); i++) {
        if (used[i]) continue;      // bu sayı zaten kullanıldı, atla

        used[i] = true;
        current.push_back(nums[i]); // seç

        solve(nums, used, current, result);  // devam et

        current.pop_back();  // geri al
        used[i] = false;     // tekrar kullanılabilir yap
    }
}
```

### Adım adım örnek: nums = [1, 2]

```
solve(current=[], used=[F,F])
│
├── i=0 → nums[0]=1 seç
│   used=[T,F], current=[1]
│   solve(current=[1], used=[T,F])
│   │
│   └── i=1 → nums[1]=2 seç (i=0 zaten used=T, atlanır)
│       used=[T,T], current=[1,2]
│       solve(current=[1,2]) → size==2 → kaydet: [1,2]
│       geri al: current=[1], used=[T,F]
│   geri al: current=[], used=[F,F]
│
└── i=1 → nums[1]=2 seç
    used=[F,T], current=[2]
    solve(current=[2], used=[F,T])
    │
    └── i=0 → nums[0]=1 seç
        used=[T,T], current=[2,1]
        solve(current=[2,1]) → size==2 → kaydet: [2,1]
        geri al: current=[2], used=[F,T]
    geri al: current=[], used=[F,F]

Sonuç: [[1,2], [2,1]]
```

---

## Tekrarlı elemanlarda (örn. [1,1,2]) aynı permütasyonu iki kez üretmemek için

Önce sırala, sonra bir kuralla aynı değerdeki elemanları belirli sırayla kullan:

```cpp
sort(nums.begin(), nums.end());
// döngü içinde:
if (i > 0 && nums[i] == nums[i-1] && !used[i-1]) continue;
```
Yani: aynı değerli iki eleman varsa, öncekini kullanmadan sonrakini kullanma. Bu tek satır dışında algoritma birebir aynı.

---

## Örnek Problem: Permutations (LeetCode 46)

```cpp
vector<vector<int>> permute(vector<int>& nums) {
    vector<vector<int>> result;
    vector<int> current;
    vector<bool> used(nums.size(), false);
    solve(nums, used, current, result);
    return result;
}
```
(fonksiyon yukarıdaki `solve` ile birebir aynı)

**Complexity:** TC `O(n! * n)`, SC `O(n)` recursion derinliği.

## Bu mantıkla çözülen benzer problemler
Permutations II (tekrarlı elemanlar), Letter Case Permutation, N-Queens (her satıra bir sütun ataması aslında permütasyon).
