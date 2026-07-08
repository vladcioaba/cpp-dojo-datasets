## challenge: Convert Binary Number in a Linked List to Integer
tags: linked-list, math
track: faang
difficulty: easy

Given `head`, the reference node of a singly linked list, each node holds either `0` or `1`. The list is the binary representation of a number, most-significant bit first. Return the decimal value of that number.

Constraints: the list has `1 <= n <= 30` nodes, each `Node.val` is `0` or `1`.

Example: `1 -> 0 -> 1` → `5` (binary `101`). Example: `0` → `0`. Example: `1 -> 0 -> 0 -> 1 -> 0 -> 0 -> 1 -> 1 -> 1 -> 0 -> 0 -> 0 -> 0 -> 0 -> 0` → `18880`.

hint: Reading bits most-significant first is exactly how you'd evaluate a binary string scanning left to right.
hint: Keep a running total; for each node double the accumulator and add the current bit — `num = num * 2 + node->val`.

```cpp
// starter
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};
int getDecimalValue(ListNode* head);
```

```cpp
int getDecimalValue(ListNode* head) {
    int num = 0;
    while (head) {
        num = num * 2 + head->val;
        head = head->next;
    }
    return num;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};
static ListNode* build(const vector<int>& v) {
    ListNode dummy(0); ListNode* t = &dummy;
    for (int x : v) { t->next = new ListNode(x); t = t->next; }
    return dummy.next;
}
//__USER__
int main() {
    if (getDecimalValue(build({1,0,1})) != 5) { std::puts("case1"); return 1; }
    if (getDecimalValue(build({0}))     != 0) { std::puts("case2"); return 1; }
    if (getDecimalValue(build({1}))     != 1) { std::puts("case3"); return 1; }
    if (getDecimalValue(build({1,0,0,1,0,0,1,1,1,0,0,0,0,0,0})) != 18880) { std::puts("case4"); return 1; }
    if (getDecimalValue(build({1,1,1,1,1})) != 31) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Evaluate the binary number in a single left-to-right pass, treating the accumulator as a partially built value. Doubling before adding each bit — Horner's method in base 2 — shifts the running total left by one position, so after processing all nodes it holds the full decimal value. O(n) time, O(1) space.
