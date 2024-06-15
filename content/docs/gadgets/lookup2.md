# Lookup (Type 2)

## Recap of types

| Type | Description | Recap | This |
| ---- | ----------- | ----- | ---- |
| lookup1  | $\mathsf{Arr}[i]\in \{0,1\}$ | Each element of array $\mathsf{Arr}$ is in $\{0,1\}$ (or another small set). |
| lookup2  | $\mathsf{Arr}[i]\in \mathsf{Table}$ | Each element of array $\mathsf{Arr}$ is in a disclosed table of values $\mathsf{Table}$. | ✅ |

## Relation

$ \mathcal{R}_{\mathtt{lookup2}} := \left\{ \begin{array}{l} (\mathsf{Arr},\mathsf{T}) \end{array} \middle | \begin{array}{l} \mathsf{Arr}[i]\in\mathsf{T}, 0\leq i \leq n-1, \\ \mathsf{Poly}_\mathsf{Arr}=\mathsf{FFT.Interp}(\omega,\mathsf{Arr}), \\ \mathsf{Poly}_\mathsf{T}=\mathsf{FFT.Interp}(\omega,\mathsf{T}), \\ K_\mathsf{Arr}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr}), \\ K_\mathsf{T}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{T}), \end{array} \right\} $

## Intuition

The prover ($\mathcal{P}$) holds an array $\mathsf{Arr}$ of $n$ integers from $\mathbb{Z}_q$: $[a_0, a_1, a_2, \dots, a_{n-1}]$. It will produce a succinct (independent of $n$) proof that each value in $\mathsf{Arr}$ is the element of a public table $\mathsf{T}$. The prover will encode $\mathsf{Arr}$ and $\mathsf{T}$ into polynomials: $\mathsf{Poly}_\mathsf{Arr}$ and $\mathsf{Poly}_\mathsf{T}$. Assume $\mathsf{Arr}$ is equal to $\mathsf{T}$, then the prover can complete the proof by simply performing the product check or the permutation check between $\mathsf{Arr}$ and $\mathsf{T}$. However, if $\mathsf{Arr}$ is not equal to $\mathsf{T}$ (this is the case we want to solve), the prover needs to reveal the points where $\mathsf{Arr}$ equals to $\mathsf{T}$, which makes zero knowledge impossible. Thus, the prover needs to construct auxiliary polynomial(s) to prove the constraints between $\mathsf{Arr}$ and $\mathsf{T}$ are desired. 

