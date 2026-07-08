## challenge: Add Two Numbers II
tags: linked-list, math, stack
track: faang
difficulty: medium

You are given two non-empty singly linked lists representing two non-negative integers. The most-significant digit comes first, one digit per node. Add the two numbers and return the sum as a linked list, again most-significant digit first. Neither input has leading zeros except the number `0` itself.

Constraints: each list has `1 <= n <= 100` nodes, `0 <= Node.val <= 9`.

Example: `l1 = 7 -> 2 -> 4 -> 3` (7243), `l2 = 5 -> 6 -> 4` (564) → `7 -> 8 -> 0 -> 7` (7807). Example: `l1 = 2 -> 4 -> 3`, `l2 = 5 -> 6 -> 4` → `8 -> 0 -> 7`. Example: `l1 = 0`, `l2 = 0` → `0`.

hint: Addition proceeds from the least-significant digit, but here the least-significant digits are at the *tails* — you need to process the lists back to front.
hint: Push each list's digits onto a stack (or into an array) so you can pop them least-significant first; add with a carry and prepend each new digit to the front of the result.

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
    std::vector<int> s1, s2;
    for (ListNode* p = l1; p; p = p->next) s1.push_back(p->val);
    for (ListNode* p = l2; p; p = p->next) s2.push_back(p->val);
    int i = (int)s1.size() - 1, j = (int)s2.size() - 1, carry = 0;
    ListNode* head = nullptr;
    while (i >= 0 || j >= 0 || carry) {
        int sum = carry;
        if (i >= 0) sum += s1[i--];
        if (j >= 0) sum += s2[j--];
        ListNode* node = new ListNode(sum % 10);
        node->next = head;
        head = node;
        carry = sum / 10;
    }
    return head;
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
    if (toVec(addTwoNumbers(build({7,2,4,3}), build({5,6,4}))) != vector<int>({7,8,0,7})) { std::puts("case1"); return 1; }
    if (toVec(addTwoNumbers(build({2,4,3}), build({5,6,4})))   != vector<int>({8,0,7}))   { std::puts("case2"); return 1; }
    if (toVec(addTwoNumbers(build({0}), build({0})))           != vector<int>({0}))       { std::puts("case3"); return 1; }
    if (toVec(addTwoNumbers(build({9,9,9}), build({1})))       != vector<int>({1,0,0,0})) { std::puts("case4"); return 1; }
    if (toVec(addTwoNumbers(build({5}), build({5})))           != vector<int>({1,0}))     { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Grade-school addition works from the least-significant digit, which lives at the tail of each forward-ordered list. Rather than reverse the inputs, dump each list's digits into an array and index from the back, adding aligned place values plus a running carry. Because the result is also most-significant first, prepend each freshly computed digit to the front of the output list, which naturally leaves a final carry as the new leading digit. O(m + n) time, O(m + n) space.
