## quiz: A disease affects 1% of people. A test is 99% sensitive (positive if diseased) and has a 5% false-positive rate. Given a positive test, what is the probability of disease?
tags: bayes, conditional-probability
track: quant

- [ ] 99%
- [x] About 17%
- [ ] 95%
- [ ] 50%

> Bayes: P(D|+) = P(+|D)P(D) / [P(+|D)P(D) + P(+|¬D)P(¬D)] = (0.99·0.01) / (0.99·0.01 + 0.05·0.99) = 0.0099 / (0.0099 + 0.0495) = 0.0099/0.0594 = 1/6 ≈ 16.7%. The rare base rate means most positives are false positives.