There are different approaches to achieving the lookup argument, e.g., [halo2](https://zcash.github.io/halo2/design/proving-system/lookup.html) and [Plookup](https://o1-labs.github.io/proof-systems/plonk/plookup.html). We will discuss these two approaches as follows.

### halo2

The lookup argument in halo2 requires the prover to construct two auxiliary vectors, $\mathsf{Arr}^\prime$ and $\mathsf{T}^\prime$, where $\mathsf{Arr}^\prime$ is the permutation of $\mathsf{Arr}$ and sorted (ascending or descending does not matter), $\mathsf{T}^\prime$ is the permutation of $\mathsf{T}$ and sorted by $\mathsf{Arr}$. For any two sets $A,B$ such that $A\subset{B}$, we say $B$ is sorted by $A$ when the elements in $A$ have the same order as they do in $B$. Let us demonstrate the halo2 lookup argument with a concrete example, $\mathsf{Arr}=\{1,2,1,6,4,5,3,0\},\mathsf{T}=\mathbb{Z}_8$. The prover constructs $\mathsf{Arr}^\prime$ and $\mathsf{T}^\prime$ such that
$$
\mathsf{Arr}^\prime=\{0,1,1,2,3,4,5,6\}
$$
$$
\mathsf{T}^\prime=\{0,1,7,2,3,4,5,6\}
$$
We can observe that $\mathsf{Arr}^\prime[i]$ is equal to $\mathsf{T}^\prime[i]$ or $\mathsf{Arr}^\prime[i-1]$. Since $\mathsf{Arr}^\prime[i-1]$ does not exist when $i=0$, we have to enforce the other rule, $\mathsf{Arr}^\prime[0]=\mathsf{T}^\prime[0]$. With these two constraints and the proof that $\mathsf{Arr}^\prime$ is the permutation of $\mathsf{Arr}$ and $\mathsf{T}^\prime$ is the permutation of $\mathsf{T}$, the prover can prove each element in $\mathsf{Arr}$ exists in $\mathsf{T}$.

### Plookup

Unlike halo2, Plookup requires only one auxiliary vector, $\mathsf{S}$, where $\mathsf{S}$ is the union set of $\mathsf{Arr}$ and $\mathsf{T}$, and sorted by $\mathsf{T}$. The prover encodes $\mathsf{Arr}$, $\mathsf{T}$, and $\mathsf{S}$ into polynomials: $\mathsf{Poly}_\mathsf{Arr}$, $\mathsf{Poly}_\mathsf{T}$, and $\mathsf{Poly}_\mathsf{S}$, and computes $\prod{\mathsf{Poly}_\mathsf{Arr}\cdot\prod\mathsf{Poly}_\mathsf{T}}$ and $\prod{\mathsf{Poly}_\mathsf{S}}$ with some random challenges. The theorem tells us the two products are equal if and only if: $\mathsf{Arr}\subset\mathsf{T}$, and $\mathsf{S}=(\mathsf{Arr},\mathsf{T})$ and sorted by $\mathsf{T}$.

## Protocol Details

### halo2

#### Array Level

* $\mathcal{P}$ and $\mathcal{V}$ are given a public table $\mathsf{T}$
* $\mathcal{P}$ holds an array $\mathsf{Arr}=[a_0,a_1,a_2,\dots,a_{n-1}]$ of $n$ integers ($a_i\in\mathbb{Z}_q$)
* $\mathcal{P}$ computes an array $\mathsf{Arr}^\prime=[a_0^\prime,a_1^\prime,a_2^\prime,\dots,a_{n-1}^\prime]$ of $n$ integers ($a_i^\prime\in\mathbb{Z}_q$) such that:
    * $\mathsf{Arr}^\prime$ is a permutation of $\mathsf{Arr}$ and sorted in ascending or descending
* $\mathcal{P}$ computes an array $\mathsf{T}^\prime$ such that:
    * $\mathsf{T}^\prime$ is a permutation of $\mathsf{T}$ and sorted by $\mathsf{Arr}^\prime$
* $\mathsf{Arr}^\prime$ and $\mathsf{T}^\prime$ have the following relations:
    * $\mathsf{Arr}^\prime[0]=\mathsf{T}^\prime[0]$
    * $\mathsf{Arr}^\prime[i]=\mathsf{T}^\prime[i]\mid\mathsf{Arr}^\prime[i-1],i\ne{0}$

#### Polynomial Level

We assume arrays $\mathsf{Arr}$, $\mathsf{Arr}^\prime$, $\mathsf{T}$, and $\mathsf{T}^\prime$ are encoded as the y-coordinates into a univariant polynomial where the x-coordinates (called the domain $\mathcal{H}_\kappa$) are chosen as the multiplicative group of order $\kappa$ with generator $\omega\in\mathbb{G}_\kappa$ (see [Background](../background/poly-iop.md) for more). In short, $\omega^0$ is the first element and $\omega^{\kappa-1}$ is the last element of $\mathcal{H}_\kappa$. If $\kappa$ is larger than the length of the array, the array can be padded with elements of value 1 (which will not change the product).

Recall the four constraints we want to prove:
1. $\mathsf{Arr}^\prime$ is a permutation of $\mathsf{Arr}$
2. $\mathsf{T}^\prime$ is a permutation of $\mathsf{T}$
3. The first value in $\mathsf{Arr}^\prime$ equals to the first value in $\mathsf{T}^\prime$
4. The rest of the values in $\mathsf{Arr}^\prime$ are of the form $\mathsf{Arr}^\prime[i]=\mathsf{T}^\prime[i]\mid\mathsf{Arr}^\prime[i-1],i\ne{0}$

In polynomial form, the constraints are ($\alpha,\beta$ are challenges from $\mathcal{V}$):
1. $\prod[\alpha-\mathsf{Poly}_{\mathsf{Arr}}(X)]=\prod[\alpha-\mathsf{Poly}_{\mathsf{Arr}^\prime}(X)]$
2. $\prod[\beta-\mathsf{Poly}_{\mathsf{T}}(X)]=\prod[\beta-\mathsf{Poly}_{\mathsf{T}^\prime}(X)]$
3. For $X=\omega^0$: $\mathsf{Poly}_{\mathsf{Arr}^\prime}(X)=\mathsf{Poly}_{\mathsf{T}^\prime}(X)$
4. For all $X\in\mathcal{H}_\kappa\setminus{\omega^0}$: $[\mathsf{Poly}_{\mathsf{Arr}^\prime}(X)-\mathsf{Poly}_{\mathsf{T}^\prime}(X)]\cdot[\mathsf{Poly}_{\mathsf{Arr}^\prime}(X)-\mathsf{Poly}_{\mathsf{Arr}^\prime}(X\cdot\omega^{-1})]=0$

We take care of the "for $X$" conditions by zeroing out the rest of the polynomial that is not zero. See the gadget <span style="border-style:dotted;border-width: 2px;"> [zero1](./zero1)</span> for more on why this works.

1. $\mathsf{Poly}_\mathsf{Vanish1}(X)=\prod[\alpha-\mathsf{Poly}_{\mathsf{Arr}}(X)]-\prod[\alpha-\mathsf{Poly}_{\mathsf{Arr}^\prime}(X)]=0$
2. $\mathsf{Poly}_\mathsf{Vanish2}(X)=\prod[\beta-\mathsf{Poly}_{\mathsf{T}}(X)]-\prod[\beta-\mathsf{Poly}_{\mathsf{T}^\prime}(X)]=0$
3. $\mathsf{Poly}_\mathsf{Vanish3}(X)=[\mathsf{Poly}_{\mathsf{Arr^\prime}}(X)-\mathsf{Poly}_{\mathsf{T^\prime}}(X)]\cdot\frac{X^\kappa-1}{X-\omega^0}=0$
4. $\mathsf{Poly}_\mathsf{Vanish4}(X)=[\mathsf{Poly}_{\mathsf{Arr}^\prime}(X)-\mathsf{Poly}_{\mathsf{T}^\prime}(X)]\cdot[\mathsf{Poly}_{\mathsf{Arr}^\prime}(X)-\mathsf{Poly}_{\mathsf{Arr}^\prime}(X\cdot\omega^{-1})]\cdot(X-\omega^0)=0$

Instead of proving $\mathsf{Poly}_\mathsf{Vanish1}$ and $\mathsf{Poly}_\mathsf{Vanish2}$ are vanishing through two [permutation]() checks, we can construct an accumulator $\mathsf{Poly}_\mathsf{Z}$ to make it more efficient (the domain needs to be expanded to $2\kappa$, denoted by $k$):
* $\mathsf{Poly}_\mathsf{Z}(\omega^0)=\mathsf{Poly}_\mathsf{Z}(\omega^{k-1})=1$
* For all $X\in\mathcal{H}_\kappa\setminus{\omega^{k-1}}$: $\mathsf{Poly}_\mathsf{Z}(X\cdot\omega)=\mathsf{Poly}_\mathsf{Z}(X)\cdot\frac{[\mathsf{Poly}_\mathsf{Arr}(X)+\alpha]\cdot[\mathsf{Poly}_\mathsf{T}(X)+\beta]}{[\mathsf{Poly}_\mathsf{Arr^\prime}(X)+\alpha]\cdot[\mathsf{Poly}_\mathsf{T^\prime}(X)+\beta]}$

Now we have the new $\mathsf{Poly}_\mathsf{Vanish1}$ and $\mathsf{Poly}_\mathsf{Vanish2}$:
1. $\mathsf{Poly}_\mathsf{Vanish1}=[\mathsf{Poly}_\mathsf{Z}(X)-1]\cdot\frac{X^k-1}{(X-\omega^0)\cdot(X-\omega^{k-1})}=0$
2. $\mathsf{Poly}_\mathsf{Vanish2}=\{\mathsf{Poly}_\mathsf{Z}(X\cdot\omega)\cdot[\mathsf{Poly}_\mathsf{Arr^\prime}(X)+\alpha]\cdot[\mathsf{Poly}_\mathsf{T^\prime}(X)+\beta]-\mathsf{Poly}_\mathsf{Z}(X)\cdot[\mathsf{Poly}_\mathsf{Arr}(X)+\alpha]\cdot[\mathsf{Poly}_\mathsf{T}(X)+\beta]\}\cdot(X-\omega^{k-1})=0$

These equations are true for every value of $X \in \mathcal{H}_k$ (but not necessarily true outside of these values). To show this, we divide each polynomial by  $X^k - 1$, which is a minimal vanishing polynomial for $\mathcal{H}_k$ that does not require interpolation to create. If the quotients are polynomials (and not rational functions), then $\mathsf{Poly}_\mathsf{Vanish1}(X)$, $\mathsf{Poly}_\mathsf{Vanish2}(X)$, $\mathsf{Poly}_\mathsf{Vanish3}(X)$, and $\mathsf{Poly}_\mathsf{Vanish4}(X)$ must be vanishing on $\mathcal{H}_k$ too. Specifically, the prover computes,

1. $Q_1(X) = \frac{\mathsf{Poly}_\mathsf{Vanish1}(X)}{X^k - 1}$
2. $Q_2(X) = \frac{\mathsf{Poly}_\mathsf{Vanish2}(X)}{X^k - 1}$
3. $Q_3(X) = \frac{\mathsf{Poly}_\mathsf{Vanish3}(X)}{X^k - 1}$
4. $Q_4(X) = \frac{\mathsf{Poly}_\mathsf{Vanish4}(X)}{X^k - 1}$

Instead of proving the four polynomials are zero polynomials one by one, we can linearly combine the four polynomials with a random challenge $\rho$ sent by $\mathcal{V}$ to compute:
* $W(X)=\mathsf{Poly}_\mathsf{Vanish1}(X)+\rho\cdot{\mathsf{Poly}_\mathsf{Vanish2}(X)}+\rho^2\cdot{\mathsf{Poly}_\mathsf{Vanish3}(X)}+\rho^3\cdot{\mathsf{Poly}_\mathsf{Vanish4}(X)}=0$

When $\mathsf{Poly}_\mathsf{Vanish1}(X),\mathsf{Poly}_\mathsf{Vanish2}(X),\mathsf{Poly}_\mathsf{Vanish3}(X),\mathsf{Poly}_\mathsf{Vanish4}(X)$ are vanishing on the domain $\mathcal{H}_k$, $W(X)$ is also vanishing with high probability. Again, if and only if $W(X)$ is vanishing over the field $\mathcal{H}_k$, $Q(X)=W(X)/(X^k-1)$ exists.

By rearranging, we can get $\mathsf{Poly}_\mathsf{Zero}(X)$ as a true zero polynomial (zero at every value both in $\mathcal{H}_\kappa$ and outside of it):
$$
\mathsf{Poly}_\mathsf{Zero}(X)=\mathsf{Poly}_\mathsf{Vanish1}(X)+\rho\cdot\mathsf{Poly}_\mathsf{Vanish2}(X)+\rho^2\cdot\mathsf{Poly}_\mathsf{Vanish3}(X)+\rho^3\cdot\mathsf{Poly}_\mathsf{Vanish4}(X)-Q(X)\cdot(X^n-1)=0
$$

Ultimately the halo2 lookup argument will satisfy the following constraints at the Commitment Level:
1. Show $Q(X)$ exists
2. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is correctly constructed from $\mathsf{Poly}_\mathsf{Z}(X)$,  $\mathsf{Poly}_\mathsf{Arr}(X)$, $\mathsf{Poly}_\mathsf{T}(X)$, $\mathsf{Poly}_\mathsf{Arr^\prime}(X)$, and $\mathsf{Poly}_\mathsf{T^\prime}(X)$
3. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is a zero polynomial

#### Commitment Level

The verifier will never see the arrays or polynomials themselves. They are undisclosed because they either (i) contain private data or (ii) are too large to examine and maintain a succinct proof system. Instead, the prover will use commitments.

The prover will write the following commitments to the transcript:

* $K_\mathsf{Z}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Z}(X))$
* $K_\mathsf{Arr}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr}(X))$
* $K_\mathsf{T}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{T}(X))$
* $K_\mathsf{Arr^\prime}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr^\prime}(X))$
* $K_\mathsf{T^\prime}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{T^\prime}(X))$

