# Range

## Recap of types

| Type  | Description               | Recap                                                        | This |
| ----- | ------------------------- | ------------------------------------------------------------ | ---- |
| [range](#) | $\mathsf{Arr}[i]\in[0,r]$ | Each element of array $\mathsf{Arr}$ is in the range $[0,r]$ | ✅    |

## Relation

$\mathcal{R}_{\mathtt{add1}} := \left\{ \begin{array}{l} (K_\mathsf{Arr}) \end{array} \middle | \begin{array}{l} 0\le{\mathsf{Arr}[i]}\le{r}, i\in[0,n), \\ \mathsf{Poly}_\mathsf{Arr}=\mathsf{FFT.Interp}(\omega,\mathsf{Arr}), \\ K_\mathsf{Arr}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr}), \end{array} \right\}$

## Intuition

To prove each element of array $\mathsf{Arr}$ is in the range $[0,r]$, one of the most intuitive ways is we create a vector containing the numbers from $0$ to $r$ and run the lookup argument for $\mathsf{Arr}$. Another approach is we prove each element is in $[0,r]$. Specifically, we decompose the target number to digits in some base $x$ and prove (i) the digits are valid and (ii) the number can be recovered from the digits and the base. The prover ($\mathcal{P}$) holds a number $\eta$ and a vector $\mathsf{T}$ of $k=\lceil\log_x{\eta}\rceil$ integers from $\mathbb{Z}_q$: $[a_0,a_1,a_2,\dots,a_{k-1}]$. Then $r=x^k$ and the prover shows $\eta \in [0, r]$. It will produce a succinct (logarithm base $x$ of $\eta$; for simplicity, we will use base $2$) proof that the vector $\mathsf{T}$ satisfies the following conditions: (i) the first value of $\mathsf{T}$ equals to $\eta$ (ii) the last value of $\mathsf{T}$ equals to one or zero (iii) any value minus two times the next value is equal to one or zero in $\mathsf{T}$. The prover will encode $\mathsf{Arr}$ and $\mathsf{T}$ into two polynomials: $\mathsf{Poly}_\mathsf{Arr}$ and $\mathsf{Poly}_\mathsf{T}$ (using [evaluation points](../../background/poly-iop) on the domain $\mathcal{H}_\kappa$). It will commit to each polynomial: $K_\mathsf{Arr}$ and $K_\mathsf{T}$. The verifier ($\mathcal{V}$) cannot check any of the $\mathsf{Arr}$, $\mathsf{T}$ or $\mathsf{Poly}_\mathsf{Arr}$, $\mathsf{Poly}_\mathsf{T}$ values directly. Instead the verifier only sees $K_\mathsf{Arr}$, and $K_\mathsf{T}$.

Consider a small numerical example where $\eta = 14$, working with $x=2$. Since $k=\lceil\log_2{\eta}\rceil = 4$, we will demonstrate that $\eta \in [0,r=2^k]$ by constructing $\mathsf{Arr}$ consisting of $k$ integers. First, we know $\mathsf{T}[0] = \eta = 14$:

- $\mathsf{T} = [14, \bot, \bot, \bot]$

Now, for $\mathsf{T}[1]$ we want an integer $a_1$ such that $14 - 2\cdot a_1$ equals 1 or 0. Since 14 is even, we are looking for $a_1$ such that $14 - 2\cdot a_1=0$, thus $a_1=7$:

- $\mathsf{T} = [14, 7, \bot, \bot]$

Now we find $a_2$ such that $7-2\cdot a_2$ equals 1 or 0. Since 7 is odd, we are looking for $a_2$ such that $7-2\cdot a_2=1$, thus $a_2 = 3$:

- $\mathsf{T} = [14, 7, 3, \bot]$

Finally, we find $a_3$ such that $3 - 2\cdot a_3$ equals 1 or 0. Again, since 3 is odd, we are looking for $a_3$ such that $3-2\cdot a_3=1$, thus $a_3= 1$:

- $\mathsf{T} = [14, 7, 3, 1]$

Now, $\mathsf{Arr}$ is $k$ elements long, and the last element of $\mathsf{T}$ is one or zero, so we have constructed the desired vector $\mathsf{T}$. Note that the last element of $\mathsf{T}$ will only be zero if $\eta \leq x^{k-1}$.

