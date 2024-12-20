# Lookup (Type 2)

## Recap of types

| Type | Description | Recap | This |
| ---- | ----------- | ----- | ---- |
| [lookup1](../lookup1)  | $\mathsf{Arr}[i]\in \{0,1\}$ | Each element of array $\mathsf{Arr}$ is in $\{0,1\}$ (or another small set). |
| [lookup2](#)  | $\mathsf{Arr}[i]\in \mathsf{Table}$ | Each element of array $\mathsf{Arr}$ is in a disclosed table of values $\mathsf{Table}$. | ✅ |

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

We start with an unoptimized version of Plookup that is conceptually simpler than the final version and is still fully succinct. The inputs are: an array containing the values of the lookup table $\mathsf{T}$ of size $n_1$; and an array of length $n_2$ (typically longer than $n_1$ but not necessarily) that will be proven to contain only values from somewhere in $\mathsf{T}$. 

The prover will create 5 helper arrays ($\mathsf{Acc_1}$ to $\mathsf{Acc_5}$) to demonstrate this overall property ($\mathsf{Arr}[i]\in\mathsf{T}$ for all $i$). The helper arrays will each be of size $n_1+n_2$ with the exception of $\mathsf{Acc_4}$ which is $n_2$. They are as follows:

1. $\mathsf{Acc_1}=\mathsf{Arr}||\mathsf{T}$
2. $\mathsf{Acc_2}=\mathtt{Sort}(\mathsf{Acc_1})$
3. $\mathsf{Acc_3}[i]=\mathsf{Acc_2}[i+1]-\mathsf{Acc_2}[i]$
4. $\mathsf{Acc_4}[i]=\mathsf{T}[i+1]-\mathsf{T}[i]$
5. $\mathsf{Acc_5}=\mathsf{Acc_4}||\{0\}^{n_2}$ where $n_1=|\mathsf{Arr}|$

It is probably worth an example at this point.

* $\mathsf{T}=[7,0,15,3]$
* $\mathsf{Arr}=[7,0,15,15,7,7,15,0,0,7,15,7]$
* $\mathsf{Acc_1}=[7,0,15,15,7,7,15,0,0,7,15,7,7,0,15,3]$
* $\mathsf{Acc_2}=[7,7,7,7,7,7,0,0,0,0,15,15,15,15,15,3]$
* $\mathsf{Acc_3}=[0,0,0,0,0,-7,0,0,0,15,0,0,0,0,-12,\bot]$
* $\mathsf{Acc_4}=[-7,15,-12,\bot]$
* $\mathsf{Acc_5}=[-7,15,-12,\bot,0,0,0,0,0,0,0,0,0,0,0,0]$

The first array $\mathsf{Acc_1}$ is the concatenation for $\mathsf{Arr}$ and $\mathsf{T}$. The intuition for this is as follows. Every element of $\mathsf{Arr}$ is in $\mathsf{T}$ (what is being proven) but the converse is not true, not every element of $\mathsf{T}$ is necessarily in $\mathsf{Arr}$. By constructing $\mathsf{Acc_1}$, the prover has an array where every element of $\mathsf{T}$ appears at least once. This will be convenient later. Additionally, if the prover can show every element in $\mathsf{Acc_1}$ is in $\mathsf{T}$, it implies every element in the original $\mathsf{Arr}$ is in $\mathsf{T}$ so it can now move forward with $\mathsf{Acc_1}$.

How does $\mathsf{Acc_1}$ compare to $\mathsf{T}$? They both have the same elements but $\mathsf{Acc_1}$ has a bunch of extra duplicates of values. Also the appearance of elements in $\mathsf{Acc_1}$ are in a different (arbitrary) order. Next the prover will sort the values of $\mathsf{Acc_1}$, grouping all duplicates together, and having them appear in the same order as the original $\mathsf{T}$ (which does not have to be sorted). 

Now how does $\mathsf{Acc_2}$ compare to $\mathsf{T}$? They have the same elements in the same order (including 3 which is not in the original $\mathsf{Arr}$) however $\mathsf{Acc_2}$ has a bunch of duplicates of some of the elements. Next we want to "flag" all the elements that are duplicates. The way we do this is to take the difference between each neighbouring elements. If the neighbouring elements are the same (duplicates), this is will place a 0 in that position. If they are not, a non-zero number will appear instead.

This is straight-forward until we hit the last element in $\mathsf{Acc_2}$ and $\mathsf{Acc_3}$ which has no "next" element in the array. We will leave it for now as an arbitrary integer $\bot$. 

We have marked the duplicates elements but have some other integers when neighbouring elements are not the same. The idea is do the same thing with $\mathsf{T}$ and we create $\mathsf{Acc_4}$, which we then pad with $n_2$ zeros.

* $\mathsf{Acc_5}=[-7,15,-12,\bot,0,0,0,0,0,0,0,0,0,0,0,0]$

If $\mathsf{Acc_5}$ is a permutation of $\mathsf{Acc_3}$ and everything else is correctly constructed, then all elements in $\mathsf{Arr}$ are from $\mathsf{T}$. 



The prover will show that $\mathsf{Acc_1}$ is constructed correctly with the $\mathtt{concat}$ gadget. It will not prove that $\mathsf{Acc_2}$ is sorted correctly (if it does not sort it correctly, the protocol will not work) but it will prove that $\mathsf{Acc_2}$ is a permutation of $\mathsf{Acc_1}$ using $\mathtt{shuffle1}(\mathsf{Acc_1},\mathsf{Acc_2})$. It will prove $\mathsf{Acc_3}$ is constructed correctly by $\mathsf{add1}(\mathsf{Acc_2}[i+1],-\mathsf{Acc_2}[i])$ and $\mathsf{Acc_3}$ is $\mathsf{add1}(\mathsf{T}[i+1],-\mathsf{T}[i])$. Last, it will prove that $\mathsf{Acc_5}$ is correctly formed with $\mathtt{concat}$ and finally, that it is a permutation of $\mathsf{Acc_3}$ with $\mathtt{shuffle1}(\mathsf{Acc_5},\mathsf{Acc_3})$.









 $\mathsf{T}$ to the end of $\mathsf{Arr}$ does not change anything about the argument  

In fact, the only difference between $\mathsf{Acc_1}$ and $\mathsf{T}$ itself is that there are several duplicates 



With these arrays, the following constraints are demonstrated:

1. Uses $\mathtt{concat}$ gadget
2. Uses $\mathtt{shuffle1}(\mathsf{Acc_1},\mathsf{Acc_2})$
3. Uses $\mathsf{add1}(\mathsf{Acc_2}[i+1],-\mathsf{Acc_2}[i])$
4. Uses $\mathsf{add1}(\mathsf{T}[i+1],-\mathsf{T}[i])$
5. Uses $\mathtt{concat}$ and then $\mathtt{shuffle1}(\mathsf{Acc_5},\mathsf{Acc_3})$



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

We assume arrays $\mathsf{Arr}$, $\mathsf{Arr}^\prime$, $\mathsf{T}$, and $\mathsf{T}^\prime$ are encoded as the y-coordinates into a univariant polynomial where the x-coordinates (called the domain $\mathcal{H}_\kappa$) are chosen as the multiplicative group of order $\kappa$ with generator $\omega\in\mathbb{G}_\kappa$ (see [Background](../../background/poly-iop) for more). In short, $\omega^0$ is the first element and $\omega^{\kappa-1}$ is the last element of $\mathcal{H}_\kappa$. If $\kappa$ is larger than the length of the array, the array can be padded with elements of value 1 (which will not change the product).

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

We take care of the "for $X$" conditions by zeroing out the rest of the polynomial that is not zero. See the gadget <span style="border-style:dotted;border-width: 2px;"> [zero1](../zero1)</span> for more on why this works.

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

#### Array Level

* $\mathcal{P}$ and $\mathcal{V}$ are given a public table $\mathsf{T}=[t_0,t_1,t_2,\dots,t_{d-1}]$ of $d$ integers ($t_i\in\mathbb{Z}_q$)
* $\mathcal{P}$ holds an array $\mathsf{Arr}=[a_0,a_1,a_2,\dots,a_{n-1}]$ of $n$ integers ($a_i\in\mathbb{Z}_q$)
* $\mathcal{P}$ constructs an array $\mathsf{S}=(\mathsf{Arr},\mathsf{T})=[s_0,s_1,s_2,\dots,s_{n+d-1}]$ of $n+d$ integers ($s_i\in\mathbb{Z}_q$) such that:
    * $\mathsf{S}$ is a union set of $\mathsf{Arr}$ and $\mathsf{T}$
    * $\mathsf{S}$ is sorted by $\mathsf{T}$
* $\mathsf{Arr}$, $\mathsf{T}$, and $\mathsf{S}$ have the following relations:
    * For each $i\in[0,d-1)$, there exists a $j\in[0,n+d-1)$ such that $(t_i,t_{i+1})=(s_j,s_{j+1})$
    * Let $I$ be the set of those $d-1$ indices, and let $I^\prime:=[0,n+d-1)\setminus{I}$. For each $i\in{I^\prime}$, there exists a $j\in[0,n)$ such that $s_i=s_{i+1}=a_{j}$

#### Polynomial Level

We assume arrays $\mathsf{Arr}$, $\mathsf{T}$, and $\mathsf{S}$ are encoded as the y-coordinates into a univariant polynomial where the x-coordinates (called the domain $\mathcal{H}_\kappa$) are chosen as the multiplicative group of order $\kappa$ with generator $\omega\in\mathbb{G}_\kappa$ (see [Background](../../background/poly-iop) for more). In short, $\omega^0$ is the first element and $\omega^{\kappa-1}$ is the last element of $\mathcal{H}_\kappa$. If $\kappa$ is larger than the length of the array, the array can be padded with elements of value 1 (which will not change the product).

Recall the two constraints we want to prove:
1. For each $i\in[0,d-1)$, there exists a $j\in[0,n+d-1)$ such that $(t_i,t_{i+1})=(s_j,s_{j+1})$
2. Let $I$ be the set of those $d-1$ indices, and let $I^\prime:=[0,n+d-1)\setminus{I}$. For each $i\in{I^\prime}$, there exists a $j\in[0,n)$ such that $s_i=s_{i+1}=a_{j}$

In polynomial form, the constraints are ($\alpha,\beta$ are challenges from $\mathcal{V}$):
1. $(1+\beta)^n\prod_{i<{n}}[\alpha+\mathsf{Poly}_{\mathsf{Arr}}(\omega^i)]\cdot\prod_{i<{d-1}}[\alpha(1+\beta)+\mathsf{Poly}_{\mathsf{T}}(\omega^i)+\beta\mathsf{Poly}_{\mathsf{T}}(\omega^{i+1})]=\prod_{i<{n+d-1}}[\alpha(1+\beta)+\mathsf{Poly}_{\mathsf{S}}(\omega^i)+\beta\mathsf{Poly}_{\mathsf{S}}(\omega^{i+1})]$

To efficiently prove the above polynomial holds, we can use a similar trick in halo2 lookup by constructing an accumulator :
* $\mathsf{Poly}_\mathsf{Z}(\omega^0)=\mathsf{Poly}_\mathsf{Z}(\omega^{n+d-1})=1$
* For $i\in[0,n+d-1)$: $\mathsf{Poly}_\mathsf{Z}(X\omega)=\mathsf{Poly}_\mathsf{Z}(X)\cdot\frac{(1+\beta)[\alpha+\mathsf{Poly}_{\mathsf{Arr}}(X)]\cdot[\alpha(1+\beta)+\mathsf{Poly}_{\mathsf{T}}(X)+\beta\mathsf{Poly}_{\mathsf{T}}(X\omega)]}{\alpha(1+\beta)+\mathsf{Poly}_{\mathsf{S}}(X)+\beta\mathsf{Poly}_{\mathsf{S}}(X\omega)}$

However, the above accumulator does not exist because the degree of the denominator, $\mathsf{Poly}_\mathsf{S}$, is different from $\mathsf{Poly}_\mathsf{Arr}$ and $\mathsf{Poly}_\mathsf{T}$. Specifically, there are $n$ elements in $\mathsf{Arr}$, $d$ elements in $\mathsf{T}$, but $n+d$ elements in $\mathsf{S}$. Thus we have to decompose the denominator to make it have the same iteration as $\mathsf{Arr}$ and $\mathsf{T}$ do. It is worth noting for the numerator, the left term contains $\mathsf{Poly}_\mathsf{Arr}(X)$ while the right term contains $\mathsf{Poly}_\mathsf{T}(X)$ and $\mathsf{Poly}_\mathsf{T}(X\omega)$. Therefore, it will be convenient to assume $d=n+1$. The order of $\mathsf{S}$ is $2n+1$. To traverse $\mathsf{S}$ in $n$ steps, we can halve it to $\mathsf{S}_\mathsf{l}=[s_0,s_1,\dots,s_{n-1},s_n]$ and $\mathsf{S}_\mathsf{h}=[s_{n},s_{n+1},\dots,s_{2n-1},s_{2n}]$, and prove $\mathsf{S}_\mathsf{l}[n]=\mathsf{S}_\mathsf{h}[0]$. Then we can compute the accumulator such that:
* $\mathsf{Poly}_\mathsf{Z}(\omega^0)=\mathsf{Poly}_\mathsf{Z}(\omega^{\kappa-1})=1$
* For $X\in\mathcal{H}_n\setminus{\omega^{\kappa-1}}$: $\mathsf{Poly}_\mathsf{Z}(X\omega)=\mathsf{Poly}_\mathsf{Z}(X)\cdot\frac{(1+\beta)[\alpha+\mathsf{Poly}_{\mathsf{Arr}}(X)]\cdot[\alpha(1+\beta)+\mathsf{Poly}_{\mathsf{T}}(X)+\beta\mathsf{Poly}_{\mathsf{T}}(X\omega)]}{[\alpha(1+\beta)+\mathsf{Poly}_{\mathsf{S}_\mathsf{l}}(X)+\beta\mathsf{Poly}_{\mathsf{S}_\mathsf{l}}(X\omega)]\cdot[\alpha(1+\beta)+\mathsf{Poly}_{\mathsf{S}_\mathsf{h}}(X)+\beta\mathsf{Poly}_{\mathsf{S}_\mathsf{h}}(X\omega)]}$
* For $X=\omega^{\kappa-1}$: $\mathsf{Poly}_{\mathsf{S}_\mathsf{l}}(X)=\mathsf{Poly}_{\mathsf{S}_\mathsf{h}}(X\omega)$

Similarly, we take care of the "for $X$" conditions by zeroing out the rest of the polynomial that is not zero. See the gadget <span style="border-style:dotted;border-width: 2px;"> [zero1](../zero1)</span> for more on why this works.

1. $\mathsf{Poly}_\mathsf{Vanish1}(X)=[\mathsf{Poly}_{\mathsf{Z}}(X)-1]\cdot\frac{X^n-1}{(X-\omega^0)(X-\omega^{\kappa-1})}=0$
2. $\displaylines{\mathsf{Poly}_\mathsf{Vanish2}(X)=\{\mathsf{Poly}_\mathsf{Z}(X\omega)\cdot[\alpha(1+\beta)+\mathsf{Poly}_{\mathsf{S}_\mathsf{l}}(X)+\beta\mathsf{Poly}_{\mathsf{S}_\mathsf{l}}(X\omega)]\cdot[\alpha(1+\beta)+\mathsf{Poly}_{\mathsf{S}_\mathsf{h}}(X)+\beta\mathsf{Poly}_{\mathsf{S}_\mathsf{h}}(X\omega)]-\\\mathsf{Poly}_\mathsf{Z}(X)\cdot(1+\beta)[\alpha+\mathsf{Poly}_{\mathsf{Arr}}(X)]\cdot[\alpha(1+\beta)+\mathsf{Poly}_{\mathsf{T}}(X)+\beta\mathsf{Poly}_{\mathsf{T}}(X\omega)]\}\cdot(X-\omega^{\kappa-1})}=0$
3. $\mathsf{Poly}_\mathsf{Vanish3}(X)=[\mathsf{Poly}_{\mathsf{S}_\mathsf{l}}(X)-\mathsf{Poly}_{\mathsf{S}_\mathsf{h}}(X\omega^{n+1})]\cdot\frac{X^n-1}{X-\omega^{\kappa-1}}=0$

These equations are true for every value of $X\in\mathcal{H}_n$ (but not necessarily true outside of these values). To show this, we divide each polynomial by  $X^n-1$, which is a minimal vanishing polynomial for $\mathcal{H}_n$ that does not require interpolation to create. If the quotients are polynomials (and not rational functions), then $\mathsf{Poly}_\mathsf{Vanish1}(X)$, $\mathsf{Poly}_\mathsf{Vanish2}(X)$, and $\mathsf{Poly}_\mathsf{Vanish3}(X)$ must be vanishing on $\mathcal{H}_n$ too. Specifically, the prover computes,

1. $Q_1(X) = \frac{\mathsf{Poly}_\mathsf{Vanish1}(X)}{X^n - 1}$
2. $Q_2(X) = \frac{\mathsf{Poly}_\mathsf{Vanish2}(X)}{X^n - 1}$
3. $Q_3(X) = \frac{\mathsf{Poly}_\mathsf{Vanish3}(X)}{X^n - 1}$

Instead of proving the three polynomials are zero polynomials one by one, we can linearly combine the three polynomials with a random challenge $\rho$ sent by $\mathcal{V}$ to compute:
* $W(X)=\mathsf{Poly}_\mathsf{Vanish1}(X)+\rho\cdot{\mathsf{Poly}_\mathsf{Vanish2}(X)}+\rho^2\cdot{\mathsf{Poly}_\mathsf{Vanish3}(X)}=0$

When $\mathsf{Poly}_\mathsf{Vanish1}(X),\mathsf{Poly}_\mathsf{Vanish2}(X),\mathsf{Poly}_\mathsf{Vanish3}(X)$ are vanishing on the domain $\mathcal{H}_n$, $W(X)$ is also vanishing with high probability. Again, if and only if $W(X)$ is vanishing over the field $\mathcal{H}_n$, $Q(X)=W(X)/(X^n-1)$ exists.

By rearranging, we can get $\mathsf{Poly}_\mathsf{Zero}(X)$ as a true zero polynomial (zero at every value both in $\mathcal{H}_n$ and outside of it):
$$
\mathsf{Poly}_\mathsf{Zero}(X)=\mathsf{Poly}_\mathsf{Vanish1}(X)+\rho\cdot\mathsf{Poly}_\mathsf{Vanish2}(X)+\rho^2\cdot\mathsf{Poly}_\mathsf{Vanish3}(X)-Q(X)\cdot(X^n-1)=0
$$

Ultimately the Plookup will satisfy the following constraints at the Commitment Level:
1. Show $Q(X)$ exists
2. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is correctly constructed from $\mathsf{Poly}_\mathsf{Z}(X)$,  $\mathsf{Poly}_\mathsf{Arr}(X)$, $\mathsf{Poly}_\mathsf{T}(X)$, $\mathsf{Poly}_{\mathsf{S}_\mathsf{l}}(X)$, and $\mathsf{Poly}_{\mathsf{S}_\mathsf{h}}(X)$
3. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is a zero polynomial

#### Commitment Level

The verifier will never see the arrays or polynomials themselves. They are undisclosed because they either (i) contain private data or (ii) are too large to examine and maintain a succinct proof system. Instead, the prover will use commitments.

The prover will write the following commitments to the transcript:

* $K_\mathsf{Z}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Z}(X))$
* $K_\mathsf{Arr}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr}(X))$
* $K_\mathsf{T}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{T}(X))$
* $K_{\mathsf{S}_\mathsf{l}}=\mathsf{KZG.Commit}(\mathsf{Poly}_{\mathsf{S}_\mathsf{l}}(X))$
* $K_{\mathsf{S}_\mathsf{h}}=\mathsf{KZG.Commit}(\mathsf{Poly}_{\mathsf{S}_\mathsf{h}}(X))$