The prover will generate a random challenge evaluation point (using strong Fiat-Shamir) on the polynomial that is outside of $\mathcal{H}_k$. Call this point $\rho$. It will be used by the prover to create polynomial $Q(X)$ (see above) and the prover will write to the transcript: 

* $\rho$
* $K_Q=\mathsf{KZG.Commit}(Q(X))$

The prover will generate a second random challenge evaluation point (using strong Fiat-Shamir) on the polynomial that is outside of $\mathcal{H}_k$. Call this point $\zeta$. The prover will write the three points and opening proofs to the transcript:

* $\zeta$
* $\mathsf{Poly}_\mathsf{Z}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Z},\zeta)$
* $\mathsf{Poly}_\mathsf{Z}(\zeta\omega)=\mathsf{KZG.Open}(K_\mathsf{Z},\zeta\omega)$
* $\mathsf{Poly}_\mathsf{Arr}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr},\zeta)$
* $\mathsf{Poly}_\mathsf{Arr^\prime}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr^\prime},\zeta)$
* $\mathsf{Poly}_\mathsf{Arr^\prime}(\zeta\omega^{-1})=\mathsf{KZG.Open}(K_\mathsf{Arr^\prime},\zeta\omega^{-1})$
* $\mathsf{Poly}_\mathsf{T}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{T},\zeta)$
* $\mathsf{Poly}_\mathsf{T^\prime}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{T^\prime},\zeta)$
* $Q(\zeta)=\mathsf{KZG.Open}(K_Q,\zeta)$

