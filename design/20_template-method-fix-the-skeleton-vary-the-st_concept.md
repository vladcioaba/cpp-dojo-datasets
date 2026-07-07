## fact: Template Method — fix the skeleton, vary the steps (NVI)
tags: patterns, behavioral, template-method, nvi
track: design

The **Template Method** puts the invariant *shape* of an algorithm in one non-virtual base method and lets subclasses override individual *steps*. The order and the surrounding bookkeeping live in exactly one place; subclasses can't accidentally reorder them.

Pairing it with the **Non-Virtual Interface** idiom — public method non-virtual, the customizable steps `private virtual` — lets the base wrap every call with pre/post logic while still allowing overrides. (This is the OOP counterpart to `std::sort` taking a comparator.)

```cpp
class DataExporter {
public:
    void run() {                                   // the template method — fixed skeleton
        open();
        for (auto& row : fetch()) write(format(row));
        close();
    }
    virtual ~DataExporter() = default;
private:
    virtual std::vector<Row> fetch() = 0;          // steps subclasses customize
    virtual std::string format(const Row&) = 0;
    void open(); void close(); void write(std::string_view);   // shared, not overridable
};
```
