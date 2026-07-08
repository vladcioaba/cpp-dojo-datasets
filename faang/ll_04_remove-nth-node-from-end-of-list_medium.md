## challenge: Remove Nth Node From End of List
tags: linked-list, two-pointers
track: faang
difficulty: medium

Given the `head` of a singly linked list, remove the `n`-th node from the end of the list and return its head. Do it in one pass.

Constraints: the list has `1 <= sz <= 30` nodes, `1 <= Node.val <= 100`, `1 <= n <= sz`.

Example: `1 -> 2 -> 3 -> 4 -> 5`, `n = 2` → `1 -> 2 -> 3 -> 5`. Example: `1`, `n = 1` → empty list. Example: `1 -> 2`, `n = 1` → `1`.

hint: "N-th from the end" is awkward directly, but a gap of `n` between two pointers converts it to a front-relative position.
hint: Advance a fast pointer `n` nodes ahead first, then move fast and a slow pointer together until fast reaches the last node.
hint: Anchor slow on a dummy node before the head so slow lands on the predecessor of the target — even when the target is the head itself — then splice with `slow->next = slow->next->next`.

```cpp
// starter
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};
ListNode* removeNthFromEnd(ListNode* head, int n);
```

```cpp
ListNode* removeNthFromEnd(ListNode* head, int n) {
    ListNode dummy(0);
    dummy.next = head;
    ListNode* fast = &dummy;
    ListNode* slow = &dummy;
    for (int i = 0; i < n; ++i) fast = fast->next;
    while (fast->next) {
        fast = fast->next;
        slow = slow->next;
    }
    slow->next = slow->next->next;
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
    if (toVec(removeNthFromEnd(build({1,2,3,4,5}), 2)) != vector<int>({1,2,3,5})) { std::puts("case1"); return 1; }
    if (toVec(removeNthFromEnd(build({1}), 1))         != vector<int>({}))        { std::puts("case2"); return 1; }
    if (toVec(removeNthFromEnd(build({1,2}), 1))       != vector<int>({1}))       { std::puts("case3"); return 1; }
    if (toVec(removeNthFromEnd(build({1,2}), 2))       != vector<int>({2}))       { std::puts("case4"); return 1; }
    if (toVec(removeNthFromEnd(build({1,2,3,4,5}), 5)) != vector<int>({2,3,4,5})) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Open a gap of `n` edges between a fast and slow pointer by moving fast ahead first. Advancing both until fast hits the last node leaves slow on the node just before the target, so a single splice removes it. Starting both at a dummy sentinel handles removing the head uniformly. One pass, O(n) time, O(1) space.
