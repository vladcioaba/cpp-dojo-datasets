## challenge: Reverse Linked List II
tags: linked-list, two-pointers
track: faang
difficulty: medium

Given the `head` of a singly linked list and two integers `left` and `right` with `left <= right`, reverse the nodes from position `left` to position `right` (1-indexed) and return the head. Do it in one pass.

Constraints: the list has `1 <= n <= 500` nodes, `-500 <= Node.val <= 500`, `1 <= left <= right <= n`.

Example: `1 -> 2 -> 3 -> 4 -> 5`, `left = 2`, `right = 4` → `1 -> 4 -> 3 -> 2 -> 5`. Example: `5`, `left = 1`, `right = 1` → `5`. Example: `3 -> 5`, `left = 1`, `right = 2` → `5 -> 3`.

hint: A dummy node in front of the head lets you treat `left = 1` (reversing from the very start) with the same code as any interior range.
hint: Walk to the node just before position `left`; then repeatedly pull the node right after your cursor to the front of the sublist — the classic "head-insertion" reversal — exactly `right - left` times.

```cpp
// starter
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};
ListNode* reverseBetween(ListNode* head, int left, int right);
```

```cpp
ListNode* reverseBetween(ListNode* head, int left, int right) {
    ListNode dummy(0);
    dummy.next = head;
    ListNode* prev = &dummy;
    for (int i = 0; i < left - 1; ++i) prev = prev->next;
    ListNode* cur = prev->next;
    for (int i = 0; i < right - left; ++i) {
        ListNode* moved = cur->next;
        cur->next = moved->next;
        moved->next = prev->next;
        prev->next = moved;
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
    if (toVec(reverseBetween(build({1,2,3,4,5}), 2, 4)) != vector<int>({1,4,3,2,5})) { std::puts("case1"); return 1; }
    if (toVec(reverseBetween(build({5}), 1, 1))         != vector<int>({5}))         { std::puts("case2"); return 1; }
    if (toVec(reverseBetween(build({3,5}), 1, 2))       != vector<int>({5,3}))       { std::puts("case3"); return 1; }
    if (toVec(reverseBetween(build({1,2,3,4,5}), 1, 5)) != vector<int>({5,4,3,2,1})) { std::puts("case4"); return 1; }
    if (toVec(reverseBetween(build({1,2,3,4,5}), 3, 5)) != vector<int>({1,2,5,4,3})) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** A dummy head absorbs the awkward case where `left = 1`, so `prev` — the node immediately before the reversed segment — always exists. Fixing `cur` as the first node of the segment, each iteration lifts the node just after `cur` and re-inserts it directly behind `prev`, growing the reversed prefix one node at a time. After `right - left` such moves the segment is fully flipped while everything outside it is untouched. One pass, O(n) time, O(1) space.
