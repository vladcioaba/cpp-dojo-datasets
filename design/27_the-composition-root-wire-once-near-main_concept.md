## fact: The composition root — wire once, near main
tags: architecture, composition-root, di, testability
track: design

A **composition root** is the single place — as close to `main()` as possible — where the concrete object graph is constructed and wired together. Everywhere else takes its dependencies as parameters. Concentrating all the `new`/`make_unique` calls here means only one file knows the concrete types; the rest of the system is pure policy against interfaces.

This is the alternative to singletons. A `Service::instance()` scattered through the code hides its dependencies and shares mutable state across tests (order-dependent failures). Explicit wiring at the root is verbose once, and testable everywhere.

```cpp
int main() {                       // the composition root
    SystemClock             clock;
    PostgresOrderRepository repo{connect()};
    StripeGateway           gateway{api_key};
    OrderService            service{gateway, repo};   // graph assembled in one place
    serve(service);
}
// Contrast: OrderService calling Repository::instance() — invisible coupling, untestable.
```
