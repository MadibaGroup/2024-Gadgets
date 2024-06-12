# Addition (Type 1)

## Recap of types

| Type           | Description                                                  | Recap                                                        | This |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| [add1](./add1) | $\mathsf{Arr}_3=\mathsf{Arr}_1 + \mathsf{Arr}_2$             | $\mathsf{Arr}_3$ is the element-wise addition of $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$. | ✅    |
| [add2](./add2) | $\mathsf{Sum}_\mathsf{Arr}=\sum_{i = 0}^{n-1} \mathsf{Arr}[i]$ | $\mathsf{Sum}_\mathsf{Arr}$ is the disclosed sum of all the elements in $\mathsf{Arr}$. |      |
| [add3](./add3) | $\sum_{i = 0}^{n-1} \mathsf{Arr}_1[i]=\sum_{i = 0}^{n-1} \mathsf{Arr}_2[i]$ | $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$ have the same undisclosed sum. |      |

## Relation

$ \mathcal{R}_{\mathtt{mult2}} := \left\{ \begin{array}{l} (K_\mathsf{Arr_1},K_\mathsf{Arr_2},K_\mathsf{Arr_3}) \end{array} \middle | \begin{array}{l} \mathsf{Arr_3}[i]=\mathsf{Arr_1}[i]+\mathsf{Arr_2}[i], 0\leq i \leq n, \\ \mathsf{Poly}_\mathsf{Arr_j}=\mathsf{FFT.Interp}(\omega,\mathsf{Arr_j}), 1\leq j \leq 3, \\ K_\mathsf{Arr_j}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_j}), 1\leq j \leq 3, \end{array} \right\} $​

## Intuition

The prover ($\mathcal{P}$) holds two arrays $\mathsf{Arr_1}$ and $\mathsf{Arr_2}$ of $n$ integers from $\mathbb{Z}_q$: $[a_0, a_1, a_2, \dots, a_{n-1}]$. It will produce a succinct (independent of $n$) proof that $\mathsf{Arr_3}$ is the element-wise sum of all the elements in the array: $\mathsf{Arr_3}[i]=\mathsf{Arr_1}[i]+\mathsf{Arr_2}[i]$. The prover will encode the three arrays into three polynomials: $\mathsf{Poly}_\mathsf{Arr_1}$, $\mathsf{Poly}_\mathsf{Arr_2}$, and $\mathsf{Poly}_\mathsf{Arr_3}$ (using [evaluation points]() on the domain $\mathcal{H}_\kappa$). It will commit to each polynomial: $K_\mathsf{Arr_1}$,$K_\mathsf{Arr_2}$, and $K_\mathsf{Arr_3}$. The verifier ($\mathcal{V}$) cannot check any of the $\mathsf{Arr_i}$ or $\mathsf{Poly}_\mathsf{Arr_i}$ values directly (they may contain secret information, and even if they do not, they are too long to check) so the verifier only sees $K_\mathsf{Arr_1}$,$K_\mathsf{Arr_2}$, and $K_\mathsf{Arr_3}$.

In order to prove$K_\mathsf{Arr_1}$,$K_\mathsf{Arr_2}$, and $K_\mathsf{Arr_3}$ are consistent, the prover can use one of two methods. 

The most straight-forward method is to use the additive homomorphic property of the KZG polynomial commitment scheme which states that for equal-sized polynomials on the same domain:

* $K_\mathsf{Arr_1}\otimes K_\mathsf{Arr_2}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_1}\oplus \mathsf{Poly}_\mathsf{Arr_2})$

Here $\otimes$ is multiplication in the KZG group (*e.g.,* $\mathbb{G}_1$ in BLS12-381) while $\oplus$ is addition in $\mathbb{Z}_q$ of each evaluation point in $ \mathsf{Poly}_\mathsf{Arr_1}$ with each evaluation point in $\mathsf{Poly}_\mathsf{Arr_2}$. If the prover ($\mathcal{P}$) can set $K_\mathsf{Arr_3}\leftarrow K_\mathsf{Arr_1}\otimes K_\mathsf{Arr_2}$, then the verifier can check $K_\mathsf{Arr_3}\stackrel{?}{=}K_\mathsf{Arr_1}\otimes K_\mathsf{Arr_2}$.