In order to prove $K_\mathsf{Arr}$ and $K_\mathsf{T}$ satisfy the above three conditions, one of the most straightforward methods is to use the additive homomorphic property of the KZG commitment scheme, i.e., prove $K_\mathsf{Arr}$ and $K_\mathsf{T}$ satisfy the conditions through the group addition. This method works if the target condition only involves additive operations. However, there are multiplications in the above conditions, so it is infeasible to calculate the product of two KZG commitments unless the t-SDH can be solved. Even if the verifier is given the powers of the KZG commitments, it is inefficient to perform scalar multiplications for two commitments (imagine the time complexity of multiplying two $d$-degree polynomials is $O(d^2)$), which implies this method is hard to be succinct.

The second method is more general and widely used. The basic idea is instead of proving the commitments satisfy the conditions, the prover reveals the evaluations of the polynomials at a random point sent by the verifier to prove the evaluations satisfy the conditions; at the same time, the prover proves the evaluations are valid through the binding property of KZG commitment. This method works because of the Schwartz-Zippel lemma, which tells us if the equation (of the polynomials) holds at a random evaluation point on the domain $\mathcal{H}_\kappa$, then it holds at any point on $\mathcal{H}_\kappa$ with high probability. The probability is $d/|\mathbb{F}|$, where $d$ is the number of roots of the equation and $\mathbb{F}$ is the field of the random evaluation point. This means as long as the field is big enough, the probability of failure is negligible. By rearranging the polynomials, the verifier can challenge any point on the group field rather than $\mathcal{H}_\kappa$, which makes the probability of failure tend to $0$.

## Protocol Details

### Array Level

* $\mathcal{P}$ holds a number $\eta\in\mathbb{Z}$
* $\mathcal{P}$ computes or holds an array $\mathsf{T}=[t_0,t_1,t_2,\dots,t_{k-1}]$ of $k$ (recall $k=\lceil\log_2{\eta}\rceil$) integers ($t_i\in\mathbb{Z}$) such that:
  * $\mathsf{T}[0]=\eta$
  * $\mathsf{T}[k-1]\in\{0,1\}$
  * $\mathsf{T}[i]-2\cdot\mathsf{T}[i+1]\in\{0,1\}$

### Polynomial Level

We assume the array $\mathsf{T}$ is encoded as the y-coordinates into a univariant polynomial where the x-coordinates (called the domain $\mathcal{H}_\kappa$) are chosen as the multiplicative group of order $\kappa$ with generator $\omega\in\mathbb{G}_\kappa$ (see [Background](../../background/poly-iop) for more). In short, $\omega^0$ is the first element and $\omega^{\kappa-1}$ is the last element of $\mathcal{H}_\kappa$. If $\kappa$ is larger than the length of the array, the array can be padded with elements of value 0 (which will not change the sum).

Recall the constraints we want to prove: 

1. $\mathsf{T}[0]=\eta$
2. $\mathsf{T}[k-1]\in\{0,1\}$
3. $\mathsf{T}[i]-2\cdot\mathsf{T}[i+1]\in\{0,1\}$

In polynomial form, the constraints are:

1. For $X=\omega^0$: $\mathsf{Poly}_\mathsf{T}(X)=\eta$
2. For $X=\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{T}(X)\in\{0,1\}$
3. For all $X=\mathcal{H}_\kappa\setminus{\omega^{\kappa-1}}$: $\mathsf{Poly}_\mathsf{T}(X)-2\cdot\mathsf{Poly}_\mathsf{T}(X\omega)\in\{0,1\}$

Note because the value of $\eta$ is a secret, $\mathcal{P}$ will not reveal $\eta$ to let $\mathcal{V}$ verify $\eta$ is the evaluation of $\mathsf{Poly}_\mathsf{T}(\omega^0)$. $\mathcal{P}$ will leverage the hiding property of KZG (Pedersen) commitment to prove the committed $\eta$ is the correct evaluation. Specifically, $\mathcal{P}$ claims the committed $\eta$ is the correct one and opens $\mathsf{Poly}_\mathsf{T}$ at $\omega^0$. If the committed $\eta$ satisfy the KZG verification, $\mathcal{V}$ can believe the first constraint is satisfied.

We take care of the "for $X$" conditions of constraints 2 and 3 by zeroing out the rest of the polynomial that is not zero. See the gadget <span style="border-style:dotted;border-width: 2px;"> [zero1](../zero1)</span> for more on why this works.

