## challenge: Swap Nodes in Pairs
tags: linked-list, recursion
track: faang
difficulty: medium

Given the `head` of a singly linked list, swap every two adjacent nodes and return the new head. You must swap the nodes themselves (rewire the pointers), not merely exchange the values. If the list has an odd length, the final lone node stays in place.

Constraints: the list has `0 <= n <= 100` nodes, `0 <= Node.val <= 100`.

Example: `1 -> 2 -> 3 -> 4` → `2 -> 1 -> 4 -> 3`. Example: `1 -> 2 -> 3` → `2 -> 1 -> 3`. Example: empty list → empty list.

hint: Each swap rewires three pointers: the predecessor's `next`, and the two nodes' `next` fields.
hint: A dummy node before the head gives a stable predecessor for the very first pair.
hint: With `prev`, `first = prev->next`, `second = first->next`: set `first->next = second->next`, `second->next = first`, `prev->next = second`, then move `prev` to `first` and repeat while another full pair exists.

```cpp
// starter
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};
ListNode* swapPairs(ListNode* head);
```

```cpp
ListNode* swapPairs(ListNode* head) {
    ListNode dummy(0);
    dummy.next = head;
    ListNode* prev = &dummy;
    while (prev->next && prev->next->next) {
        ListNode* first = prev->next;
        ListNode* second = first->next;
        first->next = second->next;
        second->next = first;
        prev->next = second;
        prev = first;
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
    if (toVec(swapPairs(build({1,2,3,4}))) != vector<int>({2,1,4,3})) { std::puts("case1"); return 1; }
    if (toVec(swapPairs(build({})))        != vector<int>({}))        { std::puts("case2"); return 1; }
    if (toVec(swapPairs(build({1})))       != vector<int>({1}))       { std::puts("case3"); return 1; }
    if (toVec(swapPairs(build({1,2,3})))   != vector<int>({2,1,3}))   { std::puts("case4"); return 1; }
    if (toVec(swapPairs(build({1,2,3,4,5,6}))) != vector<int>({2,1,4,3,6,5})) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Walk the list a pair at a time behind a dummy sentinel. For each adjacent `first`/`second`, reconnect the predecessor to `second`, point `second` back at `first`, and forward `first` to whatever followed `second`. Advancing `prev` to `first` positions it before the next pair; the loop guard `prev->next && prev->next->next` leaves a trailing odd node untouched. O(n) time, O(1) space.
