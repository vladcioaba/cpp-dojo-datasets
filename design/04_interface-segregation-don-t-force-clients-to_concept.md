## fact: Interface Segregation — don't force clients to depend on unused methods
tags: solid, isp, interfaces
track: design

The **Interface Segregation Principle**: many small, role-specific interfaces beat one fat one. A read-only client forced to depend on a `Device` that also declares `write()` and `reset()` recompiles when those unrelated methods change, and every mock it writes must stub methods it never calls.

Split the fat interface into cohesive roles. A type can implement several; a client depends only on the slice it uses.

```cpp
// Fat: a read-only consumer still drags in write() and reset().
struct Device {
    virtual std::string read() = 0;
    virtual void write(std::string_view) = 0;
    virtual void reset() = 0;
    virtual ~Device() = default;
};

// Segregated roles — depend on exactly what you call.
struct Readable { virtual std::string read() = 0;            virtual ~Readable() = default; };
struct Writable { virtual void write(std::string_view) = 0;  virtual ~Writable() = default; };
```