The prover will generate a random challenge evaluation point (using strong Fiat-Shamir) on the polynomial that is outside of $\mathcal{H}_n$. Call this point $\rho$. It will be used by the prover to create polynomial $Q(X)$ (see above) and the prover will write to the transcript: 

* $\rho$
* $K_Q=\mathsf{KZG.Commit}(Q(X))$

The prover will generate a second random challenge evaluation point (using strong Fiat-Shamir) on the polynomial that is outside of $\mathcal{H}_n$. Call this point $\zeta$. The prover will write the three points and opening proofs to the transcript:

* $\zeta$
* $\mathsf{Poly}_\mathsf{Z}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Z},\zeta)$
* $\mathsf{Poly}_\mathsf{Z}(\zeta\omega)=\mathsf{KZG.Open}(K_\mathsf{Z},\zeta\omega)$
* $\mathsf{Poly}_\mathsf{Arr}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr},\zeta)$
* $\mathsf{Poly}_\mathsf{T}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{T},\zeta)$
* $\mathsf{Poly}_\mathsf{T}(\zeta\omega)=\mathsf{KZG.Open}(K_\mathsf{T},\zeta\omega)$
* $\mathsf{Poly}_{\mathsf{S}_\mathsf{l}}(\zeta)=\mathsf{KZG.Open}(K_{\mathsf{S}_\mathsf{l}},\zeta)$
* $\mathsf{Poly}_{\mathsf{S}_\mathsf{l}}(\zeta\omega)=\mathsf{KZG.Open}(K_{\mathsf{S}_\mathsf{l}},\zeta\omega)$
* $\mathsf{Poly}_{\mathsf{S}_\mathsf{h}}(\zeta)=\mathsf{KZG.Open}(K_{\mathsf{S}_\mathsf{h}},\zeta)$
* $\mathsf{Poly}_{\mathsf{S}_\mathsf{h}}(\zeta\omega)=\mathsf{KZG.Open}(K_{\mathsf{S}_\mathsf{h}},\zeta\omega)$
* $Q(\zeta)=\mathsf{KZG.Open}(K_Q,\zeta)$

