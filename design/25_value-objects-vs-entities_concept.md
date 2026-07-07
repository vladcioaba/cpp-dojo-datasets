## fact: Value objects vs entities
tags: architecture, ddd, value-object, entity, immutability
track: design

Two kinds of domain type. An **entity** has an identity that persists through change: a `User` with id 42 is the same user after a name change; equality is by id. A **value object** is defined *entirely* by its attributes: `Money{500,"USD"}` equals any other `Money{500,"USD"}`; equality is by value — and it should be **immutable**, so operations return new values instead of mutating.

Getting this distinction right removes a class of bugs: value objects are safely shared and copied; entities are the things you track, store, and reference by id.

```cpp
// Value object: equal-by-value, immutable, no identity.
struct Money {
    std::int64_t cents;
    std::string  currency;
    friend bool operator==(const Money&, const Money&) = default;   // value equality
    Money added(const Money& o) const { return {cents + o.cents, currency}; }   // returns a new value
};

// Entity: identity by id; two Users named "Ada" are still different users.
class User { UserId id_; std::string name_; /* equal iff id_ == other.id_ */ };
```
