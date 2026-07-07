## quiz: You roll a fair six-sided die repeatedly. What is the expected number of rolls to see all six faces at least once?
tags: expected-value, coupon-collector, dice
track: quant

- [ ] 21
- [x] 14.7
- [ ] 6
- [ ] 12.25

> Coupon collector: after collecting k distinct faces, the wait for a new one is geometric with success (6−k)/6, so its expectation is 6/(6−k). Total = 6(1/6 + 1/5 + 1/4 + 1/3 + 1/2 + 1/1) = 6·(1+½+⅓+¼+⅕+⅙) = 6·2.45 = 14.7.
