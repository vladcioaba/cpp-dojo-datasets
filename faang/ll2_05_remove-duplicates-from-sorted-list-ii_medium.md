## challenge: Remove Duplicates from Sorted List II
tags: linked-list, two-pointers
track: faang
difficulty: medium

Given the `head` of a sorted singly linked list, delete **all** nodes that have a duplicate number, leaving only the numbers that appear exactly once in the original list. Return the head of the still-sorted result.

Constraints: the list has `0 <= n <= 300` nodes, `-100 <= Node.val <= 100`, the list is sorted in non-decreasing order.

Example: `1 -> 2 -> 3 -> 3 -> 4 -> 4 -> 5` → `1 -> 2 -> 5`. Example: `1 -> 1 -> 1 -> 2 -> 3` → `2 -> 3`. Example: `1 -> 1` → empty list.

hint: Because the list is sorted, all copies of a value are consecutive — you can detect a duplicated value by looking only at adjacent nodes.
hint: Use a dummy head and a `prev` pointer to the last kept node; when you find a run of equal values, skip the whole run and reconnect `prev->next` past it, so even the first copy is dropped.

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
    ListNode dummy(0);
    dummy.next = head;
    ListNode* prev = &dummy;
    ListNode* cur = head;
    while (cur) {
        if (cur->next && cur->next->val == cur->val) {
            int v = cur->val;
            while (cur && cur->val == v) cur = cur->next;
            prev->next = cur;
        } else {
            prev = cur;
            cur = cur->next;
        }
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
    if (toVec(deleteDuplicates(build({1,2,3,3,4,4,5}))) != vector<int>({1,2,5})) { std::puts("case1"); return 1; }
    if (toVec(deleteDuplicates(build({1,1,1,2,3})))     != vector<int>({2,3}))   { std::puts("case2"); return 1; }
    if (toVec(deleteDuplicates(build({1,1})))           != vector<int>({}))      { std::puts("case3"); return 1; }
    if (toVec(deleteDuplicates(build({})))              != vector<int>({}))      { std::puts("case4"); return 1; }
    if (toVec(deleteDuplicates(build({1,2,2})))         != vector<int>({1}))     { std::puts("case5"); return 1; }
    if (toVec(deleteDuplicates(build({-1,0,0,0,3})))    != vector<int>({-1,3}))  { std::puts("case6"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** Keep `prev` pointing at the last node confirmed to survive, starting from a dummy so the real head can also be dropped. At each step compare `cur` with its neighbor: if they share a value, advance `cur` past every node equal to that value and reconnect `prev->next` to whatever follows, erasing the entire run. Otherwise the value is unique so far, so accept `cur` and move `prev` forward. Sortedness guarantees each distinct value forms one contiguous block, making a single O(n) pass with O(1) space sufficient.
