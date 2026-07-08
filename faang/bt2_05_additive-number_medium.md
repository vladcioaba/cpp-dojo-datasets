## challenge: Additive Number
tags: backtracking, string
track: faang
difficulty: medium

An additive number is a string of digits that can be split into an additive sequence: a sequence of at least three numbers where, apart from the first two, each number equals the sum of the two preceding it. Given a string `num` containing only digits, return `true` if it is an additive number. Numbers in the sequence may not have redundant leading zeros, so `"1"` is fine but a multi-digit number like `"03"` is not; a lone `"0"` is allowed.

Constraints: `1 <= num.length <= 35`, `num` consists only of digits `'0'`–`'9'`. The numbers can exceed 64-bit range, so add them as decimal strings.

Example: `num = "112358"` -> `true` (`1, 1, 2, 3, 5, 8`). Example: `num = "199100199"` -> `true` (`1, 99, 100, 199`). Example: `num = "1023"` -> `false`.

hint: Once the first two numbers are fixed, the entire remaining sequence is forced — each next number is the sum of the previous two.
hint: Try every split for the first and second numbers (respecting the no-leading-zero rule), then greedily verify that the rest of the string matches the forced sums.
hint: Sums can overflow 64-bit integers for long inputs, so add the operands as decimal strings.

```cpp
// starter
#include <string>
bool isAdditiveNumber(std::string num);
```

```cpp
bool isAdditiveNumber(std::string num) {
    int n = (int)num.size();
    auto add = [](const std::string& a, const std::string& b) {
        std::string r;
        int i = (int)a.size() - 1, j = (int)b.size() - 1, carry = 0;
        while (i >= 0 || j >= 0 || carry) {
            int s = carry + (i >= 0 ? a[i--] - '0' : 0) + (j >= 0 ? b[j--] - '0' : 0);
            r.push_back(char('0' + s % 10));
            carry = s / 10;
        }
        std::reverse(r.begin(), r.end());
        return r;
    };
    std::function<bool(int, const std::string&, const std::string&)> dfs =
        [&](int start, const std::string& a, const std::string& b) -> bool {
        if (start == n) return true;
        std::string sum = add(a, b);
        if (start + (int)sum.size() > n) return false;
        if (num.compare(start, sum.size(), sum) != 0) return false;
        return dfs(start + (int)sum.size(), b, sum);
    };
    for (int i = 1; i < n; ++i) {
        if (i > 1 && num[0] == '0') break;
        std::string a = num.substr(0, i);
        for (int j = i + 1; j < n; ++j) {
            if (j - i > 1 && num[i] == '0') break;
            std::string b = num.substr(i, j - i);
            if (dfs(j, a, b)) return true;
        }
    }
    return false;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
#include <algorithm>
#include <functional>
using std::string;
//__USER__
int main() {
    struct T { const char* s; bool want; };
    T tests[] = {
        {"112358", true}, {"199100199", true}, {"1023", false},
        {"000", true}, {"101", true}, {"1", false}, {"10", false},
        {"11235813213455", true}, {"1203", false}, {"123456789", false},
        {"0235813", false}
    };
    for (auto& t : tests) {
        if (isAdditiveNumber(string(t.s)) != t.want) {
            std::printf("fail:%s\n", t.s); return 1;
        }
    }
    std::puts("PASS");
}
```

**Editorial:** Only the first two numbers are free choices; enumerate their splits (rejecting redundant leading zeros), then let each subsequent number be forced as the sum of the previous two and check that it matches the string. Adding on decimal strings avoids overflow when the sequence grows large. With `n` up to 35 there are O(n^2) starting pairs and each verification is linear, so O(n^3) overall.
