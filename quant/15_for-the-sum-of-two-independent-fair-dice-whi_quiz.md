## quiz: For the sum of two independent fair dice, which pair (mean, variance) is correct?
tags: expected-value, variance, dice
track: quant

- [ ] E = 7, Var = 35/12
- [x] E = 7, Var = 35/6
- [ ] E = 7, Var = 5.5
- [ ] E = 3.5, Var = 35/12

> Each die has mean 3.5, so the sum has E = 7. One die's variance is E[X²] − E[X]² = 91/6 − 49/4 = (182−147)/12 = 35/12. Variances of independent variables add, so Var(sum) = 2·35/12 = 35/6 ≈ 5.83. The trap answer 35/12 forgets to double it.
