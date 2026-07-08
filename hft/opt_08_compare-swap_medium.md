## challenge: Branchless compare-swap (sorting-network primitive)
tags: branchless, bit-tricks, low-level-optimization
track: hft
difficulty: medium

A compare-exchange orders two elements in place: after it runs, `a <= b`. It is the atom of sorting networks, which HFT code uses to sort tiny fixed-size sets (top-of-book levels) with zero data-dependent branches. The naive `if (a > b) std::swap(a, b)` branches on the data; on random inputs it mispredicts ~50% of the time at ~15-20 cycles each. Implement `void cswap(int& a, int& b)` that leaves the smaller in `a` and the larger in `b`, branchlessly.

Constraints: `a`, `b` any 32-bit `int` including `INT_MIN`/`INT_MAX`. Use only comparison and bitwise selection — no additions, so no overflow. If already ordered, leave them unchanged.

Example: `cswap` on `(5, 3)` → `a=3, b=5`. Example: `(3, 5)` → `a=3, b=5` (unchanged). Example: `(INT_MAX, INT_MIN)` → `a=INT_MIN, b=INT_MAX`.

hint: Compute the min and the max with the same mask, then write them back — no swap, no branch.
hint: Build the mask once: `int m = -(a < b);` (all-ones if already ordered, all-zeros if not).
hint: `mn = b ^ ((a ^ b) & m)` is the smaller and `mx = a ^ ((a ^ b) & m)` is the larger; store `a = mn; b = mx;`.

```cpp
// starter
void cswap(int& a, int& b);
```

```cpp
void cswap(int& a, int& b) {
    int m  = -(a < b);                 // -1 if a < b else 0
    int d  = (a ^ b) & m;
    int mn = b ^ d;                    // min(a, b)
    int mx = a ^ d;                    // max(a, b)
    a = mn;
    b = mx;
}
```

```cpp
// harness
#include <cstdio>
#include <climits>
//__USER__
static int check(int a0, int b0, int wa, int wb) {
    int a = a0, b = b0;
    cswap(a, b);
    if (a != wa || b != wb) { std::printf("cswap(%d,%d)=(%d,%d) want (%d,%d)\n", a0, b0, a, b, wa, wb); return 1; }
    return 0;
}
int main() {
    if (check(5, 3, 3, 5)) return 1;
    if (check(3, 5, 3, 5)) return 1;
    if (check(7, 7, 7, 7)) return 1;
    if (check(INT_MAX, INT_MIN, INT_MIN, INT_MAX)) return 1;
    if (check(INT_MIN, INT_MAX, INT_MIN, INT_MAX)) return 1;
    if (check(-2, -9, -9, -2)) return 1;
    if (check(0, INT_MIN, INT_MIN, 0)) return 1;
    if (check(INT_MAX, 0, 0, INT_MAX)) return 1;
    std::puts("PASS");
}
```

**Editorial:** Order-two-in-place is a min and a max sharing one mask. `m = -(a < b)` is all-ones when the pair is already sorted and all-zeros otherwise; `d = (a ^ b) & m` is `a ^ b` in the sorted case and `0` in the unsorted case. Then `b ^ d` yields the smaller and `a ^ d` the larger, and writing both back leaves `a <= b` unconditionally. No `swap`, no jump — the naive `if (a > b) swap` mispredicts on random data for ~15-20 cycles per miss, which is why sorting networks built from branchless compare-exchanges beat comparison sorts on tiny fixed inputs. Only comparisons and XOR/AND touch the values, so `INT_MIN`/`INT_MAX` are safe. Compilers emit a pair of `cmov`s. O(1).