To check the proof, the verifier uses the transcript to construct the value $Y_\mathsf{Zero}$ as follows:

* $Y_\mathsf{Vanish1}=[\mathsf{Poly}_\mathsf{Z}(\zeta)-1]\cdot\frac{(\zeta^n-1)}{(\zeta-\omega^0)\cdot(\zeta-\omega^{\kappa-1})}$
* $\displaylines{Y_\mathsf{Vanish2}=\{\mathsf{Poly}_\mathsf{Z}(\zeta\omega)\cdot[\alpha(1+\beta)+\mathsf{Poly}_{\mathsf{S}_\mathsf{l}}(\zeta)+\beta\mathsf{Poly}_{\mathsf{S}_\mathsf{l}}(\zeta\omega)]\cdot[\alpha(1+\beta)+\mathsf{Poly}_{\mathsf{S}_\mathsf{h}}(\zeta)+\beta\mathsf{Poly}_{\mathsf{S}_\mathsf{h}}(\zeta\omega)]-\\\mathsf{Poly}_\mathsf{Z}(\zeta)\cdot(1+\beta)[\alpha+\mathsf{Poly}_\mathsf{Arr}(\zeta)]\cdot[\alpha(1+\beta)+\mathsf{Poly}_\mathsf{T}(\zeta)+\beta\mathsf{Poly}_\mathsf{T}(\zeta\omega)]\}\cdot(\zeta-\omega^{\kappa-1})}$
* $Y_\mathsf{Vanish3}=[\mathsf{Poly}_{\mathsf{S}_\mathsf{l}}(\zeta)-\mathsf{Poly}_{\mathsf{S}_\mathsf{l}}(\zeta\omega^{n+1})]\cdot\frac{\zeta^n-1}{\zeta-\omega^{\kappa-1}}$
* $Y_\mathsf{Zero}=Y_\mathsf{Vanish1}+\rho\cdot{Y_\mathsf{Vanish2}}+\rho^2\cdot{Y_\mathsf{Vanish3}}-Q(\zeta)\cdot(\zeta^k - 1)$

