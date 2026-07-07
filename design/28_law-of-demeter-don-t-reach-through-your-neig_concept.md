## fact: Law of Demeter — don't reach through your neighbors
tags: architecture, coupling, law-of-demeter, cohesion
track: design

The **Law of Demeter** ("don't talk to strangers"): a method should only call methods on `this`, its parameters, objects it creates, and its direct members — not on objects returned by those. A "train wreck" like `a.b().c().d()` couples the caller to the *entire* chain of intermediate types; change any of their internals and this line breaks.

The fix is **tell, don't ask**: push the behavior to the object that owns the data. Low coupling and high cohesion are the goal — each unit hides its internals and exposes intent.

```cpp
// Smell: reaching through the object graph (couples you to Customer, Wallet, Balance):
order.customer().wallet().balance().deduct(amount);

// Fix: tell the object what you want; it coordinates its own internals:
order.pay(amount);
```
