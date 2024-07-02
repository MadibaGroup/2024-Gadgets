# Concatenation

## Recap of types

| Type | Description | Recap | This |
| ---- | ----------- | ----- | ---- |
| concat | $\mathsf{Arr}_3=\mathsf{Arr}_1\cup\mathsf{Arr}_2$ | $\mathsf{Arr}_3$ is the concatenation of $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$ | ✅ |

## Relation

$$
\mathcal{R}_{\mathtt{mult1}} := \left\{ \begin{array}{l} (K_{\mathsf{Arr}_1},K_{\mathsf{Arr}_2},K_{\mathsf{Arr}_3}) \end{array} \middle | \begin{array}{l} \mathsf{Arr}_3=\mathsf{Arr}_1\cup\mathsf{Arr}_2, \\ \mathsf{Arr}_3[i]=\mathsf{Arr}_1[i],i\in[0,n_1), \\ \mathsf{Arr}_3[i+n_1]=\mathsf{Arr}_2[i],i\in[0,n_2), \\ \mathsf{Poly}_\mathsf{Arr_j}=\mathsf{FFT.Interp}(\omega,\mathsf{Arr_j}), 1\leq j \leq 3, \\ K_\mathsf{Arr_j}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_j}), 1\leq j \leq 3, \end{array} \right\}
$$

## Intuition

The prover ($\mathcal{P}$) holds three arrays $\mathsf{Arr}_1$, $\mathsf{Arr}_2$ and $\mathsf{Arr_3}$. He wants to convince the verifier ($\mathcal{V}$) $\mathsf{Arr}_3$ is the concatenation of $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$. Specifically, assume there are $n_1$, $n_2$, and $n_3$ elements in $\mathsf{Arr}_1$, $\mathsf{Arr}_2$, and $\mathsf{Arr}_3$ respectively, then the prover will prove: (i) $\mathsf{Arr}_3[i]=\mathsf{Arr}_1[i],i\in[0,n_1)$ (ii) $\mathsf{Arr}_3[i+n_1]=\mathsf{Arr}_2[i],i\in[0,n_2)$. It is intuitive to think about using the grand product check in the Plookup. However, the product check of Plookup holds if and only if (i) $\mathsf{Arr}_1$ is the subset of $\mathsf{Arr}_2$ (ii) $\mathsf{Arr}_3$ is the union set of $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$ (iii) $\mathsf{Arr}_3$ is sorted by $\mathsf{Arr}$. Thus, we cannot use the product check directly, but we can leverage the same fact in the product check of Plookup: the product of $\mathsf{Arr}_3$ equals the product of $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$ if and only if $\mathsf{Arr}_3$ is the union set of $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$ (permutation check). Additionally, we need to prove the concatenation relationship rather than the permutation. To solve this problem, we can break $\mathsf{Arr}_3$ to two sub arrays $\mathsf{Arr}_{3_l}$ and $\mathsf{Arr}_{3_h}$ as we do in Plookup, and prove the product of $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$ equals the product of $\mathsf{Arr}_{3_l}$ and $\mathsf{Arr}_{3_h}$.

The other approach is fill $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$ with zero to $n_1+n_2$, and construct a new vector $\mathsf{Arr}_2^\prime$ by rotating right $\mathsf{Arr}_2$ by $n_1$, so that $\mathsf{Arr}_3$ is the addition of $\mathsf{Arr}_1$ and $\mathsf{Arr}_2^\prime$. Instead of performing the product check in the previous approach, the prover proves three things: (i) $\mathsf{Arr}_2^\prime$ is rotated from $\mathsf{Arr}_2$ (ii) $\mathsf{Arr}_3=\mathsf{Arr}_1+\mathsf{Arr}_2^\prime$ (iii) $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$ are correctly fill with zero.

## Protocol Details

### Array Level

* $\mathcal{P}$ holds an array $\mathsf{Arr}_1=[a_{(1,0)},a_{(1,1),\dots,a_{(1,n_1-1)}}]$ of $n_1$ integers and fills it with zero to $n_1+n_2$ such that
    * $\mathsf{Arr}_1[i]=a_{(1,i)},i\in[0,n_1)$
    * $\mathsf{Arr}_1[i]=0,i\in[n_1,n_1+n_2)$
