## challenge: Restore IP Addresses
tags: backtracking, string
track: faang
difficulty: medium

Given a string `s` containing only digits, return all possible valid IP addresses that can be formed by inserting three dots into `s`. You may not reorder or remove any digits. A valid IP address consists of exactly four integers, each between `0` and `255` inclusive, separated by single dots, and no integer may have a leading zero (so `"0"` is valid but `"00"`, `"01"`, and `"255255"` are not). Return the addresses in any order.

Constraints: `1 <= s.length <= 20`, `s` consists of digits only.

Example: `s = "25525511135"` -> `["255.255.11.135","255.255.111.35"]`. Example: `s = "0000"` -> `["0.0.0.0"]`. Example: `s = "101023"` -> `["1.0.10.23","1.0.102.3","10.1.0.23","10.10.2.3","101.0.2.3"]`.

hint: There are exactly four segments; think of placing three cuts, each segment being 1 to 3 digits long.
hint: Backtrack over the segments: at each step try a segment of length 1, 2, or 3 that forms a number in `[0, 255]`, then recurse for the next segment.
hint: Reject a segment with a leading zero unless it is exactly `"0"`, and only accept when you have used all four segments and consumed the entire string.

```cpp
// starter
#include <vector>
#include <string>
std::vector<std::string> restoreIpAddresses(std::string s);
```

```cpp
std::vector<std::string> restoreIpAddresses(std::string s) {
    std::vector<std::string> res;
    int n = (int)s.size();
    if (n < 4 || n > 12) return res;                 // 4 segments, each 1..3 digits
    std::vector<std::string> parts;
    std::function<void(int)> dfs = [&](int start) {
        if ((int)parts.size() == 4) {
            if (start == n)
                res.push_back(parts[0] + "." + parts[1] + "." + parts[2] + "." + parts[3]);
            return;
        }
        for (int len = 1; len <= 3 && start + len <= n; ++len) {
            std::string seg = s.substr(start, len);
            if (seg.size() > 1 && seg[0] == '0') break;   // leading zero: no longer segment works either
            if (std::stoi(seg) > 255) break;              // once over 255, longer is also over
            parts.push_back(seg);
            dfs(start + len);
            parts.pop_back();
        }
    };
    dfs(0);
    return res;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <string>
#include <functional>
#include <algorithm>
using std::vector;
using std::string;
static vector<string> canon(vector<string> v) { std::sort(v.begin(), v.end()); return v; }
//__USER__
int main() {
    {
        vector<string> want = {"255.255.11.135","255.255.111.35"};
        if (canon(restoreIpAddresses("25525511135")) != canon(want)) { std::puts("case1"); return 1; }
    }
    {
        vector<string> want = {"0.0.0.0"};
        if (canon(restoreIpAddresses("0000")) != canon(want)) { std::puts("case2"); return 1; }
    }
    {
        vector<string> want = {"1.0.10.23","1.0.102.3","10.1.0.23","10.10.2.3","101.0.2.3"};
        if (canon(restoreIpAddresses("101023")) != canon(want)) { std::puts("case3"); return 1; }
    }
    {
        // too short / too long produce nothing
        if (!restoreIpAddresses("1").empty()) { std::puts("case4a"); return 1; }
        if (!restoreIpAddresses("111111111111111").empty()) { std::puts("case4b"); return 1; }
    }
    std::puts("PASS");
}
```

**Editorial:** The four segments and three dots mean the length must lie between 4 and 12. Backtracking tries segment lengths 1 to 3 at each position, rejecting leading-zero segments longer than one digit and values above 255; because a longer segment only grows the number, both rejections can `break` the loop. An address is recorded only when all four segments are placed and the whole string is consumed. The search tree is tiny (at most `3^3` splits), so this runs in effectively constant time for bounded input.
