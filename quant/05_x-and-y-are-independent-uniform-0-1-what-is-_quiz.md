## quiz: X and Y are independent Uniform(0,1). What is E[max(X, Y)]?
tags: order-statistics, continuous, expected-value
track: quant

- [ ] 1/2
- [x] 2/3
- [ ] 3/4
- [ ] 1/3

> Let M = max(X,Y). Then P(M ≤ x) = P(X≤x)P(Y≤x) = x², so the density is 2x on (0,1). E[M] = ∫₀¹ x·2x dx = ∫₀¹ 2x² dx = 2/3. In general E[max of n iid U(0,1)] = n/(n+1); here n=2 gives 2/3.
