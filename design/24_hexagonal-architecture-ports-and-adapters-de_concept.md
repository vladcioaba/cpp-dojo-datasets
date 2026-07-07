## fact: Hexagonal architecture — ports and adapters, depend inward
tags: architecture, hexagonal, ports-adapters, layering
track: design

**Hexagonal (ports & adapters)** architecture keeps the domain at the center. The domain declares *ports* — interfaces for the things it needs (persistence, notifications) — and infrastructure supplies *adapters* that implement them. The crucial rule: **all dependencies point inward.** The domain never `#include`s a database driver or an HTTP header; the database depends on the domain's port, not the reverse.

Swap Postgres for an in-memory store by providing a different adapter — the domain doesn't know or care, which is what makes it unit-testable without infrastructure.

```cpp
// Port — declared and owned by the domain:
struct OrderRepository {
    virtual void save(const Order&) = 0;
    virtual std::optional<Order> find(OrderId) = 0;
    virtual ~OrderRepository() = default;
};
// Adapter — lives in infrastructure, depends inward on the port:
class PostgresOrderRepository : public OrderRepository { /* SQL, driver includes live here */ };
// Domain services take OrderRepository&, never the concrete adapter.
```
