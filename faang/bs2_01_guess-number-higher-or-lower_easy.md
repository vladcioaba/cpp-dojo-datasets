## challenge: Guess Number Higher or Lower
tags: binary-search, interactive
track: faang
difficulty: easy

I picked a number from `1` to `n`. You must guess it. After each guess you may call the API `int guess(int num)`, which returns `-1` if your guess is higher than my number, `1` if it is lower, and `0` when you are correct. Return my number using as few guesses as possible.

Constraints: `1 <= n <= 2^31 - 1`, `1 <= pick <= n`.

Example: `n = 10`, pick is `6` → `6`. Example: `n = 1`, pick is `1` → `1`. Example: `n = 2`, pick is `1` → `1`.

hint: Each `guess` call tells you which half of the remaining range holds the answer — a textbook halving.
hint: Keep a window `[lo, hi]`; if `guess(mid)` says your guess is too high, discard the upper half, and vice versa.
hint: Use `mid = lo + (hi - lo) / 2`; with `n` near `2^31 - 1`, `lo + hi` would overflow a 32-bit `int`.

```cpp
// starter
int guess(int num);   // provided by the platform: -1 too high, 1 too low, 0 correct
int guessNumber(int n);
```

```cpp
int guessNumber(int n) {
    int lo = 1, hi = n;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        int r = guess(mid);
        if (r == 0) return mid;
        if (r < 0) hi = mid - 1;   // guess too high, look lower
        else lo = mid + 1;         // guess too low, look higher
    }
    return -1;
}
```

```cpp
// harness
#include <cstdio>
static int g_pick;
int guess(int num) { if (num == g_pick) return 0; return num > g_pick ? -1 : 1; }
//__USER__
int main() {
    g_pick = 6;          if (guessNumber(10) != 6) { std::puts("case1"); return 1; }
    g_pick = 1;          if (guessNumber(1)  != 1) { std::puts("case2"); return 1; }
    g_pick = 1;          if (guessNumber(2)  != 1) { std::puts("case3"); return 1; }
    g_pick = 2;          if (guessNumber(2)  != 2) { std::puts("case4"); return 1; }
    g_pick = 1702766719; if (guessNumber(2126753390) != 1702766719) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** The `guess` response is monotonic in `num`, so binary search the range `[1, n]`. A "too high" reply means the pick lies strictly below `mid` (`hi = mid - 1`); "too low" means it lies above (`lo = mid + 1`); `0` is the hit. Computing `mid = lo + (hi - lo) / 2` avoids the overflow that `lo + hi` would cause near the 32-bit ceiling. O(log n) guesses, O(1) space.
