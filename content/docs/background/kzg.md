# Polynomial Commitment Schemes (PCS)

Say that Carol sends a polynomial to Alice and the same polynomial to Bob. Later Alice and Bob are sitting together and want to make sure their polynomials, which they got from Carol, are actually the same. How do you compare polynomials?

There are many sufficient comparisons, but here are two ways to get started:

1. Ensure the degree $d$ is the same and each coefficient is the same
2. Ensure the degree $d$ is the same and check at $d+1$ points





## KZG

**Definition 1 (Polynomial Commitment Scheme).** A polynomial commitment scheme (PCS) is an interactive proof system that enables $\mathcal{P}$ to convince $\mathcal{V}$ that he knows a polynomial, without revealing the polynomial directly. $\mathcal{P}$ and $\mathcal{V}$ run the protocol in three moves: **gen**, **com**, and **open**. [Plonk]