To check the proof, the verifier uses the transcript to construct the value $Y_\mathsf{Zero}$ as follows:

* $Y_\mathsf{Vanish1}=[\mathsf{Poly}_\mathsf{Z}(\zeta)-1]\cdot\frac{(\zeta^k-1)}{(\zeta-\omega^0)\cdot(\zeta-\omega^{k-1})}$
* $Y_\mathsf{Vanish2}=\{\mathsf{Poly}_\mathsf{Z}(\zeta\omega)\cdot[\mathsf{Poly}_\mathsf{Arr^\prime}(\zeta)+\alpha]\cdot[\mathsf{Poly}_\mathsf{T^\prime}(\zeta)+\beta]-\mathsf{Poly}_\mathsf{Z}(\zeta)\cdot[\mathsf{Poly}_\mathsf{Arr}(\zeta)+\alpha]\cdot[\mathsf{Poly}_\mathsf{T}(\zeta)+\beta]\}\cdot(\zeta-\omega^{k-1})$
* $Y_\mathsf{Vanish3}=[\mathsf{Poly}_{\mathsf{Arr^\prime}}(\zeta)-\mathsf{Poly}_{\mathsf{T^\prime}}(\zeta)]\cdot\frac{\zeta^k-1}{\zeta-\omega^0}$
* $Y_\mathsf{Vanish4}=[\mathsf{Poly}_{\mathsf{Arr}^\prime}(\zeta)-\mathsf{Poly}_{\mathsf{T}^\prime}(\zeta)]\cdot[\mathsf{Poly}_{\mathsf{Arr}^\prime}(\zeta)-\mathsf{Poly}_{\mathsf{Arr}^\prime}(\zeta\omega^{-1})]\cdot(\zeta-\omega^0)$
* $Y_\mathsf{Zero}=Y_\mathsf{Vanish1}+\rho\cdot{Y_\mathsf{Vanish2}}+\rho^2\cdot{Y_\mathsf{Vanish3}}+\rho^3\cdot{Y_\mathsf{Vanish4}}-Q(\zeta)\cdot(\zeta^k - 1)$

Finally, if the constraint system is true, the following constraint will be true (and will be false otherwise with overwhelming probability, due to the Schwartz-Zippel lemma on $\rho$ and $\zeta$) :

* $Y_\mathsf{Zero}\overset{?}{=}0$

### Plookup