1. $\mathsf{Poly}_\mathsf{Vanish1}(X)=\mathsf{Poly}_\mathsf{T}(X)\cdot[\mathsf{Poly}_\mathsf{T}(X)-1]\cdot\frac{X^\kappa-1}{X-\omega^{\kappa-1}}=0$
2. $\mathsf{Poly}_\mathsf{Vanish2}(X)=[\mathsf{Poly}_\mathsf{T}(X)-2\cdot\mathsf{Poly}_\mathsf{T}(X\omega)]\cdot[\mathsf{Poly}_\mathsf{T}(X)-2\cdot\mathsf{Poly}_\mathsf{T}(X\omega)-1]\cdot(X-\omega^{\kappa-1})=0$

The two equations are vanishing for every value of $X\in\mathcal{H}_\kappa$ (but not necessarily true outside of these values). To show this, we divide each polynomial by $X^\kappa-1$, which is a minimal vanishing polynomial for $\mathcal{H}_\kappa$ that does not require interpolation to create. If the quotient is polynomial (and not a rational function), then $\mathsf{Poly}_\mathsf{Vanish1}(X)$ and $\mathsf{Poly}_\mathsf{Vanish2}(X)$ must be vanishing on $\mathcal{H}_\kappa$ too. Specifically, the prover computes:

1. $Q_1(X) = \frac{\mathsf{Poly}_\mathsf{Vanish1}(X)}{X^\kappa - 1}$
2. $Q_2(X) = \frac{\mathsf{Poly}_\mathsf{Vanish2}(X)}{X^\kappa - 1}$

We can replace polynomials $Q_1(X)$, and $Q_2(X)$ with a single polynomial $Q(X)$. We can do this because all three constraints have the same format: $\mathsf{Poly}_\mathsf{Vanish_i}(X)=0$. The batching technique is to create a new polynomial with all two $\mathsf{Poly}_\mathsf{Vanish_i}(X)$ values as coefficients. If and (overwhelmingly) only if all three are vanishing, then so will the new polynomial. This polynomial will be evaluated at a random challenge point $\rho$ selected after the commitments to the earlier polynomials are fixed. 

$Q(X) = \frac{\mathsf{Poly}_\mathsf{Vanish1}(X)+\rho\cdot\mathsf{Poly}_\mathsf{Vanish2}(X)}{X^n - 1}$

By rearranging, we can get $\mathsf{Poly}_\mathsf{Zero}(X)$ as a true zero polynomial (zero at every value both in $\mathcal{H}_\kappa$ and outside of it):

$\mathsf{Poly}_\mathsf{Zero}(X)=\mathsf{Poly}_\mathsf{Vanish1}(X) + \rho\cdot\mathsf{Poly}_\mathsf{Vanish2}(X)-Q(X)\cdot(X^{\kappa-1}-1)=0$

Ultimately the range gadget will satisfy the following constraints at the Commitment Level:

1. Show $Q(X)$ exists (as a polynomial that evenly divides the divisor)
2. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is correctly constructed from $\mathsf{Poly}_\mathsf{T}(X)$
3. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is the zero polynomial

### Commitment Level

The verifier will never see the arrays or polynomials themselves. They are undisclosed because they either (i) contain private data or (ii) are too large to examine and maintain a succinct proof system. Instead, the prover will use commitments.

The prover will write the following commitments to the transcript:

* $K_\mathsf{T}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{T}(X))$

The prover will generate a random challenge evaluation point (using strong Fiat-Shamir) on the polynomial that is outside of $\mathcal{H}_\kappa$. Call this point $\rho$. It will be used by the prover to create polynomial $Q(X)$ (see above) and the prover will write to the transcript:

* $\rho$
* $K_Q=\mathsf{KZG.Commit}(Q(X))$

The prover will generate a second random challenge evaluation point (using strong Fiat-Shamir) on the polynomial that is outside of $\mathcal{H}_\kappa$. Call this point $\zeta$. The prover will write the point and opening proofs to the transcript:

* $\zeta$
* $\mathsf{Poly}_\mathsf{T}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{T},\zeta)$
* $\mathsf{Poly}_\mathsf{T}(\zeta\omega)=\mathsf{KZG.Open}(K_\mathsf{T},\zeta\omega)$
* $Q(\zeta)=\mathsf{KZG.Open}(K_Q,\zeta)$

To check the proof, the verifier uses the transcript to construct the value $Y_\mathsf{Zero}$ as follows:

