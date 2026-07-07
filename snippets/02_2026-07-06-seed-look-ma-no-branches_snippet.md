## snippet: 2026-07-06 — seed — "Look ma, no branches"
tags: core, style

```cpp
bool is_even(int n) {
    return n % 2 == 0 ? true : false;
}
```

**Analysis:** The viral "spot the smell" post. `x ? true : false` is a no-op on something already `bool` — `return n % 2 == 0;` says the same thing. Harmless here, but the pattern signals the author thinks of booleans as things to *produce with an if* rather than values to compute. Watch for its cousins: `if (cond) return true; else return false;` and `== true` comparisons.