* $\mathcal{P}$ holds an array $\mathsf{Arr}_2=[a_{(2,0)},a_{(2,1),\dots,a_{(2,n_1-1)}}]$ of $n_2$ integers and fills it with zero to $n_1+n_2$ such that
    * $\mathsf{Arr}_2[i]=a_{(2,i)},i\in[0,n_2)$
    * $\mathsf{Arr}_2[i]=0,i\in[n_2,n_1+n_2)$
* $\mathcal{P}$ holds an array $\mathsf{Arr}_3$ of $n_1+n_2$ integers
* $\mathcal{P}$ computes or holds an arrays $\mathsf{Arr}_2^\prime$ of $n_1+n_2$ integers such that
    * $\mathsf{Arr}_2^\prime[i+n_1]=\mathsf{Arr}_2[i],i\in[0,n_1+n_2)$
* $\mathsf{Arr}_1$, $\mathsf{Arr}_2^\prime$, and $\mathsf{Arr}_3$ satisfy:
    * $\mathsf{Arr}_3[i]=\mathsf{Arr}_1[i]+\mathsf{Arr}_2^\prime[i],i\in[0,n_1+n_2)$

### Polynomial Level

We assume arrays $\mathsf{Arr}_1$, $\mathsf{Arr}_2$, and $\mathsf{Arr}_3$ are encoded as the y-coordinates into a univariant polynomial where the x-coordinates (called the domain $\mathcal{H}_\kappa$) are chosen as the multiplicative group of order $\kappa$ with generator $\omega\in\mathbb{G}_\kappa$ (see [Background](../background/poly-iop.md) for more). In short, $\omega^0$ is the first element and $\omega^{\kappa-1}$ is the last element of $\mathcal{H}_\kappa$. If $\kappa$ is larger than the length of the array, the array can be padded with elements of value 1 (which will not change the product).

Recall the constraints we want to prove: 

1. $\mathsf{Arr}_3[i]=\mathsf{Arr}_1[i]+\mathsf{Arr}_2^\prime[i],i\in[0,n_1+n_2)$
2. $\mathsf{Arr}_2^\prime[i+n_1]=\mathsf{Arr}_2[i],i\in[n_1,n_1+n_2)$
3. $\mathsf{Arr}_1[i]=0,i\in[n_1,n_1+n_2)$
4. $\mathsf{Arr}_2[i]=0,i\in[n_2,n_1+n_2)$

In polynomial form, the constraints are:

1. $\mathsf{Poly}_{\mathsf{Arr}_3}(X)=\mathsf{Poly}_{\mathsf{Arr}_1}(X)+\mathsf{Poly}_{\mathsf{Arr}_2^\prime}(X)$
2. $\mathsf{Poly}_{\mathsf{Arr}_2}(X)=\mathsf{Poly}_{\mathsf{Arr}_2^\prime}(X\omega^{n_1})$
3. For $X\in[n_1,n_1+n_2)$, $\mathsf{Poly}_{\mathsf{Arr}_1}(X)=0$
4. For $X\in[n_2,n_1+n_2)$, $\mathsf{Poly}_{\mathsf{Arr}_2}(X)=0$

Next we take care of the "for $X$" conditions by zeroing out the rest of the polynomial that is not zero. See the gadget <span style="border-style:dotted;border-width: 2px;"> [zero1](./zero1)</span> for more on why this works.

1. $\mathsf{Poly}_\mathsf{Vanish1}(X)=\mathsf{Poly}_{\mathsf{Arr}_3}(X)-\mathsf{Poly}_{\mathsf{Arr}_1}(X)-\mathsf{Poly}_{\mathsf{Arr}_2^\prime}(X)$
2. $\mathsf{Poly}_\mathsf{Vanish2}(X)=\mathsf{Poly}_{\mathsf{Arr}_2}(X)-\mathsf{Poly}_{\mathsf{Arr}_2^\prime}(X\omega^{n_1})$
3. $\mathsf{Poly}_\mathsf{Vanish3}(X)=\mathsf{Poly}_{\mathsf{Arr}_1}(X)\cdot\frac{X^\kappa-1}{\prod_{i=n_1}^{n_1+n_2-1}(X-\omega^i)}$
4. $\mathsf{Poly}_\mathsf{Vanish4}(X)=\mathsf{Poly}_{\mathsf{Arr}_2}(X)\cdot\frac{X^\kappa-1}{\prod_{i=n_2}^{n_1+n_2-1}(X-\omega^i)}$

