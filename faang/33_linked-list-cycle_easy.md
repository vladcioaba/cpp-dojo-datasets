## challenge: Linked List Cycle
tags: linked-list, two-pointers, hash-table
track: faang
difficulty: easy

Given the `head` of a singly linked list, determine whether the list contains a cycle. A cycle exists if some node can be reached again by continuously following the `next` pointer. Return `true` if there is a cycle and `false` otherwise, using O(1) extra space.

Constraints: the list has `0 <= n <= 10^4` nodes, `-10^5 <= Node.val <= 10^5`.

Example: `3 -> 2 -> 0 -> -4` with the tail pointing back to the node at index `1` → `true`. Example: `1 -> 2 -> 3` with no back-link → `false`.

hint: If a runner moving two steps ever meets a runner moving one step, they must be circling the same loop.
hint: Use Floyd's tortoise-and-hare: a slow pointer that advances one node and a fast pointer that advances two.
hint: Stop when the fast pointer (or its `next`) falls off the end — that means no cycle; if slow ever equals fast, there is one.

```cpp
// starter
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};
bool hasCycle(ListNode* head);
```

```cpp
bool hasCycle(ListNode* head) {
    ListNode* slow = head;
    ListNode* fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) return true;
    }
    return false;
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
static ListNode* buildCycle(const vector<int>& v, int pos) {
    vector<ListNode*> nodes;
    ListNode dummy(0); ListNode* t = &dummy;
    for (int x : v) { t->next = new ListNode(x); t = t->next; nodes.push_back(t); }
    if (pos >= 0 && !nodes.empty() && pos < (int)nodes.size())
        nodes.back()->next = nodes[pos];
    return dummy.next;
}
//__USER__
int main() {
    if (!hasCycle(buildCycle({3,2,0,-4}, 1))) { std::puts("case1"); return 1; }  // cycle present
    if ( hasCycle(buildCycle({1,2,3,4,5}, -1))) { std::puts("case2"); return 1; } // no cycle
    if ( hasCycle(buildCycle({}, -1)))          { std::puts("case3"); return 1; } // empty list
    if ( hasCycle(buildCycle({1}, -1)))         { std::puts("case4"); return 1; } // single node, no cycle
    if (!hasCycle(buildCycle({1}, 0)))          { std::puts("case5"); return 1; } // single node self-cycle
    std::puts("PASS");
}
```

**Editorial:** Advance a slow pointer one node and a fast pointer two nodes per step. In a cyclic list the fast pointer laps the slow one and they collide; in an acyclic list the fast pointer reaches the end first. This uses O(n) time and O(1) extra space, avoiding the hash-set alternative that stores every visited node.