Finally, if the constraint system is true, the following constraint will be true (and will be false otherwise with overwhelming probability, due to the Schwartz-Zippel lemma on $\rho$ and $\zeta$) :

* $Y_\mathsf{Zero}\overset{?}{=}0$

## Security Proof

### halo2

#### Completeness

Any honest prover can do the computations explained above and create an accepting proof.

#### Soundness

We prove knowledge soundness in the Algebraic Group Model (AGM). To do so, we must prove that there exists an efficient extractor $\mathcal{E}$ such that for any algebraic adversary $\mathcal{A}$ the probability of $\mathcal{A}$ winning the following game is $\mathsf{negl}(\lambda)$.

1. Given $[g,g^\tau,g^{\tau^2},\dots,g^{\tau^{n-1}}]$, $\mathcal{A}$ outputs commitments to $\mathsf{Poly}_\mathsf{Arr}(X)$, $\mathsf{Poly}_\mathsf{Arr^\prime}(X)$, $\mathsf{Poly}_\mathsf{T}(X)$, $\mathsf{Poly}_\mathsf{T^\prime}(X)$, $Q$
2. $\mathcal{E}$, given access to $\mathcal{A}$'s outputs from the previous step, outputs $\mathsf{Poly}_\mathsf{Arr}(X)$, $\mathsf{Poly}_\mathsf{Arr^\prime}(X)$, $\mathsf{Poly}_\mathsf{T}(X)$, $\mathsf{Poly}_\mathsf{T^\prime}(X)$, $Q$
3. $\mathcal{A}$ plays the part of the prover in showing that $Y_\mathsf{Zero}$ is zero at a random challenge $\zeta$
4. $\mathcal{A}$ wins if
    * $\mathcal{V}$ accepts at the end of the protocol
    * For some $i\in[0,\kappa-1]$, $\mathsf{Arr}[i]\notin\mathsf{T}$

