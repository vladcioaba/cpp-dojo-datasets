## challenge: Remove Duplicates from Sorted List
tags: linked-list
track: faang
difficulty: easy

Given the `head` of a sorted singly linked list, delete all duplicates so that each value appears only once. Return the head of the still-sorted list.

Constraints: the list has `0 <= n <= 300` nodes, sorted non-decreasing, `-100 <= Node.val <= 100`.

Example: `1 -> 1 -> 2` → `1 -> 2`. Example: `1 -> 1 -> 2 -> 3 -> 3` → `1 -> 2 -> 3`. Example: empty list → empty list.

hint: Because the list is sorted, all equal values are adjacent, so you only ever compare neighbors.
hint: Walk a single cursor along the list comparing `cur->val` with `cur->next->val`.
hint: On a match, splice out the successor with `cur->next = cur->next->next` and do NOT advance — a run of three or more equal values needs repeated splicing; only advance when the values differ.

```cpp
// starter
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};
ListNode* deleteDuplicates(ListNode* head);
```

```cpp
ListNode* deleteDuplicates(ListNode* head) {
    ListNode* cur = head;
    while (cur && cur->next) {
        if (cur->val == cur->next->val)
            cur->next = cur->next->next;
        else
            cur = cur->next;
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
    if (toVec(deleteDuplicates(build({1,1,2})))     != vector<int>({1,2}))   { std::puts("case1"); return 1; }
    if (toVec(deleteDuplicates(build({1,1,2,3,3}))) != vector<int>({1,2,3})) { std::puts("case2"); return 1; }
    if (toVec(deleteDuplicates(build({})))          != vector<int>({}))      { std::puts("case3"); return 1; }
    if (toVec(deleteDuplicates(build({1,1,1})))     != vector<int>({1}))     { std::puts("case4"); return 1; }
    if (toVec(deleteDuplicates(build({-3,-3,0,0,0,5}))) != vector<int>({-3,0,5})) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Sorted order guarantees duplicates are contiguous, so a single cursor comparing each node with its neighbor suffices. When two neighbors are equal, unlink the second and keep the cursor put so consecutive duplicates collapse; otherwise step forward. The list is traversed once in O(n) time with O(1) extra space, mutating pointers in place.
