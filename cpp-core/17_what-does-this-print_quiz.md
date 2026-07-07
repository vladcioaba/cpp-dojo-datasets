## quiz: What does this print?
tags: core, slicing

```cpp
struct Animal { virtual std::string speak() const { return "..."; } };
struct Dog : Animal { std::string speak() const override { return "woof"; } };

void greet(Animal a) { std::cout << a.speak(); }
int main() { Dog d; greet(d); }
```

- [ ] woof
- [x] ...
- [ ] Compile error
- [ ] Undefined behavior

> `greet` takes `Animal` **by value**: the `Dog` is sliced — only the `Animal` subobject is copied, and the dynamic type of `a` is exactly `Animal`. Virtual dispatch has nothing to dispatch to. Pass polymorphic types by reference or pointer.
