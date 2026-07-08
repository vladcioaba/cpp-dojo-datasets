## challenge: Split Linked List in Parts
tags: linked-list
track: faang
difficulty: medium

Given the `head` of a singly linked list and an integer `k`, split the list into `k` consecutive parts. The parts should have sizes as equal as possible: no two parts differ in size by more than one, and earlier parts are never smaller than later ones. Some parts may be empty (`nullptr`). Return an array of the `k` part heads, in order.

Constraints: the list has `0 <= n <= 1000` nodes, `0 <= Node.val <= 1000`, `1 <= k <= 50`.

Example: `1 -> 2 -> 3`, `k = 5` → `[[1], [2], [3], [], []]`. Example: `1 -> 2 -> 3 -> 4 -> 5 -> 6 -> 7 -> 8 -> 9 -> 10`, `k = 3` → `[[1,2,3,4], [5,6,7], [8,9,10]]`.

hint: Count the nodes first; with `n` nodes and `k` parts, each part has `n / k` nodes and the first `n % k` parts get one extra.
hint: Walk the list cutting off each computed run by setting the last node's `next` to `nullptr` before moving on, and leave the remaining slots as `nullptr`.

```cpp
// starter
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};
std::vector<ListNode*> splitListToParts(ListNode* head, int k);
```

```cpp
std::vector<ListNode*> splitListToParts(ListNode* head, int k) {
    int n = 0;
    for (ListNode* p = head; p; p = p->next) ++n;
    int size = n / k, extra = n % k;
    std::vector<ListNode*> res(k, nullptr);
    ListNode* cur = head;
    for (int i = 0; i < k && cur; ++i) {
        res[i] = cur;
        int partSize = size + (i < extra ? 1 : 0);
        for (int j = 0; j < partSize - 1; ++j) cur = cur->next;
        ListNode* nxt = cur->next;
        cur->next = nullptr;
        cur = nxt;
    }
    return res;
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
static vector<vector<int>> toParts(const vector<ListNode*>& parts) {
    vector<vector<int>> out; for (ListNode* p : parts) out.push_back(toVec(p)); return out;
}
//__USER__
int main() {
    if (toParts(splitListToParts(build({1,2,3}), 5))
        != vector<vector<int>>({{1},{2},{3},{},{}})) { std::puts("case1"); return 1; }
    if (toParts(splitListToParts(build({1,2,3,4,5,6,7,8,9,10}), 3))
        != vector<vector<int>>({{1,2,3,4},{5,6,7},{8,9,10}})) { std::puts("case2"); return 1; }
    if (toParts(splitListToParts(build({}), 3))
        != vector<vector<int>>({{},{},{}})) { std::puts("case3"); return 1; }
    if (toParts(splitListToParts(build({1,2,3,4}), 2))
        != vector<vector<int>>({{1,2},{3,4}})) { std::puts("case4"); return 1; }
    if (toParts(splitListToParts(build({7}), 1))
        != vector<vector<int>>({{7}})) { std::puts("case5"); return 1; }
    std::puts("PASS");
}
```

**Editorial:** First count `n`. Integer division gives the base part size `n / k`, and the remainder `n % k` says how many of the leading parts carry one extra node. Traverse once, and for part `i` advance `partSize - 1` nodes to reach its last node, sever the link there, and continue from the saved successor. Slots past the list's end stay `nullptr`. The count plus the single splitting pass are both O(n + k), with O(k) space for the output array.
