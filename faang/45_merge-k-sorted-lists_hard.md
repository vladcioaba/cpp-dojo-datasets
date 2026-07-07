## challenge: Merge k Sorted Lists
tags: heap, linked-list, divide-and-conquer
track: faang
difficulty: hard

You are given an array of `k` linked lists, each sorted in ascending order. Merge all the lists into one sorted linked list and return its head.

Constraints: `0 <= k <= 10^4`, `0 <= lists[i].length <= 500`, `-10^4 <= Node.val <= 10^4`, each list is sorted non-decreasing.

Example: `lists = [[1,4,5],[1,3,4],[2,6]]` -> `[1,1,2,3,4,4,5,6]`. Example: `lists = []` -> `[]`.

hint: At any moment the next node of the answer is the smallest current head across all `k` lists.
hint: A min-heap (`std::priority_queue`) of the current heads lets you pop the minimum in O(log k) and push its successor.
hint: Alternatively pair lists up and merge them two at a time (divide and conquer) for O(N log k) as well; either way handle empty lists and an empty input.

```cpp
// starter
#include <vector>
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};
ListNode* mergeKLists(std::vector<ListNode*>& lists);
```

```cpp
ListNode* mergeKLists(std::vector<ListNode*>& lists) {
    auto cmp = [](ListNode* a, ListNode* b) { return a->val > b->val; };
    std::priority_queue<ListNode*, std::vector<ListNode*>, decltype(cmp)> pq(cmp);
    for (ListNode* l : lists)
        if (l) pq.push(l);
    ListNode dummy(0);
    ListNode* tail = &dummy;
    while (!pq.empty()) {
        ListNode* node = pq.top(); pq.pop();
        tail->next = node;
        tail = node;
        if (node->next) pq.push(node->next);
    }
    tail->next = nullptr;
    return dummy.next;
}
```

```cpp
// harness
#include <cstdio>
#include <vector>
#include <queue>
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
    { vector<ListNode*> l{build({1,4,5}), build({1,3,4}), build({2,6})};
      if (toVec(mergeKLists(l)) != vector<int>({1,1,2,3,4,4,5,6})) { std::puts("case1"); return 1; } }
    { vector<ListNode*> l{};
      if (toVec(mergeKLists(l)) != vector<int>({}))                { std::puts("case2"); return 1; } }
    { vector<ListNode*> l{build({})};
      if (toVec(mergeKLists(l)) != vector<int>({}))                { std::puts("case3"); return 1; } }
    { vector<ListNode*> l{build({2,4,6,8})};
      if (toVec(mergeKLists(l)) != vector<int>({2,4,6,8}))         { std::puts("case4"); return 1; } }
    { vector<ListNode*> l{build({}), build({-1,0,1}), build({})};
      if (toVec(mergeKLists(l)) != vector<int>({-1,0,1}))          { std::puts("case5"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Keep a min-heap of the current head of every non-empty list. Repeatedly pop the smallest node, splice it onto the result tail, and push its successor. Each of the `N` total nodes enters and leaves the heap once, so the cost is O(N log k) time and O(k) extra space for the heap. Empty and null lists are simply never pushed.
