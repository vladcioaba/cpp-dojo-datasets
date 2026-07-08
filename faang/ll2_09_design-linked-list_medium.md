## challenge: Design Linked List
tags: linked-list, design
track: faang
difficulty: medium

Design your own singly linked list. Implement the `MyLinkedList` class with 0-indexed access: `get(index)` returns the value at `index` or `-1` if it's out of range; `addAtHead(val)` inserts a node before the first element; `addAtTail(val)` appends a node; `addAtIndex(index, val)` inserts a node before the `index`-th node — if `index` equals the length the node is appended, if `index` is greater than the length nothing happens; `deleteAtIndex(index)` removes the `index`-th node if it is valid.

Constraints: `0 <= index, val <= 1000`; at most `2000` calls are made across `get`, `addAtHead`, `addAtTail`, `addAtIndex`, and `deleteAtIndex`.

Example: `addAtHead(1)`, `addAtTail(3)`, `addAtIndex(1, 2)` builds `1 -> 2 -> 3`; `get(1)` → `2`; `deleteAtIndex(1)` leaves `1 -> 3`; `get(1)` → `3`.

hint: Track the current length so `addAtTail` and the bounds checks in `get`/`addAtIndex`/`deleteAtIndex` are O(1) decisions.
hint: Route `addAtHead` and `addAtTail` through `addAtIndex` (with index `0` and index `size`) to avoid duplicating the insertion logic; to insert or delete at position `i` you first walk to node `i - 1`.

```cpp
// starter
class MyLinkedList {
public:
    MyLinkedList();
    int get(int index);
    void addAtHead(int val);
    void addAtTail(int val);
    void addAtIndex(int index, int val);
    void deleteAtIndex(int index);
};
```

```cpp
class MyLinkedList {
    struct Node {
        int val;
        Node* next;
        Node(int v) : val(v), next(nullptr) {}
    };
    Node* head;
    int sz;
public:
    MyLinkedList() : head(nullptr), sz(0) {}

    int get(int index) {
        if (index < 0 || index >= sz) return -1;
        Node* cur = head;
        for (int i = 0; i < index; ++i) cur = cur->next;
        return cur->val;
    }

    void addAtHead(int val) { addAtIndex(0, val); }

    void addAtTail(int val) { addAtIndex(sz, val); }

    void addAtIndex(int index, int val) {
        if (index > sz) return;
        if (index < 0) index = 0;
        Node* node = new Node(val);
        if (index == 0) {
            node->next = head;
            head = node;
        } else {
            Node* prev = head;
            for (int i = 0; i < index - 1; ++i) prev = prev->next;
            node->next = prev->next;
            prev->next = node;
        }
        ++sz;
    }

    void deleteAtIndex(int index) {
        if (index < 0 || index >= sz) return;
        if (index == 0) {
            Node* old = head;
            head = head->next;
            delete old;
        } else {
            Node* prev = head;
            for (int i = 0; i < index - 1; ++i) prev = prev->next;
            Node* old = prev->next;
            prev->next = old->next;
            delete old;
        }
        --sz;
    }
};
```

```cpp
// harness
#include <cstdio>
#include <vector>
using std::vector;
//__USER__
int main() {
    { MyLinkedList l;
      l.addAtHead(1); l.addAtTail(3); l.addAtIndex(1, 2);   // 1 -> 2 -> 3
      if (l.get(1) != 2) { std::puts("case1"); return 1; }
      l.deleteAtIndex(1);                                    // 1 -> 3
      if (l.get(1) != 3) { std::puts("case2"); return 1; }
      if (l.get(3) != -1) { std::puts("case3"); return 1; } }
    { MyLinkedList l;
      l.addAtHead(7); l.addAtHead(2); l.addAtHead(1);        // 1 -> 2 -> 7
      l.addAtIndex(3, 0);                                    // 1 -> 2 -> 7 -> 0
      l.deleteAtIndex(2);                                    // 1 -> 2 -> 0
      if (l.get(0) != 1) { std::puts("case4"); return 1; }
      if (l.get(1) != 2) { std::puts("case5"); return 1; }
      if (l.get(2) != 0) { std::puts("case6"); return 1; }
      l.addAtHead(6);                                        // 6 -> 1 -> 2 -> 0
      l.addAtTail(4);                                        // 6 -> 1 -> 2 -> 0 -> 4
      if (l.get(0) != 6) { std::puts("case7"); return 1; }
      l.addAtIndex(5, 5);                                    // 6 -> 1 -> 2 -> 0 -> 4 -> 5
      if (l.get(5) != 5) { std::puts("case8"); return 1; }
      l.addAtIndex(7, 9);                                    // out of range, ignored
      if (l.get(6) != -1) { std::puts("case9"); return 1; } }
    { MyLinkedList l;
      if (l.get(0) != -1) { std::puts("case10"); return 1; }
      l.deleteAtIndex(0);                                    // no-op on empty list
      l.addAtTail(10);
      if (l.get(0) != 10) { std::puts("case11"); return 1; } }
    std::puts("PASS");
}
```

**Editorial:** Store a head pointer and a running size. Keeping `size` current turns every bounds check and `addAtTail` into O(1) logic, while the actual work of insertion and deletion is funneled through positional helpers that walk to node `index - 1` and re-link its `next`. Insertion at index `0` and deletion of the head are handled as special cases since there is no predecessor. Each `get`, `addAtIndex`, and `deleteAtIndex` is O(index) time; the structure uses O(n) space overall.
