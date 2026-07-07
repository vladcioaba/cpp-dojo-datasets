## challenge: Top K Frequent Elements
tags: heap, hash-table, bucket-sort, arrays-hashing
track: faang
difficulty: medium

Given an integer array `nums` and an integer `k`, return the `k` most frequent elements. The answer is guaranteed to be unique. Return them in any order.

Constraints: `1 <= nums.length <= 10^5`, `k` is in `[1, number of distinct elements]`.

Example: `nums = [1,1,1,2,2,3], k = 2` → `[1,2]`. Example: `nums = [1], k = 1` → `[1]`.

hint: First count how often each value appears; then you only need the k largest counts.
hint: Frequencies are bounded by n, so instead of sorting counts you can bucket values by their frequency.
hint: Index buckets by count (0..n), then scan from the highest bucket downward, collecting values until you have k.

```cpp
// starter
#include <vector>
std::vector<int> topKFrequent(std::vector<int>& nums, int k);
```

```cpp
std::vector<int> topKFrequent(std::vector<int>& nums, int k) {
    std::unordered_map<int, int> cnt;
    for (int x : nums) ++cnt[x];
    int n = (int)nums.size();
    std::vector<std::vector<int>> buckets(n + 1);
    for (auto& [val, c] : cnt) buckets[c].push_back(val);
    std::vector<int> out;
    for (int f = n; f >= 1 && (int)out.size() < k; --f) {
        for (int v : buckets[f]) {
            out.push_back(v);
            if ((int)out.size() == k) break;
        }
    }
    return out;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <unordered_map>
#include <algorithm>
using std::vector;
//__USER__
int main() {
    { vector<int> n{1,1,1,2,2,3}; auto r = topKFrequent(n, 2); std::sort(r.begin(), r.end());
      if (r != vector<int>({1,2})) { std::puts("case1"); return 1; } }
    { vector<int> n{1}; auto r = topKFrequent(n, 1); std::sort(r.begin(), r.end());
      if (r != vector<int>({1})) { std::puts("case2"); return 1; } }
    { vector<int> n{4,4,4,5,5,6,6,6,6}; auto r = topKFrequent(n, 1); std::sort(r.begin(), r.end());
      if (r != vector<int>({6})) { std::puts("case3"); return 1; } }
    { vector<int> n{-1,-1,2,2,2,3}; auto r = topKFrequent(n, 2); std::sort(r.begin(), r.end());
      if (r != vector<int>({-1,2})) { std::puts("case4"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Count occurrences with a hash map, then place each value into a bucket indexed by its frequency. Scanning buckets from the highest frequency down collects the top k without ever fully sorting. O(n) time, O(n) space.
