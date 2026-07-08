## challenge: Remove Linked List Elements
tags: linked-list, recursion
track: faang
difficulty: easy

Given the `head` of a singly linked list and an integer `val`, remove every node whose value equals `val` and return the new head.

Constraints: the list has `0 <= n <= 10^4` nodes, `1 <= Node.val <= 50`, `0 <= val <= 50`.

Example: `1 -> 2 -> 6 -> 3 -> 4 -> 5 -> 6`, `val = 6` → `1 -> 2 -> 3 -> 4 -> 5`. Example: `7 -> 7 -> 7 -> 7`, `val = 7` → empty list. Example: empty list, `val = 1` → empty list.

hint: The head itself might need removing, and it might need removing several times in a row.
hint: A dummy node placed before the head turns "delete the head" into the same case as deleting any interior node.
hint: Keep a cursor on the node *before* the one you inspect; when `cur->next->val == val`, splice it out with `cur->next = cur->next->next` without advancing, otherwise move forward.

```cpp
// starter
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};
ListNode* removeElements(ListNode* head, int val);
```

```cpp
ListNode* removeElements(ListNode* head, int val) {
    ListNode dummy(0);
    dummy.next = head;
    ListNode* cur = &dummy;
    while (cur->next) {
        if (cur->next->val == val)
            cur->next = cur->next->next;
        else
            cur = cur->next;
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
    if (toVec(removeElements(build({1,2,6,3,4,5,6}), 6)) != vector<int>({1,2,3,4,5})) { std::puts("case1"); return 1; }
    if (toVec(removeElements(build({7,7,7,7}), 7))       != vector<int>({}))          { std::puts("case2"); return 1; }
    if (toVec(removeElements(build({}), 1))              != vector<int>({}))          { std::puts("case3"); return 1; }
    if (toVec(removeElements(build({1,2,3}), 4))         != vector<int>({1,2,3}))     { std::puts("case4"); return 1; }
    if (toVec(removeElements(build({6,1,6}), 6))         != vector<int>({1}))         { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** A dummy sentinel before the head unifies head deletion with interior deletion, so no special-casing is needed even when leading nodes (or every node) match. Keep a cursor on the predecessor: if the next node matches, bypass it and stay put so runs of matches are all removed; otherwise advance. One pass, O(n) time, O(1) space.
