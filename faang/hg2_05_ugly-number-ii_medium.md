## challenge: Ugly Number II
tags: heap, priority-queue, math, hash-table

track: faang
difficulty: medium

An ugly number is a positive integer whose only prime factors are `2`, `3`, and `5`. Given an integer `n`, return the `n`-th ugly number. By convention `1` is the first ugly number.

Constraints: `1 <= n <= 1690`.

Example: `n = 10` → `12` (the sequence begins `1, 2, 3, 4, 5, 6, 8, 9, 10, 12`). Example: `n = 1` → `1`. Example: `n = 7` → `8`.

hint: Every ugly number (except 1) is `2`, `3`, or `5` times a smaller ugly number, so you can generate them in increasing order.
hint: A min-heap always hands you the next smallest ugly number; multiply it by `2`, `3`, `5` and push the products back.
hint: The same product can arise several ways (e.g. `6 = 2·3 = 3·2`), so use a set to avoid inserting duplicates.

```cpp
// starter
int nthUglyNumber(int n);
```

```cpp
int nthUglyNumber(int n) {
    std::priority_queue<long long, std::vector<long long>, std::greater<long long>> pq;
    std::unordered_set<long long> seen;
    pq.push(1); seen.insert(1);
    long long ugly = 1;
    const int factors[3] = {2, 3, 5};
    for (int i = 0; i < n; ++i) {
        ugly = pq.top(); pq.pop();
        for (int f : factors) {
            long long nxt = ugly * f;
            if (seen.insert(nxt).second) pq.push(nxt);
        }
    }
    return (int)ugly;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <queue>
#include <functional>
#include <unordered_set>
//__USER__
int main() {
    if (nthUglyNumber(1)  != 1)  { std::puts("case1"); return 1; }
    if (nthUglyNumber(7)  != 8)  { std::puts("case2"); return 1; }
    if (nthUglyNumber(10) != 12) { std::puts("case3"); return 1; }
    if (nthUglyNumber(11) != 15) { std::puts("case4"); return 1; }
    if (nthUglyNumber(15) != 24) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Generate ugly numbers in ascending order with a min-heap seeded by `1`. Pop the smallest, record it as the next ugly number, and push its products with `2`, `3`, and `5`. Because the same value can be produced by multiple factor paths, guard insertions with a hash set so each distinct number enters the heap once. Extracting `n` values costs O(n log n) time and O(n) space; using 64-bit arithmetic avoids overflow while forming the products.
