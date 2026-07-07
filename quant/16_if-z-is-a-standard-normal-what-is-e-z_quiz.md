## quiz: If Z is a standard normal, what is E[|Z|]?
tags: continuous, normal, expected-value
track: quant

- [x] √(2/π) ≈ 0.80
- [ ] 0
- [ ] 1
- [ ] 2/π ≈ 0.64

> E[|Z|] = 2∫₀^∞ z·φ(z) dz where φ(z) = e^(−z²/2)/√(2π). The integral of z·e^(−z²/2) is 1, so E[|Z|] = 2/√(2π) = √(2/π) ≈ 0.7979. E[Z] = 0 by symmetry, but the absolute value has positive mean; √(Var) = 1 is a different quantity.
