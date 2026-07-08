## challenge: Time Based Key-Value Store
tags: binary-search, design, hash-table
track: faang
difficulty: medium

Design a time-based key-value store that can store multiple values for the same key at different timestamps and retrieve the value for a key at a given time. Implement the `TimeMap` class: `set(key, value, timestamp)` stores the mapping, and `get(key, timestamp)` returns the value with the largest `timestamp_prev <= timestamp` that was previously set; if none exists it returns `""`. For each key, `set` is called with strictly increasing timestamps.

Constraints: `1 <= key.length, value.length <= 100`, `1 <= timestamp <= 10^7`, timestamps for a given key are strictly increasing across `set` calls, at most `2 * 10^5` calls in total.

Example: `set("foo","bar",1)`; `get("foo",1)` → `"bar"`; `get("foo",3)` → `"bar"`; `set("foo","bar2",4)`; `get("foo",4)` → `"bar2"`; `get("foo",5)` → `"bar2"`.

hint: Group each key's `(timestamp, value)` entries in a list; since `set` timestamps increase, each list is already sorted by time.
hint: `get` is a floor query: find the entry with the largest timestamp not exceeding the query — a binary search over that key's list.
hint: Search for the first timestamp strictly greater than the query; the entry just before it (if any) is the answer.

```cpp
// starter
#include <string>
class TimeMap {
public:
    TimeMap();
    void set(std::string key, std::string value, int timestamp);
    std::string get(std::string key, int timestamp);
};
```

```cpp
class TimeMap {
    std::unordered_map<std::string, std::vector<std::pair<int, std::string>>> store;
public:
    TimeMap() {}
    void set(std::string key, std::string value, int timestamp) {
        store[key].push_back({timestamp, value});   // appended in increasing time order
    }
    std::string get(std::string key, int timestamp) {
        auto it = store.find(key);
        if (it == store.end()) return "";
        auto& v = it->second;
        int lo = 0, hi = (int)v.size();
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (v[mid].first <= timestamp) lo = mid + 1;   // candidate, look right
            else hi = mid;
        }
        if (lo == 0) return "";        // every stored time is later than the query
        return v[lo - 1].second;
    }
};
```

```cpp
// harness
#include <cstdio>
#include <string>
#include <vector>
#include <unordered_map>
#include <utility>
//__USER__
int main() {
    TimeMap tm;
    tm.set("foo", "bar", 1);
    if (tm.get("foo", 1) != "bar")  { std::puts("case1"); return 1; }
    if (tm.get("foo", 3) != "bar")  { std::puts("case2"); return 1; }
    tm.set("foo", "bar2", 4);
    if (tm.get("foo", 4) != "bar2") { std::puts("case3"); return 1; }
    if (tm.get("foo", 5) != "bar2") { std::puts("case4"); return 1; }
    if (tm.get("foo", 0) != "")     { std::puts("case5"); return 1; }
    if (tm.get("none", 1) != "")    { std::puts("case6"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Keep a hash map from key to a vector of `(timestamp, value)` entries. Because `set` is called with increasing timestamps per key, each vector stays sorted, so `set` is O(1) amortized. `get` is a floor query: binary search for the first entry whose timestamp exceeds the query, then take the entry immediately before it (or `""` if the query precedes all stored times). Each `get` runs in O(log n) over that key's entries.
