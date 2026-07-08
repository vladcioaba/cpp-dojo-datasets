## challenge: Merge In Between Linked Lists
tags: linked-list
track: faang
difficulty: easy

You are given two singly linked lists `list1` and `list2`, and two integers `a` and `b`. Remove `list1`'s nodes from position `a` to position `b` inclusive (0-indexed), and splice `list2` into the gap. Return the head of the resulting list.

Constraints: `3 <= list1.length <= 10^4`, `1 <= a <= b < list1.length - 1`, `1 <= list2.length <= 10^4`.

Example: `list1 = 0 -> 1 -> 2 -> 3 -> 4 -> 5`, `a = 3`, `b = 4`, `list2 = 1000000 -> 1000001 -> 1000002` → `0 -> 1 -> 2 -> 1000000 -> 1000001 -> 1000002 -> 5`. Example: `list1 = 0 -> 1 -> 2 -> 3 -> 4 -> 5 -> 6`, `a = 2`, `b = 5`, `list2 = 1000000 -> 1000001 -> 1000002 -> 1000003 -> 1000004` → `0 -> 1 -> 1000000 -> 1000001 -> 1000002 -> 1000003 -> 1000004 -> 6`.

hint: You need two anchors in `list1`: the node just before position `a`, and the node just after position `b`.
hint: Walk `a - 1` steps to reach the node before the cut, then continue to the node at `b + 1`; wire the "before" node to `list2`'s head and `list2`'s tail to the "after" node.

```cpp
// starter
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};
ListNode* mergeInBetween(ListNode* list1, int a, int b, ListNode* list2);
```

```cpp
ListNode* mergeInBetween(ListNode* list1, int a, int b, ListNode* list2) {
    ListNode* before = list1;
    for (int i = 0; i < a - 1; ++i) before = before->next;
    ListNode* after = before;
    for (int i = 0; i < b - a + 2; ++i) after = after->next;
    before->next = list2;
    ListNode* tail = list2;
    while (tail->next) tail = tail->next;
    tail->next = after;
    return list1;
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
    { ListNode* r = mergeInBetween(build({0,1,2,3,4,5}), 3, 4, build({1000000,1000001,1000002}));
      if (toVec(r) != vector<int>({0,1,2,1000000,1000001,1000002,5})) { std::puts("case1"); return 1; } }
    { ListNode* r = mergeInBetween(build({0,1,2,3,4,5,6}), 2, 5, build({1000000,1000001,1000002,1000003,1000004}));
      if (toVec(r) != vector<int>({0,1,1000000,1000001,1000002,1000003,1000004,6})) { std::puts("case2"); return 1; } }
    { ListNode* r = mergeInBetween(build({0,1,2,3}), 1, 2, build({9}));
      if (toVec(r) != vector<int>({0,9,3})) { std::puts("case3"); return 1; } }
    { ListNode* r = mergeInBetween(build({0,1,2,3,4}), 1, 1, build({7,8}));
      if (toVec(r) != vector<int>({0,7,8,2,3,4})) { std::puts("case4"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** The operation is pure pointer surgery: find the node at index `a - 1` (call it `before`) and the node at index `b + 1` (call it `after`) by counting steps from the head. Reaching `after` from `before` takes `(b + 1) - (a - 1) = b - a + 2` hops. Point `before->next` at `list2`, walk to `list2`'s tail, and point that tail at `after`. The removed span is simply no longer reachable. O(list1.length + list2.length) time, O(1) space.
