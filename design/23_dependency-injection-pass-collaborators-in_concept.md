## fact: Dependency injection — pass collaborators in
tags: di, testability, constructor-injection
track: design

**Dependency injection** means a class receives its collaborators (through its constructor) instead of constructing or looking them up internally. `new`-ing a `StripeGateway` inside `OrderService`, or calling `Repository::instance()`, welds the class to one concrete implementation and to global state — untestable and unconfigurable.

Constructor injection makes every dependency visible in the signature (you can't forget one) and swappable: production passes real objects, tests pass fakes. No mocking framework required.

```cpp
class OrderService {
    PaymentGateway& gateway_;      // injected — not created inside, not a singleton
    Repository&     repo_;
public:
    OrderService(PaymentGateway& g, Repository& r) : gateway_(g), repo_(r) {}
    void checkout(const Cart& c) { gateway_.charge(c.total()); repo_.save(c); }
};
// Production wires the real ones; a test passes FakePaymentGateway + InMemoryRepository.
```