Our proof is as follows:

To make $Y_\mathsf{Zero}$ a zero polynomial, $\mathcal{A}$ has to prove the four vanishing polynomials are correct. For $\mathsf{Poly}_\mathsf{Vanish1}$ and $\mathsf{Poly}_\mathsf{Vanish2}$, the [permutation]() check tells us the probability that $\mathcal{V}$ accepts the proof is negligible if some elements in $\mathsf{Arr}$ are not in $\mathsf{T}$. By the Schwartz-Zippel lemma, we know the probability that $\mathsf{Poly}_\mathsf{Vanish3}$ is vanishing is negligible if $\mathsf{Poly}_\mathsf{Arr^\prime}(\omega^0)\ne\mathsf{Poly}_\mathsf{T^\prime}(\omega^0)$. Therefore, to win the KS game, $\mathcal{A}$ has to prove $\mathsf{Poly}_\mathsf{Vanish4}$ is correct with the winning condition (some elements in $\mathsf{Arr}$ do not appear in $\mathsf{T}$). Assume $\mathsf{Arr}[i^\prime]\notin\mathsf{T},i>0$, to make such $\mathsf{Poly}_\mathsf{Vanish4}$ exist, $\mathsf{Arr}[i^\prime]$ has to equal to $\mathsf{Arr}[i^\prime-1]$. And because $\mathsf{Arr}[i^\prime-1]\ne{\mathsf{T}[i^\prime-1]}$, $\mathsf{Arr}[i^\prime-1]$ has to equal to $\mathsf{Arr}[i^\prime-2]$. Thus, to make the winning condition hold, $\mathsf{Arr}[i^\prime]=\mathsf{Arr}[0]$ must hold, which contradicts to the condition of $\mathsf{Poly}_\mathsf{Vanish3}$, $\mathsf{Arr}[0]=\mathsf{T}[0]$.

