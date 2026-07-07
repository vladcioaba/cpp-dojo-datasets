## quiz: A fair coin is flipped until the first head. If the first head is on flip n, you are paid $2^n. What is the expected payout?
tags: st-petersburg, expected-value, variance
track: quant

- [x] Infinite (the sum diverges)
- [ ] $2
- [ ] $4
- [ ] Undefined / no answer

> The first head lands on flip n with probability (1/2)^n, paying $2^n. Expected payout = Σₙ (1/2)^n · 2^n = Σₙ 1 = 1 + 1 + 1 + … = ∞. The St. Petersburg paradox: EV is infinite yet nobody pays a large finite entry fee, because the variance is enormous and huge payoffs are astronomically rare.
