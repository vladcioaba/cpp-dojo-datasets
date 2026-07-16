## challenge: fix: stubborn negatives survive the purge
tags: code-review, debugging, iterators
track: core
difficulty: medium

This code review found a bug: after "removing all negatives", the vector sometimes still contains negative numbers — always ones that sat right next to another negative. Find and fix it — keep the function signature.

hint: Trace what happens to the indices when two negatives are adjacent.
hint: This is the classic erase-inside-a-loop bug, in index form.
hint: After erase(begin() + i), the next element shifts into slot i, but the loop increments i anyway and never examines it.

```cpp
// starter
void removeNegatives(std::vector<int>& v) {
    for (std::size_t i = 0; i < v.size(); ++i) {
        if (v[i] < 0) {
            v.erase(v.begin() + i);
        }
    }
}
```

```cpp
void removeNegatives(std::vector<int>& v) {
    for (std::size_t i = 0; i < v.size(); ) {
        if (v[i] < 0) {
            v.erase(v.begin() + i);   // next element shifts into slot i
        } else {
            ++i;
        }
    }
}
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    std::vector<int> a{3, -1, -4, -2, 5, -6};
    removeNegatives(a);
    assert(a == std::vector<int>({3, 5}));

    std::vector<int> b{-7, -8};
    removeNegatives(b);
    assert(b.empty());

    std::vector<int> c{1, 2, 3};
    removeNegatives(c);
    assert(c == std::vector<int>({1, 2, 3}));

    std::puts("PASS");
}
```

**Editorial:** When `v[i]` is erased, the element after it shifts down into index `i` — but the `for` loop then executes `++i`, so the shifted element is never inspected. Two adjacent negatives therefore leave the second one behind (`{3, -1, -4, ...}` keeps `-4`). The fix is to advance `i` only when nothing was erased; equivalently, use the erase-remove idiom (`v.erase(std::remove_if(...), v.end())`), which is also O(n) instead of O(n²). Reviewers should flag any loop that erases from the container it is indexing and then unconditionally increments.
