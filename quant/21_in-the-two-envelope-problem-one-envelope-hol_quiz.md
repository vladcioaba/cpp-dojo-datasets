## quiz: In the two-envelope problem, one envelope holds twice the other. After picking one, the "always switch" argument computes ½(2x)+½(x/2)=1.25x. Why is it flawed?
tags: paradox, expected-value, conditional-probability
track: quant

- [ ] Switching genuinely earns a 25% expected gain
- [x] It reuses one symbol x for two different amounts, so the two branches aren't a valid expectation; by symmetry switching gains nothing
- [ ] You should therefore never switch
- [ ] The envelopes must contain equal amounts

> The calculation lets "x" mean your envelope's amount, but in the branch where you hold the larger sum, the other envelope is x/2, and in the branch where you hold the smaller, it is 2x — different underlying totals. With no proper prior over the amounts, the branches can't be averaged that way, and by symmetry either envelope is equally good: expected gain from switching is 0.