#### Zero-Knowledge

Before we prove the above protocol is zero-knowledge, it is worth noting the protocol is different from the lookup argument of halo2 in the real world. Specifically, the real halo2 lookup optimizes the two permutation checks into one and fills the table with some random numbers for the PLONK-based proof system. We refer to the official [halo2 handbook](https://zcash.github.io/halo2/design/proving-system/lookup.html#zero-knowledge-adjustment) to see the details. To prove the above protocol is zero-knowledge, we do so by constructing a probabilistic polynomial time simulator $\mathcal{S}$ which, for every (possibly malicious) verifier $\mathcal{V}$, can output a view of the execution of the protocol that is indistinguishable from the view produced by the real execution of the program.

The simulator $\mathcal{S}$ generates an array $\mathsf{Arr^*}$ by randomly filling it with elements from $\mathsf{T}$, then follows the same steps a prover would prove the lookup argument. $\mathcal{S}$ computes $\mathsf{Arr^{*^\prime}},\mathsf{T^\prime}$ and interpolates the four arrays into their respective polynomials, $\mathsf{Poly}_\mathsf{Arr^*}(X)$, $\mathsf{Poly}_\mathsf{Arr^{*^\prime}}(X)$, $\mathsf{Poly}_\mathsf{T}(X)$, and $\mathsf{Poly}_\mathsf{T^\prime}(X)$. It computes $Q^*(X)$ and finally outputs the commitments to each of these polynomials (and writes the commitments to the transcript). Then, it generates the random challenge $\zeta^*$ (once again this is by strong Fiat-Shamir). It creates opening proofs for $\mathsf{Poly}_\mathsf{Arr^*}(\zeta^*),\mathsf{Poly}_\mathsf{Arr^{*^\prime}}(\zeta^*),Q^*(\zeta^*)$, $\mathsf{Poly}_\mathsf{T}(\zeta^*\omega)$, and $\mathsf{Poly}_\mathsf{T^\prime}(\zeta^*\omega)$, and writes these to the transcript as well. Since $\mathcal{S}$ knows each of the above polynomials, it can honestly compute this step and the proof will be accepted by $\mathcal{V}$. The transcript it generates this way will be indistinguishable from a transcript generated from a real execution, since $\mathsf{PolyCommit}_\mathsf{Ped}$ has the property of Indistinguishability of Commitments due to the randomization by $h^{\hat{\phi}(x)}$.

### Plookup

#### Completeness

Any honest prover can do the computations explained above and create an accepting proof.

#### Soundness

We prove knowledge soundness in the Algebraic Group Model (AGM). To do so, we must prove that there exists an efficient extractor $\mathcal{E}$ such that for any algebraic adversary $\mathcal{A}$ the probability of $\mathcal{A}$ winning the following game is $\mathsf{negl}(\lambda)$.

1. Given $[g,g^\tau,g^{\tau^2},\dots,g^{\tau^{n-1}}]$, $\mathcal{A}$ outputs commitments to $\mathsf{Poly}_\mathsf{Arr}(X)$, $\mathsf{Poly}_\mathsf{T}(X)$, $\mathsf{Poly}_\mathsf{S}(X)$, $Q$
2. $\mathcal{E}$, given access to $\mathcal{A}$'s outputs from the previous step, outputs $\mathsf{Poly}_\mathsf{Arr}(X)$, $\mathsf{Poly}_\mathsf{T}(X)$, $\mathsf{Poly}_\mathsf{S}(X)$, $Q$
3. $\mathcal{A}$ plays the part of the prover in showing that $Y_\mathsf{Zero}$ is zero at a random challenge $\zeta$
4. $\mathcal{A}$ wins if
    * $\mathcal{V}$ accepts at the end of the protocol
    * For some $i\in[0,\kappa-1]$, $\mathsf{Arr}[i]\notin\mathsf{T}$

Our proof is as follows:

To make $Y_\mathsf{Zero}$ a zero polynomial, $\mathcal{A}$ has to prove the three vanishing polynomials are correct. It is easy to observe that $\mathsf{Poly}_\mathsf{Vanish1}$ and $\mathsf{Poly}_\mathsf{Vanish3}$ can be constructed even if some elements in $\mathsf{Arr}$ do not appear in $\mathsf{T}$, so we focus on the $\mathsf{Poly}_\mathsf{Vanish2}$. By the soundness of the [product check](../mult3/#soundness), we know $\mathsf{S}$ must be the union set of $\mathsf{Arr}$ and $\mathsf{T}$ if we want to the product of $\mathsf{S}[i]$ equals to the product of $\mathsf{Arr}[i]$ and $\mathsf{T}[i]$. Recall the equation $\mathcal{A}$ needs to prove:
$$
(1+\beta)^n\prod_{i<{n}}[\alpha+\mathsf{Poly}_{\mathsf{Arr}}(\omega^i)]\cdot\prod_{i<{d-1}}[\alpha(1+\beta)+\mathsf{Poly}_{\mathsf{T}}(\omega^i)+\beta\mathsf{Poly}_{\mathsf{T}}(\omega^{i+1})]=\prod_{i<{n+d-1}}[\alpha(1+\beta)+\mathsf{Poly}_{\mathsf{S}}(\omega^i)+\beta\mathsf{Poly}_{\mathsf{S}}(\omega^{i+1})]
$$
Thus, for each $i\in[0,d-1]$, there exists a factor in the right-hand side of the equation equal to $\alpha(1+\beta)+\mathsf{Poly}_{\mathsf{T}}(\omega^i)+\beta\mathsf{Poly}_{\mathsf{T}}(\omega^{i+1})$, which means $\alpha(1+\beta)+\mathsf{Poly}_{\mathsf{T}}(\omega^i)+\beta\mathsf{Poly}_{\mathsf{T}}(\omega^{i+1})=\alpha(1+\beta)+\mathsf{Poly}_{\mathsf{S}}(\omega^i)+\beta\mathsf{Poly}_{\mathsf{S}}(\omega^{i+1})$. We can get $\mathsf{Poly}_{\mathsf{T}}(\omega^i)=\mathsf{Poly}_{\mathsf{S}}(\omega^i)$ and $\mathsf{Poly}_{\mathsf{T}}(\omega^{i+1})=\mathsf{Poly}_{\mathsf{S}}(\omega^{i+1})$ for $i\in[0,d-1]$. Similarly, we can get $\mathsf{Poly}_{\mathsf{Arr}}(\omega^i)=\mathsf{Poly}_{\mathsf{S}}(\omega^i)=\mathsf{Poly}_{\mathsf{S}}(\omega^{i+1})$ for $i\in[0,n]$. Since $n$ should be equal to or less than $d$, when $\mathsf{Poly}_{\mathsf{Arr}}(\omega^i)=\mathsf{Poly}_{\mathsf{S}}(\omega^i)$, $\mathsf{Poly}_{\mathsf{T}}(\omega^i)=\mathsf{Poly}_{\mathsf{S}}(\omega^i)$ must hold at the same time, which contradicts to the winning assumption. Therefore, the protocol is sound.

#### Zero-Knowledge

To prove the above protocol is zero-knowledge, we do so by constructing a probabilistic polynomial time simulator $\mathcal{S}$ which, for every (possibly malicious) verifier $\mathcal{V}$, can output a view of the execution of the protocol that is indistinguishable from the view produced by the real execution of the program.

The simulator $\mathcal{S}$ generates an array $\mathsf{Arr^*}$ by randomly filling it with elements from $\mathsf{T}$, then follows the same steps a prover would prove the lookup argument. $\mathcal{S}$ computes $\mathsf{S}_\mathsf{l}$, $\mathsf{S}_\mathsf{h}$, and $\mathsf{Z}$ and interpolates the five arrays into their respective polynomials, $\mathsf{Poly}_\mathsf{Arr^*}(X)$, $\mathsf{Poly}_\mathsf{T}(X)$, $\mathsf{Poly}_{\mathsf{S}_\mathsf{l}}(X)$, $\mathsf{Poly}_{\mathsf{S}_\mathsf{h}}(X)$, and $\mathsf{Poly}_{\mathsf{Z}}(X)$. It computes $Q^*(X)$ and finally outputs the commitments to each of these polynomials (and writes the commitments to the transcript). Then, it generates the random challenge $\zeta^*$ (once again this is by strong Fiat-Shamir). It creates opening proofs for $\mathsf{Poly}_\mathsf{Arr^*}(\zeta^*),\mathsf{Poly}_\mathsf{T}(\zeta^*),\mathsf{Poly}_\mathsf{T}(\zeta^*\omega),\mathsf{Poly}_{\mathsf{S}_\mathsf{l}}(\zeta^*),\mathsf{Poly}_{\mathsf{S}_\mathsf{l}}(\zeta^*\omega),\mathsf{Poly}_{\mathsf{S}_\mathsf{h}}(\zeta^*),\mathsf{Poly}_{\mathsf{S}_\mathsf{h}}(\zeta^*\omega),Q^*(\zeta^*),\mathsf{Poly}_\mathsf{Z}(\zeta^*)$, and $\mathsf{Poly}_\mathsf{Z}(\zeta^*\omega)$, and writes these to the transcript as well. Since $\mathcal{S}$ knows each of the above polynomials, it can honestly compute this step and the proof will be accepted by $\mathcal{V}$. The transcript it generates this way will be indistinguishable from a transcript generated from a real execution since $\mathsf{PolyCommit}_\mathsf{Ped}$ has the property of Indistinguishability of Commitments due to the randomization by $h^{\hat{\phi}(x)}$.
