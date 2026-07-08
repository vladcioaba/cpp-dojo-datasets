## challenge: Happy Number
tags: math, hash-table, two-pointers
track: faang
difficulty: easy

Write an algorithm to determine whether a positive integer `n` is "happy". Starting from `n`, repeatedly replace the number with the sum of the squares of its digits. `n` is happy if this process eventually reaches `1`; if instead it loops endlessly without reaching `1`, it is not happy.

Constraints: `1 <= n <= 2^31 - 1`.

Example: `n = 19` → `true` (`1^2+9^2=82`, `8^2+2^2=68`, `6^2+8^2=100`, `1^2+0^2+0^2=1`). Example: `n = 2` → `false`.

hint: The sequence of digit-square sums either reaches 1 or enters a cycle — so this is really cycle detection.
hint: You could remember every number you have seen in a set, but Floyd's tortoise and hare finds the cycle with O(1) extra space.
hint: Advance a slow pointer one step and a fast pointer two steps per iteration; if they meet at a value other than 1, the number is stuck in a loop.

```cpp
// starter
bool isHappy(int n);
```

```cpp
bool isHappy(int n) {
    auto next = [](int x) {
        int s = 0;
        while (x) { int d = x % 10; s += d * d; x /= 10; }
        return s;
    };
    int slow = n, fast = next(n);
    while (fast != 1 && slow != fast) {
        slow = next(slow);
        fast = next(next(fast));
    }
    return fast == 1;
}
```

```cpp
// harness
#include <cstdio>
//__USER__
int main() {
    if (isHappy(19) != true)  { std::puts("case1"); return 1; }
    if (isHappy(2)  != false) { std::puts("case2"); return 1; }
    if (isHappy(1)  != true)  { std::puts("case3"); return 1; }
    if (isHappy(7)  != true)  { std::puts("case4"); return 1; }
    if (isHappy(100) != true) { std::puts("case5"); return 1; }
    if (isHappy(4)  != false) { std::puts("case6"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Replacing a number by the sum of the squares of its digits is a deterministic map, so the trajectory from any start is eventually periodic: it either hits the fixed point 1 or falls into a cycle (every unhappy number eventually reaches the cycle beginning at 4). Treating the map as a linked list of values, Floyd's cycle detection runs a slow and a fast walker; they collide inside any cycle. If the collision value is 1 the number is happy, otherwise it is not. O(log n) work per step with O(1) extra space.
