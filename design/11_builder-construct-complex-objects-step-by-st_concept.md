## fact: Builder — construct complex objects step by step
tags: patterns, creational, builder, fluent
track: design

When an object has many fields — several optional, some order-independent — a telescoping constructor `Thing(a, b, c, d, e)` becomes unreadable and error-prone. The **Builder** collects settings incrementally and produces the finished object with `build()`. The **fluent** form returns `*this` from each setter so calls chain into a readable sentence.

Because the object is only assembled at `build()`, the builder can validate the combination and construct an immutable result in one shot.

```cpp
enum class Method { Get, Post };
struct HttpRequest { std::string url; std::map<std::string,std::string> headers; Method method = Method::Get; };

class RequestBuilder {
    HttpRequest r_;
public:
    RequestBuilder& url(std::string u)                  { r_.url = std::move(u); return *this; }
    RequestBuilder& header(std::string k, std::string v){ r_.headers.emplace(std::move(k), std::move(v)); return *this; }
    RequestBuilder& method(Method m)                    { r_.method = m; return *this; }
    HttpRequest build() { return std::move(r_); }
};
auto req = RequestBuilder{}.url("/api").method(Method::Post).build();
```
