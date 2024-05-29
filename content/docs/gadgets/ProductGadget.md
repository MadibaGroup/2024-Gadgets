**Array Level**

- Prover has A = {a, b, c, d, e} and wants to show the $\prod A= z$
- Prover computes the accumulator B = {abcde, bcde, cde, de, e}
- To prove $\prod A= z$, it suffices to show that (i) $abcde = z$ and (ii) $B_i = A_iB_{i+1} \forall i$​​
- Note that (ii) doesn't hold in general for $i = n -1$, the last entry of A. We can deal with the simpler special case $z = 1$, and separately for $z \neq 1$​ 
- For $z = 1$, (ii) holds for $n - 1$, assuming the indices wrap around (this works if we use an nth root of unity, where n is the length of the array)
- For $z \neq 1$, the edge case will be fixed with an extra condition when we define our polynomial 

**Polynomial Level**

- The prover interpolates A and B to get $P_A(x)$ and $P_B(x)$, with the y values as the entries in the array and the x values as powers of an nth root of unity. For example $P_A(X)$ passes through ($\omega^0$, a), ($\omega^1$, b), ($\omega^2$, c), ($\omega^3$, d), ($\omega^4$, e)
- The prover then commits to these two polynomials (e.g. with KZG)
- The prover also commits to $P_{B'}(x):= P_{B}(x \cdot \omega)$. In other words, $P_B'(x)$ is the interpolation of the array $B'$ such that $B'_i = B_{i + 1}$, except $B'_{n-1}$, which is equal to $B_0$
- To prove $abcde = z$ the prover simply opens the commitment to $P_B(x)$ at $x = \omega^0 = 1$​ 
- To prove $B_i = A_iB_{i-1} \forall i$ we define $Q(X) = P_A(x)P_{B'}(x) - P_B(x)$ and show it vanishes for $1 \leq x \leq n - 1$​ (this is a separate gadget)
- For $z \neq 1$ we fix the edge case by multiplying $Q(x)$ by $(x - \omega^{n-1})$, making $\omega^{n-1}$ a root. Then $Q(x)$ vanished at $\omega^{n-1}$, even though  $B_i = A_iB_{i-1} \forall i$ doesn't hold for $i = n - 1$. In this case, $Q(X) = (P_A(x)P_{B'}(x) - P_B(x))(x - \omega^{n-1})$

**Verifier Level**

- verify $P_A(x)$ and $P_B(x)$ are correct interpolations of A and B (checking $Q(x)$​ vanishes is checking they are defined properly in terms of each other - do we have to verify that they accurately correspond to the array in any way?)

- verify $P_{B'}(x)$ is correct, i.e. $P_{B'}(x) = P_{B}(x \omega)$. To do so, the verifier sends a random challenge value $u$. The prover then sends $P_{B'}(u), P_{B}(u\omega)$ and $u\omega$ as well as a proof $\pi$ that these openings are correct. If the proof $\pi$ is correct and $P_B'(u) = P_B(u\omega)$, then with high probability we can say that $P_{B'}(x) = P_B(x\omega)$. By the Fiat-Shamir, this can also be done non-interactively.

-  verify opening of $P_A(x)$​ at $x = \omega^0 = 1$ (KZG let's us do this)

- verify $Q(x)$ is defined properly; this can be done using the commitments to $P_A(x)$, $P_B(x)$, and $P_{B'}(x)$ which the verifier has, and a pairing operation. The verifier wishes to check the following:

  ​	$comm(Q(x)) = comm(P_A(x)) \cdot P_{B'}(x) - P_B(x))$​​

  By the homomorphic property of the commitment, we could rewrite the equation as the following:

  ​	$comm(Q(x)) = comm(P_A(x)) \cdot P_{B'}(x)) - comm(P_B(x))$​

  But then we get stuck trying to deal with the mulitplication. Instead, the verifier must use a pairing operation to check the equality. First, we will rewrite our equation in terms of the group element used for the commitment:

  ​	$g^{Q(\tau)} = g^{P_A(\tau) \cdot P_{B'}(\tau) - P_B(\tau)}$​

  Now, we note the above equation holds if and only if the following equation (using pairings) holds:

  ​	$e(g,g)^{Q(\tau)}= e(g, g)^{P_A(\tau)\cdot P_{B'}(\tau) - P_B(\tau)}$​

  Which is equivalent to:

  ​	$e(g, g^{Q(\tau)}) = e(g^{P_A(\tau)}, g^{P_{B'}(\tau)}) \cdot e(g, (g^{P_B(\tau)})^{-1})$​​

  Thus this equation, which the verifier can calculate, is used to verify that $Q(x)$ is the correct polynomial.

- need to verify $Q(x)$ vanished on domain (this is a separate gadget) 

