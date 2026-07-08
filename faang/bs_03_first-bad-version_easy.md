## challenge: First Bad Version
tags: binary-search, interactive
track: faang
difficulty: easy

You are a release manager for a product with versions `1..n`. Each version is built on the previous one, so once a version is bad, every later version is bad too. You are given an API `bool isBadVersion(int version)` that tells you whether a version is bad. Find the first bad version while minimizing calls to the API.

Constraints: `1 <= bad <= n <= 2^31 - 1`. Exactly one threshold exists: versions before `bad` are good, versions from `bad` onward are bad.

Example: `n = 5`, first bad is `4`, calls resolve to `4`. Example: `n = 1`, first bad is `1` → `1`. Example: `n = 2`, first bad is `2` → `2`.

hint: The predicate `isBadVersion` is monotonic: false, false, ..., false, true, true, ...; you want the boundary.
hint: Binary search the version range `[1, n]`, treating "is bad" as the condition that pulls `hi` down toward the first true.
hint: Compute `mid = lo + (hi - lo) / 2` — with `n` near `2^31 - 1`, `lo + hi` would overflow a 32-bit `int`.

```cpp
// starter
bool isBadVersion(int version);   // provided by the platform
int firstBadVersion(int n);
```

```cpp
int firstBadVersion(int n) {
    int lo = 1, hi = n;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (isBadVersion(mid)) hi = mid;
        else lo = mid + 1;
    }
    return lo;
}
```

```cpp
// harness
#include <cstdio>
static int g_bad;
bool isBadVersion(int version) { return version >= g_bad; }
//__USER__
int main() {
    g_bad = 4;          if (firstBadVersion(5) != 4) { std::puts("case1"); return 1; }
    g_bad = 1;          if (firstBadVersion(1) != 1) { std::puts("case2"); return 1; }
    g_bad = 1;          if (firstBadVersion(2) != 1) { std::puts("case3"); return 1; }
    g_bad = 2;          if (firstBadVersion(2) != 2) { std::puts("case4"); return 1; }
    g_bad = 1702766719; if (firstBadVersion(2126753390) != 1702766719) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** The bad-version test is monotonic, so binary search finds the boundary between the last good and first bad version. Collapse the window toward the first `true`: when `mid` is bad, keep it as a candidate (`hi = mid`); otherwise discard it (`lo = mid + 1`). Using `lo + (hi - lo) / 2` prevents overflow when `n` approaches the 32-bit limit. O(log n) API calls, O(1) space.
