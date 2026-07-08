## challenge: Add Two Numbers
tags: linked-list, math, recursion
track: faang
difficulty: medium

You are given two non-empty singly linked lists representing two non-negative integers. The digits are stored in reverse order, one digit per node. Add the two numbers and return the sum as a linked list, also in reverse-order digits. Neither number has leading zeros except the number 0 itself.

Constraints: each list has `1 <= n <= 100` nodes, `0 <= Node.val <= 9`.

Example: `l1 = 2 -> 4 -> 3` (342), `l2 = 5 -> 6 -> 4` (465) → `7 -> 0 -> 8` (807). Example: `l1 = 0`, `l2 = 0` → `0`. Example: `l1 = 9 -> 9 -> 9 -> 9 -> 9 -> 9 -> 9`, `l2 = 9 -> 9 -> 9 -> 9` → `8 -> 9 -> 9 -> 9 -> 0 -> 0 -> 0 -> 1`.

hint: Reverse-order storage is a gift: node `i` of both lists holds the same place value, so you add them left to right like grade-school addition.
hint: Track a running carry; each output digit is `(a + b + carry) % 10` and the new carry is `(a + b + carry) / 10`.
hint: Loop while either list has nodes remaining OR the carry is non-zero, treating a missing node as digit 0 — that final carry-only step is what creates the leading `1` in `999...+...`.

```cpp
// starter
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};
ListNode* addTwoNumbers(ListNode* l1, ListNode* l2);
```

```cpp
ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
    ListNode dummy(0);
    ListNode* tail = &dummy;
    int carry = 0;
    while (l1 || l2 || carry) {
        int sum = carry;
        if (l1) { sum += l1->val; l1 = l1->next; }
        if (l2) { sum += l2->val; l2 = l2->next; }
        carry = sum / 10;
        tail->next = new ListNode(sum % 10);
        tail = tail->next;
    }
    return dummy.next;
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
static vector<int> toVec(ListNode* h) {
    vector<int> out; while (h) { out.push_back(h->val); h = h->next; } return out;
}
//__USER__
int main() {
    if (toVec(addTwoNumbers(build({2,4,3}), build({5,6,4}))) != vector<int>({7,0,8})) { std::puts("case1"); return 1; }
    if (toVec(addTwoNumbers(build({0}), build({0})))         != vector<int>({0}))     { std::puts("case2"); return 1; }
    if (toVec(addTwoNumbers(build({9,9,9,9,9,9,9}), build({9,9,9,9}))) != vector<int>({8,9,9,9,0,0,0,1})) { std::puts("case3"); return 1; }
    if (toVec(addTwoNumbers(build({5}), build({5})))         != vector<int>({0,1}))   { std::puts("case4"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Because digits are least-significant first, iterate both lists in lockstep adding aligned place values plus a carry, emitting `sum % 10` per node and propagating `sum / 10`. Continuing the loop while a carry remains after both lists are exhausted appends the final overflow digit. A dummy head simplifies building the result. O(max(m, n)) time, O(max(m, n)) space for the output.