The four equations are vanishing for every value of $X\in\mathcal{H}_\kappa$ (but not necessarily true outside of these values). To show this, we divide each polynomial by $X^\kappa-1$, which is a minimal vanishing polynomial for $\mathcal{H}_\kappa$ that does not require interpolation to create. If the quotient is polynomial (and not a rational function), then $\mathsf{Poly}_\mathsf{Vanish1}(X)$ and $\mathsf{Poly}_\mathsf{Vanish2}(X)$ must be vanishing on $\mathcal{H}_\kappa$ too. Specifically, the prover computes:

1. $Q_1(X)=\frac{\mathsf{Poly}_\mathsf{Vanish1}(X)}{X^\kappa-1}$
2. $Q_2(X)=\frac{\mathsf{Poly}_\mathsf{Vanish2}(X)}{X^\kappa-1}$
3. $Q_3(X)=\frac{\mathsf{Poly}_\mathsf{Vanish3}(X)}{X^\kappa-1}$
4. $Q_4(X)=\frac{\mathsf{Poly}_\mathsf{Vanish4}(X)}{X^\kappa-1}$

We can replace polynomials $Q_1(X)$, $Q_2(X)$, $Q_3(X)$, and $Q_4(X)$ with a single polynomial $Q(X)$. We can do this because all three constraints have the same format: $\mathsf{Poly}_\mathsf{Vanish_i}(X)=0$. The batching technique is to create a new polynomial with all two $\mathsf{Poly}_\mathsf{Vanish_i}(X)$ values as coefficients. If and (overwhelmingly) only if all three are vanishing, then so will the new polynomial. This polynomial will be evaluated at a random challenge point $\rho$ selected after the commitments to the earlier polynomials are fixed. 

$$
Q(X)=\frac{\mathsf{Poly}_\mathsf{Vanish1}(X)+\rho\cdot\mathsf{Poly}_\mathsf{Vanish2}(X)+\rho^2\cdot\mathsf{Poly}_\mathsf{Vanish3}(X)+\rho^3\cdot\mathsf{Poly}_\mathsf{Vanish4}(X)}{X^\kappa-1}
$$

By rearranging, we can get $\mathsf{Poly}_\mathsf{Zero}(X)$ as a true zero polynomial (zero at every value both in $\mathcal{H}_\kappa$ and outside of it):

$$
\mathsf{Poly}_\mathsf{Zero}(X)=\mathsf{Poly}_\mathsf{Vanish1}(X)+\rho\cdot\mathsf{Poly}_\mathsf{Vanish2}(X)+\rho^2\cdot\mathsf{Poly}_\mathsf{Vanish3}(X)+\rho^3\cdot\mathsf{Poly}_\mathsf{Vanish4}(X)-Q(X)\cdot(X^{\kappa-1}-1)=0
$$

Ultimately the range gadget will satisfy the following constraints at the Commitment Level:

1. Show $Q(X)$ exists (as a polynomial that evenly divides the divisor)
2. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is correctly constructed from $\mathsf{Poly}_{\mathsf{Arr}_1}(X)$, $\mathsf{Poly}_{\mathsf{Arr}_2}(X)$, and $\mathsf{Poly}_{\mathsf{Arr}_3}(X)$
3. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is the zero polynomial

### Commitment Level

The verifier will never see the arrays or polynomials themselves. They are undisclosed because they either (i) contain private data or (ii) are too large to examine and maintain a succinct proof system. Instead, the prover will use commitments.

The prover will write the following commitments to the transcript:

* $K_{\mathsf{Arr}_1}=\mathsf{KZG.Commit}(\mathsf{Poly}_{\mathsf{Arr}_1}(X))$
* $K_{\mathsf{Arr}_2}=\mathsf{KZG.Commit}(\mathsf{Poly}_{\mathsf{Arr}_2}(X))$
* $K_{\mathsf{Arr}_3}=\mathsf{KZG.Commit}(\mathsf{Poly}_{\mathsf{Arr}_3}(X))$
* $K_{\mathsf{Arr}_2^\prime}=\mathsf{KZG.Commit}(\mathsf{Poly}_{\mathsf{Arr}_2^\prime}(X))$