However a second method is needed in other cases. Note that there are many different polynomials that interpolate $\mathsf{Arr_3}$ on the domain $\mathcal{H}_\kappa$ (but are different elsewhere in the polynomial outside of $\mathcal{H}_\kappa)$. Each of these polynomials will have a unique commitment value. So it is possible that $K_\mathsf{Arr_3}\neq K_\mathsf{Arr_1}\otimes K_\mathsf{Arr_2}$, and yet $\mathsf{Arr_3}[i]=\mathsf{Arr_1}[i]+\mathsf{Arr_2}[i]$ for all $i$. This arrises when $\mathsf{Poly}_\mathsf{Arr_3}$ comes from a different part of the protocol than $\mathsf{Poly}_\mathsf{Arr_1}$  and $\mathsf{Poly}_\mathsf{Arr_2}$.

The second method is more general so it can be used in place of the first method (but is more expensive), as well as covering all cases where $\mathsf{Arr_3}[i]=\mathsf{Arr_1}[i]+\mathsf{Arr_2}[i]$. The idea is to show $\mathsf{Arr_3}[i]-\mathsf{Arr_1}[i]+\mathsf{Arr_2}[i]=0$ for each evaluation point in the domain $\mathcal{H}_\kappa$. Showing a polynomial is zero on the domain (a "vanishing polynomial") is a common sub-protocol used by many gadgets.

 

## Protocol Details

### Array Level

* $\mathcal{P}$ holds an array $\mathsf{Arr_1} = [a_{(1,0)}, a_{(1,1)}, a_{(1,2)}, \dots, a_{(1,n-1)}]$ of $n$ integers ($a_{(1,i)} \in \mathbb{Z}_q$​)
* $\mathcal{P}$ holds an array $\mathsf{Arr_2} = [a_{(2,0)}, a_{(2,1)}, a_{(2,2)}, \dots, a_{(2,n-1)}]$ of $n$ integers ($a_{(2,i)} \in \mathbb{Z}_q$)
* $\mathcal{P}$ computes or holds an array $\mathsf{Arr_3} = [a_{(3,0)}, a_{(3,1)}, a_{(3,2)}, \dots, a_{(3,n-1)}]$ of $n$ integers ($a_{(3,i)} \in \mathbb{Z}_q$) such that:
  * $\mathsf{Arr_3}[i]=\mathsf{Arr_1}[i]+\mathsf{Arr_2}[i]$ for $i$ from 0 to $n-1$

### Polynomial Level

We assume the three arrays $\mathsf{Arr_1}$, $\mathsf{Arr_2}$ and $\mathsf{Arr_3}$ are encoded as the y-coordinates into a univariant polynomial where the x-coordinates (called the domain $\mathcal{H}_\kappa$) are chosen as the multiplicative group of order $\kappa$ with generator $\omega\in\mathbb{G}_\kappa$ (see [Background](../background/poly-iop.md) for more). In short, $\omega^0$ is the first element and $\omega^{\kappa-1}$ is the last element of $\mathcal{H}_\kappa$. If $\kappa$​ is larger than the length of the array, the array can be padded.

Recall the constraint we want to prove: 

1. $\mathsf{Arr_3}[i]=\mathsf{Arr_1}[i]+\mathsf{Arr_2}[i]$ for $i$ from 0 to $n-1$

In polynomial form, the constraint is:

1. For all $X$ from $\omega^0$ to $\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Arr_3}(X)=\mathsf{Poly}_\mathsf{Arr_1}(X)+\mathsf{Poly}_\mathsf{Arr_2}(X)$ 

We adjust the constraints to show an equality with 0 and label it:

