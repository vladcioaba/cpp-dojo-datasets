## challenge: Multiply Strings
tags: math, string, simulation
track: faang
difficulty: hard

Given two non-negative integers `num1` and `num2` represented as strings, return their product, also as a string. You must not convert the inputs to a built-in integer type or use a big-integer library — perform the multiplication digit by digit.

Constraints: `1 <= num1.length, num2.length <= 200`, both strings contain only digits `0-9`, and neither has a leading zero except the value `"0"` itself.

Example: `num1 = "2", num2 = "3"` → `"6"`. Example: `num1 = "123", num2 = "456"` → `"56088"`.

hint: Reproduce grade-school multiplication: the product of `num1[i]` and `num2[j]` contributes to a fixed pair of positions in the result.
hint: A product of an `m`-digit and an `n`-digit number has at most `m + n` digits. The digit product `num1[i] * num2[j]` lands at result positions `i + j` (carry) and `i + j + 1` (units).
hint: Accumulate into an integer array of size `m + n` from least significant digit outward, propagating carries, then strip leading zeros when building the string.

```cpp
// starter
#include <string>
std::string multiply(std::string num1, std::string num2);
```

```cpp
std::string multiply(std::string num1, std::string num2) {
    if (num1 == "0" || num2 == "0") return "0";
    int m = (int)num1.size(), n = (int)num2.size();
    std::vector<int> pos(m + n, 0);
    for (int i = m - 1; i >= 0; --i) {
        for (int j = n - 1; j >= 0; --j) {
            int mul = (num1[i] - '0') * (num2[j] - '0');
            int p1 = i + j, p2 = i + j + 1;
            int sum = mul + pos[p2];
            pos[p2] = sum % 10;
            pos[p1] += sum / 10;
        }
    }
    std::string res;
    for (int d : pos) {
        if (!(res.empty() && d == 0)) res.push_back((char)(d + '0'));
    }
    return res.empty() ? "0" : res;
}
```

```cpp
// harness
#include <cstdio>
#include <string>
#include <vector>
//__USER__
int main() {
    if (multiply("2", "3")     != "6")      { std::puts("case1"); return 1; }
    if (multiply("123", "456") != "56088")  { std::puts("case2"); return 1; }
    if (multiply("0", "52")    != "0")      { std::puts("case3"); return 1; }
    if (multiply("999", "999") != "998001") { std::puts("case4"); return 1; }
    if (multiply("123456789", "987654321") != "121932631112635269") { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Long multiplication generalized to arbitrary length. The product of two numbers with `m` and `n` digits has at most `m + n` digits, so an integer buffer of that size holds the answer. Multiplying digit `num1[i]` by `num2[j]` yields a value whose units digit belongs at position `i + j + 1` and whose carry belongs at `i + j`; adding into the buffer and carrying immediately keeps every slot in `0..9` by the end. Finally, skip leading zeros while emitting the digit string (the all-zero case is short-circuited up front). O(m·n) time, O(m+n) space.
