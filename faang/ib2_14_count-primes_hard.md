## challenge: Count Primes
tags: math, sieve, array
track: faang
difficulty: hard

Given an integer `n`, return the number of prime numbers that are strictly less than `n`. A prime number is a natural number greater than 1 that has no positive divisors other than 1 and itself. For large `n`, testing each number for primality individually is too slow; a sieve computes them all together.

Constraints: `0 <= n <= 5*10^6`.

Example: `n = 10` → `4` (the primes below 10 are 2, 3, 5, 7). Example: `n = 0` → `0`. Example: `n = 1` → `0`.

hint: Use the Sieve of Eratosthenes: mark multiples of each prime as composite.
hint: Start marking multiples of a prime `p` at `p*p`; every smaller multiple was already crossed off by a smaller prime.
hint: You only need to sieve primes `p` with `p*p < n`; count the entries still marked prime at the end.

```cpp
// starter
int countPrimes(int n);
```

```cpp
int countPrimes(int n) {
    if (n < 3) return 0;
    std::vector<char> isComposite(n, 0);
    int count = 0;
    for (int p = 2; p < n; ++p) {
        if (isComposite[p]) continue;
        ++count;
        if ((long long)p * p < n)
            for (int m = p * p; m < n; m += p)
                isComposite[m] = 1;
    }
    return count;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    if (countPrimes(10) != 4)    { std::puts("case1"); return 1; }
    if (countPrimes(0) != 0)     { std::puts("case2"); return 1; }
    if (countPrimes(1) != 0)     { std::puts("case3"); return 1; }
    if (countPrimes(2) != 0)     { std::puts("case4"); return 1; }
    if (countPrimes(3) != 1)     { std::puts("case5"); return 1; }
    if (countPrimes(100) != 25)  { std::puts("case6"); return 1; }
    if (countPrimes(5000) != 669){ std::puts("case7"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** The Sieve of Eratosthenes marks composites in near-linear time. Iterate `p` from 2 upward; the first time a number is reached unmarked it is prime, so count it and cross off its multiples. Starting the crossing at `p*p` is safe because any smaller multiple `k*p` with `k < p` was already eliminated when sieving the smaller prime factor `k`. Guarding with `p*p < n` (in 64-bit to avoid overflow) skips primes too large to mark anything. Total work is O(n log log n) time and O(n) space.