The prover will generate a random challenge evaluation point (using strong Fiat-Shamir) on the polynomial that is outside of $\mathcal{H}_\kappa$. Call this point $\rho$. It will be used by the prover to create polynomial $Q(X)$ (see above) and the prover will write to the transcript:

* $\rho$
* $K_Q=\mathsf{KZG.Commit}(Q(X))$

The prover will generate a second random challenge evaluation point (using strong Fiat-Shamir) on the polynomial that is outside of $\mathcal{H}_\kappa$. Call this point $\zeta$. The prover will write the point and opening proofs to the transcript:

* $\zeta$
* $\mathsf{Poly}_{\mathsf{Arr}_1}(\zeta)=\mathsf{KZG.Open}(K_{\mathsf{Arr}_1},\zeta)$
* $\mathsf{Poly}_{\mathsf{Arr}_2}(\zeta)=\mathsf{KZG.Open}(K_{\mathsf{Arr}_2},\zeta)$
* $\mathsf{Poly}_{\mathsf{Arr}_3}(\zeta)=\mathsf{KZG.Open}(K_{\mathsf{Arr}_3},\zeta)$
* $\mathsf{Poly}_{\mathsf{Arr}_2^\prime}(\zeta\omega^{n_1})=\mathsf{KZG.Open}(K_{\mathsf{Arr}_2^\prime},\zeta\omega^{n_1})$
* $Q(\zeta)=\mathsf{KZG.Open}(K_Q,\zeta)$

To check the proof, the verifier uses the transcript to construct the value $Y_\mathsf{Zero}$ as follows:

* $Y_\mathsf{Vanish1}=\mathsf{Poly}_{\mathsf{Arr}_3}(\zeta)-\mathsf{Poly}_{\mathsf{Arr}_1}(\zeta)-\mathsf{Poly}_{\mathsf{Arr}_2^\prime}(\zeta)$
* $Y_\mathsf{Vanish2}=\mathsf{Poly}_{\mathsf{Arr}_2}(\zeta)-\mathsf{Poly}_{\mathsf{Arr}_2^\prime}(\zeta\omega^{n_1})$ 
* $Y_\mathsf{Vanish3}=\mathsf{Poly}_{\mathsf{Arr}_1}(\zeta)\cdot\frac{\zeta^\kappa-1}{\prod_{i=n_1}^{n_1+n_2-1}(\zeta-\omega^i)}$ 
* $Y_\mathsf{Vanish4}=\mathsf{Poly}_{\mathsf{Arr}_2}(\zeta)\cdot\frac{\zeta^\kappa-1}{\prod_{i=n_2}^{n_1+n_2-1}(\zeta-\omega^i)}$ 
* $Y_\mathsf{Zero}=Y_\mathsf{Vanish1}+\rho\cdot{Y_\mathsf{Vanish2}}+\rho^2\cdot{Y_\mathsf{Vanish3}}+\rho^3\cdot{Y_\mathsf{Vanish4}}-Q(\zeta)\cdot(\zeta^n-1)$

Finally, if the constraint system is true, the following constraint will be true (and will be false otherwise with overwhelming probability, due to the Schwartz-Zippel lemma on $\rho$ and $\zeta$) :

* $Y_\mathsf{Zero}\overset{?}{=}0$

## Security Proof

### Completeness

Any honest prover can do the computations explained above and create an accepting proof.

### Soundness

We prove knowledge soundness in the Algebraic Group Model (AGM). To do so, we must prove that there exists an efficient extractor $\mathcal{E}$ such that for any algebraic adversary $\mathcal{A}$ the probability of $\mathcal{A}$ winning the following game is $\mathsf{negl}(\lambda)$.

