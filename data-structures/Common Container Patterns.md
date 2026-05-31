# Common Container Patterns

| Pattern | Code | Note |
|---------|------|------|
| Zero-initialized vector | `vector<int> v(n, 0)` | n elements, all zero. |
| Vector with iota | `iota(v.begin(), v.end(), 0)` | Fill with 0..n-1. |
| Initializer list | `vector<int> v = {1,2,3,4}` | |
| Array init | `array<int,4> a = {1,2,3,4}` | Fixed size, on stack. |
| 2D vector | `vector<vector<int>> g(r, vector<int>(c, 0))` | r rows, c columns, filled with 0. |
| Concatenate two vectors | `a.insert(a.end(), b.begin(), b.end())` | Append b to end of a. |
| Vector to set | `set<int> s(v.begin(), v.end())` | Unique + sorted. |
| Set to vector | `vector<int> v(s.begin(), s.end())` | |
| Map init list | `map<string,int> m = {{"a",1},{"b",2}}` | |
| Erase-remove idiom | `v.erase(remove(v.begin(),v.end(),x), v.end())` | Remove value from vector. |
| Erase-remove_if | `v.erase(remove_if(v.begin(),v.end(),pred), v.end())` | Remove by condition. |
| Sort + unique | `sort(v.begin(),v.end()); v.erase(unique(v.begin(),v.end()),v.end())` | Remove duplicates. |
| String split | `istringstream ss(str); string tok; while(getline(ss,tok,','))` | Split by comma. |
| Max priority queue | `priority_queue<int> pq` | Default max-heap. |
| Min priority queue | `priority_queue<int, vector<int>, greater<int>> pq` | Min-heap. |
