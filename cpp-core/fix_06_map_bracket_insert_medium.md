## challenge: fix: the settings map that grows on read
tags: code-review, debugging, containers
track: core
difficulty: medium

This code review found a bug: merely *reading* a setting adds phantom empty entries to the map — after a few lookups of unknown keys, dumps of the settings show keys nobody ever set. Find and fix it — keep the function signature.

hint: The returned values look fine; watch what happens to settings.size() after a miss.
hint: This is std::map::operator[] doing more than a lookup.
hint: settings[key] default-constructs and inserts an empty string for every missing key — twice per miss here — so reads mutate the map.

```cpp
// starter
std::string getSetting(std::map<std::string, std::string>& settings,
                       const std::string& key) {
    if (settings[key].empty()) {
        return "default";
    }
    return settings[key];
}
```

```cpp
std::string getSetting(std::map<std::string, std::string>& settings,
                       const std::string& key) {
    auto it = settings.find(key);
    if (it == settings.end() || it->second.empty()) {
        return "default";
    }
    return it->second;
}
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    std::map<std::string, std::string> settings{
        {"host", "localhost"},
        {"port", "8080"},
    };
    assert(getSetting(settings, "host") == "localhost");
    assert(getSetting(settings, "retries") == "default");
    assert(settings.size() == 2);                    // buggy version inserted "retries"
    assert(getSetting(settings, "timeout") == "default");
    assert(settings.size() == 2);                    // ...and "timeout"
    assert(settings.count("retries") == 0);
    std::puts("PASS");
}
```

**Editorial:** `std::map::operator[]` is find-or-*insert*: for a missing key it default-constructs an empty `std::string`, inserts it, and returns a reference to it. Here every miss silently plants an empty entry (and the function even does it twice per call), so read paths mutate shared state — corrupting iteration, size checks, and serialization of the settings. The fix is `find` plus an end-check, leaving the map untouched. In review, `operator[]` on a map inside anything that is semantically a *read* is an instant flag — that is also why it does not exist on `const` maps.