1. Given $[g,g^\tau,g^{\tau^2},\dots,g^{\tau^{n-1}}]$, $\mathcal{A}$ outputs commitments to $\mathsf{Poly}_{\mathsf{Arr}_1}(X)$, $\mathsf{Poly}_{\mathsf{Arr}_2}(X)$, $\mathsf{Poly}_{\mathsf{Arr}_3}(X)$, $\mathsf{Poly}_{\mathsf{Arr}_2^\prime}(X)$, and $Q$
2. $\mathcal{E}$, given access to $\mathcal{A}$'s outputs from the previous step, outputs $\mathsf{Poly}_{\mathsf{Arr}_1}(X)$, $\mathsf{Poly}_{\mathsf{Arr}_2}(X)$, $\mathsf{Poly}_{\mathsf{Arr}_3}(X)$, $\mathsf{Poly}_{\mathsf{Arr}_2^\prime}(X)$, and $Q$
3. $\mathcal{A}$ plays the part of the prover in showing that $Y_\mathsf{Zero}$ is zero at a random challenge $\zeta$
4. $\mathcal{A}$ wins if
   * $\mathcal{V}$ accepts at the end of the protocol
   * $\mathsf{Arr}_3$ is not the concatenation of $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$

Our proof is as follows:

When $\mathsf{Arr}_3$ is not the concatenation of $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$, there are three cases: some elemnets in $\mathsf{Arr}_1$ do not appear in $\mathsf{Arr}_3$, or some elements in $\mathsf{Arr}_2$ do not appear in $\mathsf{Arr}_3$, or both of the previous situations. Since $\mathcal{A}$ has to win the KS game in the first and the second cases if $\mathcal{A}$ wants to win the game in the last case, it suffices to check any one of the first two situations. Now assume there are some elements in $\mathsf{Arr}_1$ do not exist in $\mathsf{Arr}_3$. Because the first $n_1$ elements in $\mathsf{Arr}_2^\prime$ are zero, we know that the equation $\mathsf{Arr}_3[i]=\mathsf{Arr}_1[i]+\mathsf{Arr}_2^\prime[i]$ does not hold for some $i\in[0,n_1)$. Thus, for $\mathsf{Poly}_\mathsf{Vanish1}$, $Q_1$ does not exist (it is a rational function, not a polynomial), which means the probability that $Y_\mathsf{Zero}$ is a zero polynomial is negligible.

### Zero-Knowledge

To prove the above protocol is zero-knowledge, we do so by constructing a probabilistic polynomial time simulator $\mathcal{S}$ which, for every (possibly malicious) verifier $\mathcal{V}$, can output a view of the execution of the protocol that is indistinguishable from the view produced by the real execution of the program.

The simulator $\mathcal{S}$ randomly generates $\mathsf{Arr}_1^*$ and $\mathsf{Arr}_2^*$ and computes $\mathsf{Arr}_3^*$, then follows the same steps a prover would prove the concatenation. $\mathcal{S}$ computes $\mathsf{Arr}_2^{\prime*}$ and interpolates $\mathsf{Poly}_{\mathsf{Arr}_1^*}$, $\mathsf{Poly}_{\mathsf{Arr}_2^*}$, $\mathsf{Poly}_{\mathsf{Arr}_3^*}$, and $\mathsf{Poly}_{\mathsf{Arr}_2^{\prime*}}$. It computes $Q^*(X)$ and finally outputs the commitments to each of these polynomials (and writes the commitments to the transcript). Then, it generates the random challenge $\zeta^*$ (once again this is by strong Fiat-Shamir). It creates opening proofs for $\mathsf{Poly}_{\mathsf{Arr}_1^*}(\zeta^*)$, $\mathsf{Poly}_{\mathsf{Arr}_2^*}(\zeta^*)$, $\mathsf{Poly}_{\mathsf{Arr}_3^*}(\zeta^*)$, and $\mathsf{Poly}_{\mathsf{Arr}_2^{\prime*}}(\zeta^*\omega^{n_1})$, and $Q^*(\zeta^*)$, and writes these to the transcript as well. Since $\mathcal{S}$ knows each of the above polynomials, it can honestly compute this step and the proof will be accepted by $\mathcal{V}$. The transcript it generates this way will be indistinguishable from a transcript generated from a real execution since $\mathsf{PolyCommit}_\mathsf{Ped}$ has the property of Indistinguishability of Commitments due to the randomization by $h^{\hat{\phi}(x)}$.
