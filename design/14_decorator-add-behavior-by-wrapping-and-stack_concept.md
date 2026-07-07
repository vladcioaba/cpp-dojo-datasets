## fact: Decorator — add behavior by wrapping, and stack it
tags: patterns, structural, decorator
track: design

A **Decorator** wraps an object behind the *same* interface and adds behavior before/after delegating inward. Unlike subclassing, decorators **compose at runtime and stack** — `Encrypting(Compressing(File))` layers two independent responsibilities without a `CompressingEncryptingFile` class. It is the pattern behind stream/filter chains.

Each layer holds the wrapped object by `unique_ptr` to the interface, so any decorator can wrap any other in any order.

```cpp
struct DataSource { virtual std::string read() = 0; virtual ~DataSource() = default; };

class CompressingSource : public DataSource {
    std::unique_ptr<DataSource> inner_;
public:
    explicit CompressingSource(std::unique_ptr<DataSource> s) : inner_(std::move(s)) {}
    std::string read() override { return compress(inner_->read()); }   // decorate, then delegate
};
// An EncryptingSource wraps identically -> stack: encrypt(compress(file)).
```
