## challenge: Sort List
tags: linked-list, sorting, divide-and-conquer, merge-sort
track: faang
difficulty: hard

Given the `head` of a singly linked list, sort it in ascending order and return the new head. Aim for O(n log n) time.

Constraints: the list has `0 <= n <= 5 * 10^4` nodes, `-10^5 <= Node.val <= 10^5`.

Example: `4 -> 2 -> 1 -> 3` → `1 -> 2 -> 3 -> 4`. Example: `-1 -> 5 -> 3 -> 4 -> 0` → `-1 -> 0 -> 3 -> 4 -> 5`. Example: empty list → empty list.

hint: Merge sort adapts to linked lists far better than array sorts — splitting and merging are pure pointer manipulation with no random access needed.
hint: Split the list into two halves using slow/fast pointers (cut the first half's tail off), recursively sort each half, then merge two sorted lists the same way you would in "merge two sorted lists".

```cpp
// starter
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};
ListNode* sortList(ListNode* head);
```

```cpp
ListNode* sortList(ListNode* head) {
    if (!head || !head->next) return head;
    // split into two halves
    ListNode* slow = head;
    ListNode* fast = head->next;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
    }
    ListNode* mid = slow->next;
    slow->next = nullptr;
    ListNode* left = sortList(head);
    ListNode* right = sortList(mid);
    // merge
    ListNode dummy(0);
    ListNode* tail = &dummy;
    while (left && right) {
        if (left->val <= right->val) { tail->next = left; left = left->next; }
        else                         { tail->next = right; right = right->next; }
        tail = tail->next;
    }
    tail->next = left ? left : right;
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
    if (toVec(sortList(build({4,2,1,3})))     != vector<int>({1,2,3,4}))       { std::puts("case1"); return 1; }
    if (toVec(sortList(build({-1,5,3,4,0})))  != vector<int>({-1,0,3,4,5}))    { std::puts("case2"); return 1; }
    if (toVec(sortList(build({})))            != vector<int>({}))              { std::puts("case3"); return 1; }
    if (toVec(sortList(build({1})))           != vector<int>({1}))             { std::puts("case4"); return 1; }
    if (toVec(sortList(build({3,3,1,2,1})))   != vector<int>({1,1,2,3,3}))     { std::puts("case5"); return 1; }
    if (toVec(sortList(build({5,4,3,2,1})))   != vector<int>({1,2,3,4,5}))     { std::puts("case6"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Merge sort is the natural fit for lists because both of its phases — splitting and merging — are sequential pointer operations, unlike quicksort's random pivots or an array's index arithmetic. Slow/fast pointers find the midpoint; starting `fast` one node ahead guarantees the left half is never empty for lists of length ≥ 2, avoiding infinite recursion. After severing the halves and sorting each recursively, a standard two-pointer merge stitches them back in order using a dummy head. The recursion has depth O(log n) and each level does O(n) merging work, so O(n log n) time with O(log n) recursion-stack space.
