## challenge: Gray Code
tags: backtracking, bit-manipulation, math
track: faang
difficulty: medium

An n-bit gray code sequence is a sequence of `2^n` integers where every integer in the range `[0, 2^n - 1]` appears exactly once, the sequence starts with `0`, and the binary representations of every pair of adjacent integers — including the last and the first, wrapping around — differ in exactly one bit. Given `n`, return any valid n-bit gray code sequence.

Constraints: `1 <= n <= 16`.

Example: `n = 2` -> `[0,1,3,2]` (`00 -> 01 -> 11 -> 10 -> 00`, each step flips one bit). Example: `n = 1` -> `[0,1]`. Any other valid ordering, such as `[0,2,3,1]` for `n = 2`, is also accepted.

hint: The output is not unique — you only need one cyclic ordering in which neighbours differ by a single bit.
hint: The reflected binary code has a closed form: the i-th value is `i XOR (i >> 1)`.
hint: Equivalently, build the sequence for `n-1` bits, mirror it, and prefix the reflected half with a leading 1 bit.

```cpp
// starter
#include <vector>
std::vector<int> grayCode(int n);
```

```cpp
std::vector<int> grayCode(int n) {
    std::vector<int> res;
    int total = 1 << n;
    res.reserve(total);
    for (int i = 0; i < total; ++i) res.push_back(i ^ (i >> 1));
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
static bool valid(const vector<int>& g, int n) {
    int sz = 1 << n;
    if ((int)g.size() != sz) return false;
    if (g[0] != 0) return false;
    vector<char> seen(sz, 0);
    for (int x : g) {
        if (x < 0 || x >= sz || seen[x]) return false;
        seen[x] = 1;
    }
    for (int i = 0; i < sz; ++i) {
        int a = g[i], b = g[(i + 1) % sz];
        if (__builtin_popcount((unsigned)(a ^ b)) != 1) return false;
    }
    return true;
}
//__USER__
int main() {
    for (int n = 1; n <= 12; ++n) {
        if (!valid(grayCode(n), n)) { std::printf("fail n=%d\n", n); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** Because the answer only needs to be *a* valid gray code, the reflected-binary formula `i XOR (i >> 1)` produces one directly: consecutive `i` and `i+1` differ by exactly one bit after the transform, and the cycle closes because `2^n - 1` maps back to a neighbour of `0`. A recursive "mirror and prefix" construction (the backtracking-style view) yields the same sequence. Generating all `2^n` values is O(2^n). The harness validates the four required properties rather than fixing one answer.
