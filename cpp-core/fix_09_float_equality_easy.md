## challenge: fix: the sensor reading that doesn't count
tags: code-review, debugging, floating-point
track: core
difficulty: easy

This code review found a bug: counting how many sensor readings equal 0.3 misses readings that were computed as 0.1 + 0.2 — values that print identically are not counted. Find and fix it — keep the function signature.

hint: Print the readings with 17 significant digits and compare them again.
hint: This is exact floating-point equality where a tolerance is needed.
hint: 0.1 + 0.2 is 0.30000000000000004 in binary floating point, so operator== rejects it; compare |r - target| against a small epsilon instead.

```cpp
// starter
int countOccurrences(const std::vector<double>& readings, double target) {
    int count = 0;
    for (double r : readings) {
        if (r == target) {
            ++count;
        }
    }
    return count;
}
```

```cpp
int countOccurrences(const std::vector<double>& readings, double target) {
    int count = 0;
    for (double r : readings) {
        if (std::fabs(r - target) < 1e-9) {
            ++count;
        }
    }
    return count;
}
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    std::vector<double> readings{0.1 + 0.2, 0.3, 0.7};
    assert(countOccurrences(readings, 0.3) == 2);   // buggy: 1 (0.1 + 0.2 != 0.3 exactly)
    assert(countOccurrences(readings, 0.7) == 1);
    assert(countOccurrences(readings, 9.9) == 0);
    std::puts("PASS");
}
```

**Editorial:** Neither 0.1 nor 0.2 nor 0.3 is exactly representable in binary floating point; `0.1 + 0.2` rounds to 0.30000000000000004, which is one ULP away from the literal `0.3`, so exact `==` calls them different even though both print as "0.3" at default precision. The fix compares against a tolerance (`std::fabs(r - target) < 1e-9`); production code should pick an epsilon meaningful for the domain, or a relative tolerance for values far from 1. Reviewers should challenge any `==`/`!=` between computed floating-point values — exact equality is only defensible for values that were assigned, never derived.
