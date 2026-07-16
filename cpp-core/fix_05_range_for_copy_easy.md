## challenge: fix: the discount that never sticks
tags: code-review, debugging, references
track: core
difficulty: easy

This code review found a bug: applying a discount to the cart runs without errors, but every price in the cart is unchanged afterwards. Find and fix it — keep the function signature.

hint: The math is right — look at the range-for declaration.
hint: The loop mutates something, but not the elements in the vector.
hint: `for (auto item : items)` copies each Item; the discount is applied to the copy, which is thrown away at the end of each iteration.

```cpp
// starter
struct Item {
    std::string name;
    double price;
};

void applyDiscount(std::vector<Item>& items, double rate) {
    for (auto item : items) {
        item.price -= item.price * rate;
    }
}
```

```cpp
struct Item {
    std::string name;
    double price;
};

void applyDiscount(std::vector<Item>& items, double rate) {
    for (auto& item : items) {
        item.price -= item.price * rate;
    }
}
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    std::vector<Item> cart{{"keyboard", 100.0}, {"mouse", 40.0}};
    applyDiscount(cart, 0.5);
    assert(cart[0].price == 50.0);
    assert(cart[1].price == 20.0);
    assert(cart[0].name == "keyboard");
    std::puts("PASS");
}
```

**Editorial:** `for (auto item : items)` declares `item` as a *copy* of each element, so the discount mutates a temporary that dies at the end of the iteration — the vector itself is untouched (and each iteration also pays for copying a `std::string`). Change the loop variable to `auto&` to mutate in place (`const auto&` is the right default when only reading). Reviewers spot this by checking every range-for that writes to the loop variable: if the intent is mutation, the declaration must be a reference.
