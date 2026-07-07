## snippet: 2026-07-06 — seed — The loop that never ends
tags: core, integer-rules

```cpp
std::vector<int> v = {1, 2, 3};
for (unsigned i = v.size() - 1; i >= 0; --i)
    std::cout << v[i];
```

**Analysis:** `i` is unsigned, so `i >= 0` is always true — after `i` hits 0, `--i` wraps to `4294967295` and `v[i]` is out-of-bounds UB. Also `v.size() - 1` on an *empty* vector wraps the same way before the loop even starts.

Fixes, best first: iterate forward with a range-for over `std::views::reverse(v)` (C++20); classic index loop with `for (auto i = v.size(); i-- > 0;)` (the "goes-to operator" `i --> 0`); or use a signed `int`/`ptrdiff_t` index. Signed/unsigned wraparound is the most reposted C++ gotcha on LinkedIn for a reason.
