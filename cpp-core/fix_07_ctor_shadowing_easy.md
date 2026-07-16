## challenge: fix: every rectangle has zero area
tags: code-review, debugging, classes
track: core
difficulty: easy

This code review found a bug: no matter what dimensions a Rectangle is constructed with, `area()` always returns 0. Find and fix it — keep the function signature.

hint: The constructor body runs without errors — but what exactly does each assignment assign to?
hint: This is name shadowing: parameters hiding members.
hint: Inside the constructor body, `width` and `height` name the parameters, so both statements assign a parameter to itself and the members keep their default 0.

```cpp
// starter
class Rectangle {
public:
    Rectangle(int width, int height) {
        width = width;
        height = height;
    }
    int area() const { return width * height; }
private:
    int width = 0;
    int height = 0;
};
```

```cpp
class Rectangle {
public:
    Rectangle(int width, int height)
        : width(width), height(height) {}
    int area() const { return width * height; }
private:
    int width = 0;
    int height = 0;
};
```

```cpp
// harness
#include <bits/stdc++.h>
//__USER__
int main() {
    Rectangle r(3, 4);
    assert(r.area() == 12);
    Rectangle q(7, 2);
    assert(q.area() == 14);
    std::puts("PASS");
}
```

**Editorial:** The constructor parameters `width` and `height` shadow the data members of the same name, so `width = width;` assigns the parameter to itself and the members are never written — they keep their default initializers of 0. A member-initializer list fixes it, because in `: width(width)` the name *outside* the parentheses is looked up as a member while the name inside is the parameter; `this->width = width;` in the body works too. Reviewers catch this by being wary of constructor bodies that assign same-named identifiers — and most compilers will point at it with `-Wshadow` or self-assign warnings, which is a good reason to build with warnings on.