1. For all $X$ from $\omega^0$ to $\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Vanish}(X)=\mathsf{Poly}_\mathsf{Arr_3}(X)-\mathsf{Poly}_\mathsf{Arr_1}(X)+\mathsf{Poly}_\mathsf{Arr_2}(X)=0$ 

This equation is true for every value of $X \in \mathcal{H}_\kappa$ (but not necessarily true outside of these values). To show this, we divide each polynomial by  $X^\kappa - 1$, which is a minimal vanishing polynomial for $\mathcal{H}_\kappa$ that does not require interpolation to create. If the quotients are polynomials (and not rational functions), then $\mathsf{Poly}_\mathsf{Vanish}(X)$ must be vanishing on $\mathcal{H}_\kappa$ too. Specifically, the prover computes:

1. $Q(X) = \frac{\mathsf{Poly}_\mathsf{Vanish}(X)}{X^\kappa - 1}$

By rearranging, we can get $\mathsf{Poly}_\mathsf{Zero}(X)$ as a true zero polynomial (zero at every value both in $\mathcal{H}_\kappa$ and outside of it):

1. $\mathsf{Poly}_\mathsf{Zero}(X)=\mathsf{Poly}_\mathsf{Vanish}(X) - Q(X)\cdot (X^n - 1)=0$​​​

Ultimately the add1 argument will satisfy the following constraints at the Commitment Level:

1. Show $Q(X)$​ exists (as a polynomial that evenly divides the divisor)
2. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is correctly constructed from $\mathsf{Poly}_\mathsf{Arr_1}(X)$, $\mathsf{Poly}_\mathsf{Arr_2}(X)$, and $\mathsf{Poly}_\mathsf{Arr_3}(X)$.
3. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is the zero polynomial

### Commitment Level

The verifier will never see the arrays or polynomials themselves. They are undisclosed because they either (i) contain private data or (ii) they are too large to examine and maintain a succinct proof system. Instead the prover will use commitments.

The prover will write the following commitments to the transcript:

* $K_\mathsf{Arr_1}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_1}(X))$​​
* $K_\mathsf{Arr_2}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_2}(X))$​
* $K_\mathsf{Arr_3}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_3}(X))$​

* $K_Q=\mathsf{KZG.Commit}(Q(X))$

The prover will generate a random challenge evaluation point (using strong Fiat-Shamir) on the polynomial that is outside of $\mathcal{H}_\kappa$. Call this point $\zeta$. The prover will write the three points and opening proofs to the transcript:

* $\zeta$
* $\mathsf{Poly}_\mathsf{Arr_1}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr_1},\zeta)$​
* $\mathsf{Poly}_\mathsf{Arr_2}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr_2},\zeta)$​
* $\mathsf{Poly}_\mathsf{Arr_3}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr_3},\zeta)$​

* $Q(\zeta)=\mathsf{KZG.Open}(K_Q,\zeta)$

To check the proof, the verifier uses the transcript to construct the value $Y_\mathsf{Zero}$ as follows:

* $Y_\mathsf{Vanish}=\mathsf{Poly}_\mathsf{Arr_1}(\zeta)-\mathsf{Poly}_\mathsf{Arr_2}(\zeta)+\mathsf{Poly}_\mathsf{Arr_3}( \zeta)$ 
* $Y_\mathsf{Zero}=Y_\mathsf{Vanish1} - Q(\zeta)\cdot (\zeta^n - 1)$​​

Finally, if the constraint system is true, the following constraint will be true (and will be false otherwise with overwhelming probability, due to the Schwartz-Zippel lemma on $\zeta$) :

* $Y_\mathsf{Zero}\overset{?}{=}0$

## Implementations

* [Rust](https://github.com/MadibaGroup/2024-Gadgets-Code/tree/main/src)
* [Mathematica](https://github.com/MadibaGroup/2024-Gadgets-Code/tree/main/Mathematica) (Toy Example)

## Security Proof

### Completeness

* 

### Soundness

- 

### Zero-Knowledge

- 

