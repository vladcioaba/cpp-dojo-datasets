## challenge: Merge Two Sorted Lists
tags: linked-list, two-pointers
track: faang
difficulty: easy

Merge two sorted singly linked lists into one sorted list by splicing their nodes together. Return the head of the merged list.

Constraints: each list has `0 <= n <= 50` nodes, sorted non-decreasing, `-100 <= Node.val <= 100`.

Example: `l1 = 1->2->4`, `l2 = 1->3->4` → `1->1->2->3->4->4`. Example: both empty → empty.

```cpp
// starter
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};
ListNode* mergeTwoLists(ListNode* l1, ListNode* l2);
```

```cpp
ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
    ListNode dummy(0);
    ListNode* tail = &dummy;
    while (l1 && l2) {
        if (l1->val <= l2->val) { tail->next = l1; l1 = l1->next; }
        else                    { tail->next = l2; l2 = l2->next; }
        tail = tail->next;
    }
    tail->next = l1 ? l1 : l2;
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
    if (toVec(mergeTwoLists(build({1,2,4}), build({1,3,4}))) != vector<int>({1,1,2,3,4,4})) { std::puts("case1"); return 1; }
    if (toVec(mergeTwoLists(build({}), build({})))           != vector<int>({}))            { std::puts("case2"); return 1; }
    if (toVec(mergeTwoLists(build({}), build({0})))          != vector<int>({0}))           { std::puts("case3"); return 1; }
    if (toVec(mergeTwoLists(build({1,3,5}), build({2,4})))   != vector<int>({1,2,3,4,5}))   { std::puts("case4"); return 1; }
    std::puts("PASS");
}
```
