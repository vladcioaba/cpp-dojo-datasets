## challenge: fix: the display name that gets stuck
tags: code-review, debugging, caching
track: core
difficulty: hard

This code review found a bug: the function sometimes returns stale data — after the first user is looked up, every other user is shown the first user's display name. Find and fix it — keep the function signature.

hint: The cache is doing its job a little too well; look at the condition that decides a cache hit.
hint: This is cache staleness: the validity check tests the wrong thing.
hint: "Is the cached string non-empty" only proves the cache was filled once by *someone* — a hit must require cachedId == userId, otherwise the first result is returned for every id forever.

```cpp
// starter
const std::string& displayName(int userId) {
    static int cachedId = -1;
    static std::string cachedName;
    if (!cachedName.empty()) {
        return cachedName;   // cache hit
    }
    cachedId = userId;
    cachedName = "user-" + std::to_string(userId);
    return cachedName;
}
```

```cpp
const std::string& displayName(int userId) {
    static int cachedId = -1;
    static std::string cachedName;
    if (cachedId != userId || cachedName.empty()) {
        cachedId = userId;
        cachedName = "user-" + std::to_string(userId);
    }
    return cachedName;   // cache hit only when cachedId == userId
}
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    assert(displayName(7) == "user-7");
    assert(displayName(7) == "user-7");     // repeat lookup: served from cache
    assert(displayName(42) == "user-42");   // buggy version returns stale "user-7"
    assert(displayName(7) == "user-7");     // and the cache must refresh back, too
    std::puts("PASS");
}
```

**Editorial:** The cache-hit test asks "do I have *a* cached name?" instead of "do I have the cached name *for this userId*?", so whichever id arrives first poisons the cache for every id after it — `cachedId` is even stored but never consulted. The fix keys the hit on `cachedId == userId` and refills otherwise. This bug family (cache checked for presence but not for matching key/version) is what "sometimes returns stale data" reports usually turn out to be; a reviewer spots it by demanding that every cache read compares against the *full* lookup key — and by noticing stored state (`cachedId`) that nothing ever reads. Note the single-entry static cache is also not thread-safe; under concurrency this wants a mutex or a per-key map.