* $Y_\mathsf{Vanish1}=\mathsf{Poly}_\mathsf{T}(\zeta)\cdot[\mathsf{Poly}_\mathsf{T}(\zeta)-1]\cdot\frac{\zeta^\kappa-1}{\zeta-\omega^{\kappa-1}}$
* $Y_\mathsf{Vanish2}=[\mathsf{Poly}_\mathsf{T}(\zeta)-2\cdot\mathsf{Poly}_\mathsf{T}(\zeta\omega)]\cdot[\mathsf{Poly}_\mathsf{T}(\zeta)-2\cdot\mathsf{Poly}_\mathsf{T}(\zeta\omega)-1]\cdot(\zeta-\omega^{\kappa-1})$ 
* $Y_\mathsf{Zero}=Y_\mathsf{Vanish1}+\rho\cdot{Y_\mathsf{Vanish2}}-Q(\zeta)\cdot(\zeta^n-1)$

Finally, if the constraint system is true, the following constraint will be true (and will be false otherwise with overwhelming probability, due to the Schwartz-Zippel lemma on $\rho$ and $\zeta$) :

* $Y_\mathsf{Zero}\overset{?}{=}0$

## Security Proof

### Completeness

Any honest prover can do the computations explained above and create an accepting proof.

### Soundness

We prove knowledge soundness in the Algebraic Group Model (AGM). To do so, we must prove that there exists an efficient extractor $\mathcal{E}$ such that for any algebraic adversary $\mathcal{A}$ the probability of $\mathcal{A}$ winning the following game is $\mathsf{negl}(\lambda)$.

1. Given $[g,g^\tau,g^{\tau^2},\dots,g^{\tau^{n-1}}]$, $\mathcal{A}$ outputs commitments to $\mathsf{Poly}_\mathsf{T}(X)$ and $Q$
2. $\mathcal{E}$, given access to $\mathcal{A}$'s outputs from the previous step, outputs $\mathsf{Poly}_\mathsf{T}(X)$ and $Q$
3. $\mathcal{A}$ plays the part of the prover in showing that $Y_\mathsf{Zero}$ is zero at a random challenge $\zeta$
4. $\mathcal{A}$ wins if
   * $\mathcal{V}$ accepts at the end of the protocol
   * $\eta\notin[0,r]$

Our proof is as follows:

First, we prove $\eta\ge{0}$. To make $\mathsf{Poly}_\mathsf{Vanish1}$ exist, the last value of $\mathsf{T}$ must be zero or one. From $\mathsf{Poly}_\mathsf{Vanish2}$, it can be observed that $\mathsf{T}[i]\ge\mathsf{T}[i+1]$ for all $i\in[0,\kappa-2]$. Thus, $\mathsf{T}[0]$ is equal to or greater than $\mathsf{T}[\kappa-1]$, i.e., $\eta\ge{0}$.

Second, we prove $\eta\le{r}$. For simplicity, we assume $r$ is the power of two (recall $\kappa=\log_2{r}$). From $\mathsf{Poly}_\mathsf{Vanish2}$, we know $\mathsf{T}[0]$ is less than or equal to $2^\kappa$. Therefore, $\eta\le{r}$.

### Zero-Knowledge

To prove the above protocol is zero-knowledge, we do so by constructing a probabilistic polynomial time simulator $\mathcal{S}$ which, for every (possibly malicious) verifier $\mathcal{V}$, can output a view of the execution of the protocol that is indistinguishable from the view produced by the real execution of the program.

The simulator $\mathcal{S}$ randomly generates an $\eta^*$, then follows the same steps a prover would prove the lookup argument. $\mathcal{S}$ computes $\mathsf{T^*}$ and interpolates $\mathsf{Poly}_\mathsf{T^*}$ from $\mathsf{T^*}$. It computes $Q^*(X)$ and finally outputs the commitments to each of these polynomials (and writes the commitments to the transcript). Then, it generates the random challenge $\zeta^*$ (once again this is by strong Fiat-Shamir). It creates opening proofs for $\mathsf{Poly}_\mathsf{T^*}(\zeta^*),\mathsf{Poly}_\mathsf{T^*}(\zeta^*\omega)$, and $Q^*(\zeta^*)$, and writes these to the transcript as well. Since $\mathcal{S}$ knows each of the above polynomials, it can honestly compute this step and the proof will be accepted by $\mathcal{V}$. The transcript it generates this way will be indistinguishable from a transcript generated from a real execution since $\mathsf{PolyCommit}_\mathsf{Ped}$ has the property of Indistinguishability of Commitments due to the randomization by $h^{\hat{\phi}(x)}$.