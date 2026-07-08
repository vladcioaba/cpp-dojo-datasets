## challenge: Count Vowels Permutation
tags: dynamic-programming
track: faang
difficulty: hard

Count how many strings of length `n` can be formed using only the vowels `a`, `e`, `i`, `o`, `u`, subject to these rules: each `a` may only be followed by `e`; each `e` may only be followed by `a` or `i`; each `i` may not be followed by another `i`; each `o` may only be followed by `i` or `u`; and each `u` may only be followed by `a`. Return the count modulo `10^9 + 7`.

Constraints: `1 <= n <= 2 * 10^4`.

Example: `n = 1` → `5` (each single vowel). Example: `n = 2` → `10` (`ae`, `ea`, `ei`, `ia`, `ie`, `io`, `iu`, `oi`, `ou`, `ua`).

hint: Track how many valid strings end in each vowel and advance one character at a time.
hint: Invert the rules: `a` can be preceded by `e`, `i`, `u`; `e` by `a`, `i`; `i` by `e`, `o`; `o` by `i`; `u` by `i`, `o`.

```cpp
// starter
int countVowelPermutation(int n);
```

```cpp
int countVowelPermutation(int n) {
    const long long MOD = 1000000007LL;
    long long a = 1, e = 1, i = 1, o = 1, u = 1;
    for (int step = 1; step < n; ++step) {
        long long na = (e + i + u) % MOD;
        long long ne = (a + i) % MOD;
        long long ni = (e + o) % MOD;
        long long no = i % MOD;
        long long nu = (i + o) % MOD;
        a = na; e = ne; i = ni; o = no; u = nu;
    }
    return (int)((a + e + i + o + u) % MOD);
}
```

```cpp
// harness
#include <cstdio>
//__USER__
int main() {
    if (countVowelPermutation(1)   != 5)        { std::puts("case1"); return 1; }
    if (countVowelPermutation(2)   != 10)       { std::puts("case2"); return 1; }
    if (countVowelPermutation(5)   != 68)       { std::puts("case3"); return 1; }
    if (countVowelPermutation(144) != 18208803) { std::puts("case4"); return 1; }
    if (countVowelPermutation(100) != 173981881){ std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Let `a,e,i,o,u` be the number of valid strings of the current length ending in each vowel. Extending by one character, a new string ending in a vowel `v` counts every string whose last vowel is legally followed by `v`. Inverting the successor rules gives `a' = e+i+u`, `e' = a+i`, `i' = e+o`, `o' = i`, `u' = i+o`, all taken modulo `10^9 + 7`. Starting from all ones for length 1 and iterating `n-1` times, the answer is the sum of the five counts. O(n) time, O(1) space.
