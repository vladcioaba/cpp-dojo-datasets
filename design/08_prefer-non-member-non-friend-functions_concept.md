## fact: Prefer non-member non-friend functions
tags: encapsulation, interfaces, meyers
track: design

Scott Meyers' guideline: if a function can be implemented using only a class's public interface, make it a **non-member non-friend** free function. Counterintuitively this *increases* encapsulation — the number of functions that can touch private state is a measure of how exposed those internals are, and a free function touches none.

It also improves packaging: free functions can live in separate headers, be grouped by client, and be added without editing the class. Keep as members only what needs private access or is fundamental to the type's identity.

```cpp
class Text {
    std::string s_;
public:
    void append(std::string_view x);
    std::size_t size() const;
    char at(std::size_t i) const;
};

// Needs no private access -> free function, not a member. Smaller blast radius.
std::size_t word_count(const Text& t);
```
