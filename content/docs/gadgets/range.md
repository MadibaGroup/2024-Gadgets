# Range

## Recap of types

| Type | Description | Recap | This |
| ---- | ----------- | ----- | ---- |
| range | $\mathsf{Arr}[i]\in[0,r]$ | Each element of array $\mathsf{Arr}$ is in the range $[0,r]$ | ✅ |

## Relation

$\mathcal{R}_{\mathtt{add1}} := \left\{ \begin{array}{l} (K_\mathsf{Arr}) \end{array} \middle | \begin{array}{l} 0\le{\mathsf{Arr}[i]}\le{r}, i\in[0,n), \\ \mathsf{Poly}_\mathsf{Arr}=\mathsf{FFT.Interp}(\omega,\mathsf{Arr}), \\ K_\mathsf{Arr}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr}), \end{array} \right\}$

## Intuition

To prove each element of array $\mathsf{Arr}$ is in the range $[0,r]$, one of the most intuitive ways is we create a vector containing the numbers from $0$ to $r$ and run the lookup argument for $\mathsf{Arr}$. Another approach is we prove each element is in $[0,r]$. Specifically, we decompose the target number to digits in some base $x$ and prove (i) the digits are valid and (ii) the number can be recovered from the digits and the base. The prover ($\mathcal{P}$) holds a number $\eta$ and $\mathsf{Arr}$ of $k=\lceil\log_x{\eta}\rceil$ integers from $\mathbb{Z}_q$: $[a_0,a_1,a_2,\dots,a_{k-1}]$. It will produce a succinct (logarithm base $x$ of $\eta$. For simplicity, we will use base $2$) proof that the vector $\mathsf{T}$ satisfies the following conditions: (i) the first value of $\mathsf{T}$ equals to $\eta$ (ii) the last value of $\mathsf{T}$ equals to one or zero (iii) any value minus two times the next value is equal to one or zero in $\mathsf{T}$. The prover will encode $\mathsf{Arr}$ and $\mathsf{T}$ into two polynomials: $\mathsf{Poly}_\mathsf{Arr}$ and $\mathsf{Poly}_\mathsf{T}$ (using [evaluation points]() on the domain $\mathcal{H}_\kappa$). He will commit to each polynomial: $K_\mathsf{Arr}$ and $K_\mathsf{T}$. The verifier ($\mathcal{V}$) cannot check any of the $\mathsf{Arr}$, $\mathsf{T}$ or $\mathsf{Poly}_\mathsf{Arr}$, $\mathsf{Poly}_\mathsf{T}$ values directly. Instead the verifier only sees $K_\mathsf{Arr}$, and $K_\mathsf{T}$.

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

We assume the array $\mathsf{T}$ is encoded as the y-coordinates into a univariant polynomial where the x-coordinates (called the domain $\mathcal{H}_\kappa$) are chosen as the multiplicative group of order $\kappa$ with generator $\omega\in\mathbb{G}_\kappa$ (see [Background](../background/poly-iop.md) for more). In short, $\omega^0$ is the first element and $\omega^{\kappa-1}$ is the last element of $\mathcal{H}_\kappa$. If $\kappa$ is larger than the length of the array, the array can be padded with elements of value 0 (which will not change the sum).

Recall the constraints we want to prove: 
1. $\mathsf{T}[0]=\eta$
2. $\mathsf{T}[k-1]\in\{0,1\}$
3. $\mathsf{T}[i]-2\cdot\mathsf{T}[i+1]\in\{0,1\}$

In polynomial form, the constraints are:
1. For $X=\omega^0$: $\mathsf{Poly}_\mathsf{T}(X)=\eta$
2. For $X=\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{T}(X)\in\{0,1\}$
3. For all $X=\mathcal{H}_\kappa\setminus{\omega^{\kappa-1}}$: $\mathsf{Poly}_\mathsf{T}(X)-2\cdot\mathsf{Poly}_\mathsf{T}(X\omega)\in\{0,1\}$

Note because the value of $\eta$ is a secret, $\mathcal{P}$ will not reveal $\eta$ to let $\mathcal{V}$ verify $\eta$ is the evaluation of $\mathsf{Poly}_\mathsf{T}(\omega^0)$. $\mathcal{P}$ will leverage the hiding property of KZG (Pedersen) commitment to prove the committed $\eta$ is the correct evaluation. Specifically, $\mathcal{P}$ claims the committed $\eta$ is the correct one and opens $\mathsf{Poly}_\mathsf{T}$ at $\omega^0$. If the committed $\eta$ satisfy the KZG verification, $\mathcal{V}$ can believe the first constraint is satisfied.

We take care of the "for $X$" conditions of constraints 2 and 3 by zeroing out the rest of the polynomial that is not zero. See the gadget <span style="border-style:dotted;border-width: 2px;"> [zero1](./zero1)</span> for more on why this works.

1. $\mathsf{Poly}_\mathsf{Vanish1}(X)=\mathsf{Poly}_\mathsf{T}(X)\cdot[\mathsf{Poly}_\mathsf{T}(X)-1]\cdot\frac{X^\kappa-1}{X-\omega^{\kappa-1}}=0$
2. $\mathsf{Poly}_\mathsf{Vanish2}(X)=[\mathsf{Poly}_\mathsf{T}(X)-2\cdot\mathsf{Poly}_\mathsf{T}(X\omega)]\cdot[\mathsf{Poly}_\mathsf{T}(X)-2\cdot\mathsf{Poly}_\mathsf{T}(X\omega)-1]\cdot(X-\omega^{\kappa-1})=0$

The two equations are vanishing for every value of $X\in\mathcal{H}_\kappa$ (but not necessarily true outside of these values). To show this, we divide each polynomial by $X^\kappa-1$, which is a minimal vanishing polynomial for $\mathcal{H}_\kappa$ that does not require interpolation to create. If the quotient is polynomial (and not a rational function), then $\mathsf{Poly}_\mathsf{Vanish1}(X)$ and $\mathsf{Poly}_\mathsf{Vanish2}(X)$ must be vanishing on $\mathcal{H}_\kappa$ too. Specifically, the prover computes:

1. $Q_1(X) = \frac{\mathsf{Poly}_\mathsf{Vanish1}(X)}{X^\kappa - 1}$
2. $Q_2(X) = \frac{\mathsf{Poly}_\mathsf{Vanish2}(X)}{X^\kappa - 1}$

We can replace polynomials $Q_1(X)$, and $Q_2(X)$ with a single polynomial $Q(X)$. We can do this because all three constraints have the same format: $\mathsf{Poly}_\mathsf{Vanish_i}(X)=0$. The batching technique is to create a new polynomial with all three $\mathsf{Poly}_\mathsf{Vanish_i}(X)$ values as coefficients. If and (overwhelmingly) only if all three are vanishing, then so will the new polynomial. This polynomial will be evaluated at a random challenge point $\rho$ selected after the commitments to the earlier polynomials are fixed. 

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
