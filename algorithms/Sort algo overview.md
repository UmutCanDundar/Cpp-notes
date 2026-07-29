# Sorting Algorithms Guide (C++)

Only the sorting algorithms worth actually knowing: fundamentals + the ones still used in practice. Redundant O(n²) algorithms that solve the same problem strictly worse (bubble sort, selection sort) are skipped in favor of **Insertion Sort**, which covers the same "basic/educational" niche but is actually useful in practice (adaptive, good for nearly-sorted/small data — and it's literally what `std::sort` switches to for small partitions).

---

## 1. Insertion Sort

### What it is
Build the sorted array one element at a time — take the next element and insert it into its correct position among the already-sorted prefix.

### When to use
- **Small arrays** (n < ~20-30) — often faster in practice than O(n log n) sorts due to low overhead. This is why `std::sort`/introsort switches to insertion sort for small sub-arrays internally.
- **Nearly-sorted data** — it's adaptive: TC approaches O(n) if the array is almost sorted.
- Online sorting: you receive elements one at a time and need the array sorted at every point.

### Complexity
- **TC:** O(n²) worst/avg, **O(n) best** (already sorted) | **SC:** O(1) | **Stable:** Yes

```cpp
void insertionSort(vector<int>& arr) {
    for (int i = 1; i < (int)arr.size(); i++) {
        int key = arr[i];
        int j = i - 1;
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = key;
    }
}
```

---

## 2. Merge Sort

### What it is
Divide the array into halves recursively, sort each half, then merge the two sorted halves back together.

### When to use
- You need a **guaranteed** O(n log n), no worst-case blowup (unlike quicksort).
- **Stability matters** (equal elements must keep their relative order — e.g., sorting objects by one field while preserving order from a previous sort).
- Sorting **linked lists** (no random access needed, unlike quicksort's partitioning).
- External sorting (data too large for memory — merge sort's sequential access pattern is ideal for disk/streams).

### Complexity
- **TC:** O(n log n) always (best = worst = avg) | **SC:** O(n) | **Stable:** Yes

```cpp
void merge(vector<int>& arr, int l, int m, int r) {
    vector<int> left(arr.begin()+l, arr.begin()+m+1);
    vector<int> right(arr.begin()+m+1, arr.begin()+r+1);
    int i = 0, j = 0, k = l;
    while (i < (int)left.size() && j < (int)right.size())
        arr[k++] = (left[i] <= right[j]) ? left[i++] : right[j++];
    while (i < (int)left.size()) arr[k++] = left[i++];
    while (j < (int)right.size()) arr[k++] = right[j++];
}

void mergeSort(vector<int>& arr, int l, int r) {
    if (l >= r) return;
    int m = l + (r - l) / 2;
    mergeSort(arr, l, m);
    mergeSort(arr, m+1, r);
    merge(arr, l, m, r);
}
```

---

## 3. Quick Sort

### What it is
Pick a pivot, partition the array so smaller elements go left and larger go right, then recursively sort each side. In-place, no extra array needed for merging.

### When to use
- General-purpose sorting when average-case speed and low memory overhead matter more than worst-case guarantees.
- This is what most standard libraries actually use under the hood (as part of **introsort** — quicksort that falls back to heapsort if recursion gets too deep, and insertion sort for small partitions). In C++, `std::sort` is an introsort.
- Use **random pivot** or **median-of-three** to avoid the O(n²) worst case on adversarial/sorted input.

### Complexity
- **TC:** O(n log n) avg, O(n²) worst (rare with good pivot selection) | **SC:** O(log n) (recursion stack) | **Stable:** No

```cpp
int partition(vector<int>& arr, int lo, int hi) {
    int pivot = arr[hi];
    int i = lo - 1;
    for (int j = lo; j < hi; j++) {
        if (arr[j] < pivot) swap(arr[++i], arr[j]);
    }
    swap(arr[i+1], arr[hi]);
    return i + 1;
}

void quickSort(vector<int>& arr, int lo, int hi) {
    if (lo >= hi) return;
    int p = partition(arr, lo, hi);
    quickSort(arr, lo, p - 1);
    quickSort(arr, p + 1, hi);
}
```

---

## 4. Heap Sort

### What it is
Build a max-heap from the array, then repeatedly extract the max element and place it at the end.

### When to use
- You need **guaranteed O(n log n) with O(1) extra space** — merge sort needs O(n) space, quicksort's worst case is O(n²); heap sort has neither problem.
- Real-time systems where worst-case time bound matters more than average-case speed.
- As the fallback inside introsort (mentioned above) when quicksort recursion goes too deep.

### Complexity
- **TC:** O(n log n) always | **SC:** O(1) | **Stable:** No

```cpp
void heapify(vector<int>& arr, int n, int i) {
    int largest = i, l = 2*i + 1, r = 2*i + 2;
    if (l < n && arr[l] > arr[largest]) largest = l;
    if (r < n && arr[r] > arr[largest]) largest = r;
    if (largest != i) {
        swap(arr[i], arr[largest]);
        heapify(arr, n, largest);
    }
}

void heapSort(vector<int>& arr) {
    int n = arr.size();
    for (int i = n/2 - 1; i >= 0; i--) heapify(arr, n, i);      // build max-heap
    for (int i = n - 1; i > 0; i--) {
        swap(arr[0], arr[i]);                                    // move max to end
        heapify(arr, i, 0);
    }
}
```

---

## 5. Counting Sort

### What it is
A **non-comparison** sort: count occurrences of each value, then reconstruct the sorted array from the counts. Doesn't compare elements to each other at all — it exploits knowing the value range.

### When to use
- Values are integers in a **small, known range** (e.g., ages 0–120, grades 0–100).
- You need better than O(n log n) — this beats the comparison-sort lower bound because it isn't a comparison sort.
- Common as a subroutine in **Radix Sort** (below) and in bucket-based problems.
- **Don't use** if the value range is huge relative to n (e.g., range = 10⁹, n = 100) — space/time becomes O(range), wasteful.

### Complexity
- **TC:** O(n + k) where k = range of input values | **SC:** O(n + k) | **Stable:** Yes (if implemented carefully)

```cpp
vector<int> countingSort(vector<int>& arr, int maxVal) {
    vector<int> count(maxVal + 1, 0);
    for (int x : arr) count[x]++;

    vector<int> result;
    result.reserve(arr.size());
    for (int v = 0; v <= maxVal; v++) {
        while (count[v]-- > 0) result.push_back(v);
    }
    return result;
}
```

---

## 6. Radix Sort

### What it is
Sort integers digit by digit (least significant to most significant), using counting sort as a stable subroutine at each digit position.

### When to use
- Sorting large sets of **fixed-length integers or strings** (e.g., phone numbers, IDs) where the number of digits `d` is small relative to `log n`.
- Beats comparison sorts when `d` (number of digits) is small: O(d·(n+k)) can outperform O(n log n) for large n.

### Complexity
- **TC:** O(d · (n + k)) where d = number of digits, k = base (usually 10) | **SC:** O(n + k) | **Stable:** Yes

```cpp
void countingSortByDigit(vector<int>& arr, int exp) {
    int n = arr.size();
    vector<int> output(n);
    int count[10] = {0};

    for (int i = 0; i < n; i++) count[(arr[i] / exp) % 10]++;
    for (int i = 1; i < 10; i++) count[i] += count[i - 1];
    for (int i = n - 1; i >= 0; i--) {
        int digit = (arr[i] / exp) % 10;
        output[--count[digit]] = arr[i];
    }
    arr = output;
}

void radixSort(vector<int>& arr) {
    if (arr.empty()) return;
    int maxVal = *max_element(arr.begin(), arr.end());
    for (int exp = 1; maxVal / exp > 0; exp *= 10) {
        countingSortByDigit(arr, exp);
    }
}
```

---

## Decision Guide

| Situation | Use |
|---|---|
| Small array (n < ~30) or nearly sorted | Insertion Sort |
| Need guaranteed O(n log n), stability required | Merge Sort |
| General-purpose, average-case speed, in-place | Quick Sort |
| Need O(n log n) worst-case AND O(1) space | Heap Sort |
| Integers in a small known range | Counting Sort |
| Large fixed-length integers/strings, few digits | Radix Sort |
| "Just sort it" in a real codebase | `std::sort` (introsort: quicksort + heapsort fallback + insertion sort for small partitions) — implement your own only for interviews/learning |

### Cheat sheet

| Algorithm | TC (avg) | TC (worst) | SC | Stable? | Comparison-based? |
|---|---|---|---|---|---|
| Insertion Sort | O(n²) | O(n²) | O(1) | Yes | Yes |
| Merge Sort | O(n log n) | O(n log n) | O(n) | Yes | Yes |
| Quick Sort | O(n log n) | O(n²) | O(log n) | No | Yes |
| Heap Sort | O(n log n) | O(n log n) | O(1) | No | Yes |
| Counting Sort | O(n + k) | O(n + k) | O(n + k) | Yes | No |
| Radix Sort | O(d(n+k)) | O(d(n+k)) | O(n + k) | Yes | No |

**Rule of thumb:** if it's an interview and they don't specify, default to explaining **Quick Sort** (average case, in-place, most commonly asked) and mention **Merge Sort** if stability or worst-case guarantees come up. In real code, just call `std::sort` — it already picks the right strategy for you.
