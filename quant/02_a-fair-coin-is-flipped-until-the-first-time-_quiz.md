## quiz: A fair coin is flipped until the first time you see two heads in a row (HH). What is the expected number of flips?
tags: expected-value, coin, markov
track: quant

- [ ] 4
- [x] 6
- [ ] 8
- [ ] 3

> Let a = expected flips from scratch, b = expected flips after just seeing one H. a = 1 + ½b + ½a and b = 1 + ½·0 + ½a. Substituting: b = 1 + a/2, so a = 1 + ½(1 + a/2) + a/2 = 3/2 + 3a/4, giving a/4 = 3/2 and a = 6. (The T-after-H resets you fully, which is what pushes it above the naive guess.)
