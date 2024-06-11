# Multiplication (Type 1)

## Recap of types

| Type                | Description                                                  | Recap                                                        | This |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| [mult1](./mult1.md) | $\mathsf{Arr}_3=\mathsf{Arr}_1 \cdot \mathsf{Arr}_2$         | $\mathsf{Arr}_3$ is the element-wise multiplication of $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$. | ✅    |
| [mult2](./mult2)    | $\mathsf{Prod}_\mathsf{Arr}=\prod_{i = 0}^{n-1} \mathsf{Arr}[i]$ | $\mathsf{Prod}_\mathsf{Arr}$ is the disclosed product of all the elements in $\mathsf{Arr}$. |      |
| [mult3](./mult3)    | $\prod_{i = 0}^{n-1} \mathsf{Arr}_1[i]=\prod_{i = 0}^{n-1} \mathsf{Arr}_2[i]$ | $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$ have the same undisclosed product. |      |

## Relation

$ \mathcal{R}_{\mathtt{mult2}} := \left\{ \begin{array}{l} (K_\mathsf{Arr_1},K_\mathsf{Arr_2},K_\mathsf{Arr_3}) \end{array} \middle | \begin{array}{l} \mathsf{Arr_3}[i]=\mathsf{Arr_1}[i]\cdot\mathsf{Arr_2}[i], 0\leq i \leq n-1, \\ \mathsf{Poly}_\mathsf{Arr_j}=\mathsf{FFT.Interp}(\omega,\mathsf{Arr_j}), 1\leq j \leq 3, \\ K_\mathsf{Arr_j}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_j}), 1\leq j \leq 3, \end{array} \right\} $

## Intuition

The prover ($\mathcal{P}$) holds two arrays $\mathsf{Arr_1}$ and $\mathsf{Arr_2}$ of $n$ integers from $\mathbb{Z}_q$: $[a_0, a_1, a_2, \dots, a_{n-1}]$. It will produce a succinct (independent of $n$) proof that $\mathsf{Arr_3}$ is the element-wise product of all the elements in the array: $\mathsf{Arr_3}[i]=\mathsf{Arr_1}[i]\cdot\mathsf{Arr_2}[i]$. The prover will encode the three arrays into three polynomials: $\mathsf{Poly}_\mathsf{Arr_1}$, $\mathsf{Poly}_\mathsf{Arr_2}$, and $\mathsf{Poly}_\mathsf{Arr_3}$ (using [evaluation points](../background/poly-iop) on the domain $\mathcal{H}_\kappa$). It will commit to each polynomial: $K_\mathsf{Arr_1}$,$K_\mathsf{Arr_2}$, and $K_\mathsf{Arr_3}$. The verifier ($\mathcal{V}$) cannot check any of the $\mathsf{Arr_i}$ or $\mathsf{Poly}_\mathsf{Arr_i}$ values directly (they may contain secret information, and even if they do not, they are too long to check) so the verifier only sees $K_\mathsf{Arr_1}$,$K_\mathsf{Arr_2}$, and $K_\mathsf{Arr_3}$.

In order to prove$K_\mathsf{Arr_1}$,$K_\mathsf{Arr_2}$, and $K_\mathsf{Arr_3}$ are consistent, the prover will compute the difference between $(\mathsf{Poly}_\mathsf{Arr_1}\cdot\mathsf{Poly}_\mathsf{Arr_2})$ and $(\mathsf{Poly}_\mathsf{Arr_3})$ using [add1](./add1). Next, it will show it is 0 for each evaluation point in the domain $\mathcal{H}_\kappa$.

## Protocol Details

### Array Level

* $\mathcal{P}$ holds an array $\mathsf{Arr_1} = [a_{(1,0)}, a_{(1,1)}, a_{(1,2)}, \dots, a_{(1,n-1)}]$ of $n$ integers ($a_{(1,i)} \in \mathbb{Z}_q$​)
* $\mathcal{P}$ holds an array $\mathsf{Arr_2} = [a_{(2,0)}, a_{(2,1)}, a_{(2,2)}, \dots, a_{(2,n-1)}]$ of $n$ integers ($a_{(2,i)} \in \mathbb{Z}_q$)
* $\mathcal{P}$ computes or holds an array $\mathsf{Arr_3} = [a_{(3,0)}, a_{(3,1)}, a_{(3,2)}, \dots, a_{(3,n-1)}]$ of $n$ integers ($a_{(3,i)} \in \mathbb{Z}_q$) such that:
  * $\mathsf{Arr_3}[i]=\mathsf{Arr_1}[i]\cdot\mathsf{Arr_2}[i]$ for $i$ from 0 to $n-1$

### Polynomial Level

We assume arrays $\mathsf{Arr}$ and $\mathsf{Acc}$ are encoded as the y-coordinates into a univariant polynomial where the x-coordinates (called the domain $\mathcal{H}_\kappa$) are chosen as the multiplicative group of order $\kappa$ with generator $\omega\in\mathbb{G}_\kappa$ (see [Background](../background/poly-iop.md) for more). In short, $\omega^0$ is the first element and $\omega^{\kappa-1}$ is the last element of $\mathcal{H}_\kappa$. If $\kappa$ is larger than the length of the array, the array can be padded with elements of value 1 (which will not change the product).

Recall the three constraints we want to prove: 

1. The first value in $\mathsf{Acc}$ matches the first value in $\mathsf{Arr}$,
2. The rest of the values in $\mathsf{Acc}$ are of the form $\mathsf{Acc}[i]=\mathsf{Arr}[i]\cdot\mathsf{Acc}[i-1]$,
3. The last value in $\mathsf{Acc}$ matches $\mathsf{Prod}_\mathsf{Arr}$.

In polynomial form, the constraints are:

1. For $X=w^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc}(X)=\mathsf{Poly}_\mathsf{Arr}(X)$,
2. For all $X$ except $X=\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc}(X)=\mathsf{Poly}_\mathsf{Arr}(X)\cdot\mathsf{Poly}_\mathsf{Acc}(\omega\cdot X)$ 
3. For $X=w^0$: $\mathsf{Poly}_\mathsf{Acc}(X)=\mathsf{Prod}_\mathsf{Arr}$

In constraint 2, $\mathsf{Poly}_\mathsf{Acc}(\omega\cdot X)$ can also be conceptualized as <span style="border-style:dotted;border-width: 2px;"> [rotate](./rotate)</span> applied to $\mathsf{Poly}_\mathsf{Acc}(X)$ by one element (rightward in the array view). Also note that constraint 2 does not hold at $X=\omega^{\kappa-1}$ because this value is defined by constraint 1 (for the last value of $X$, the "next" value, $\omega X$, wraps back to the first element of the array which is a boundary condition).

We adjust each of these constraints to show an equality with 0:

1. For $X=w^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc}(X)-\mathsf{Poly}_\mathsf{Arr}(X)=0$,
2. For all $X$ except $X=\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc}(X)-\mathsf{Poly}_\mathsf{Arr}(X)\cdot\mathsf{Poly}_\mathsf{Acc}(\omega\cdot X)=0$ 
3. For $X=w^0$: $\mathsf{Poly}_\mathsf{Acc}(X)-\mathsf{Prod}_\mathsf{Arr}=0$

Next we take care of the "for $X$" conditions by zeroing out the rest of the polynomial that is not zero. See the gadget <span style="border-style:dotted;border-width: 2px;"> [zero1](./zero1)</span> for more on why this works.

1. $\mathsf{Poly}_\mathsf{Vanish1}(X)=(\mathsf{Poly}_\mathsf{Acc}(X)-\mathsf{Poly}_\mathsf{Arr}(X))\cdot\frac{(X^\kappa-1)}{(X-\omega^{\kappa-1})}=0$,
2. $\mathsf{Poly}_\mathsf{Vanish2}(X)=(\mathsf{Poly}_\mathsf{Acc}(X)-\mathsf{Poly}_\mathsf{Arr}(X)\cdot\mathsf{Poly}_\mathsf{Acc}(\omega\cdot X))\cdot(X-\omega^{\kappa-1})=0$ 
3. $\mathsf{Poly}_\mathsf{Vanish3}(X)=(\mathsf{Poly}_\mathsf{Acc}(X)-\mathsf{Prod}_\mathsf{Arr})\cdot\frac{(X^\kappa-1)}{(X-\omega^0)}=0$

These equations are true for every value of $X \in \mathcal{H}_\kappa$ (but not necessarily true outside of these values). To show this, we divide each polynomial by  $X^\kappa - 1$, which is a minimal vanishing polynomial for $\mathcal{H}_\kappa$ that does not require interpolation to create. If the quotients are polynomials (and not rational functions), then $\mathsf{Poly}_\mathsf{Vanish1}(X)$, $\mathsf{Poly}_\mathsf{Vanish2}(X)$, and $\mathsf{Poly}_\mathsf{Vanish2}(X)$ must be vanishing on $\mathcal{H}_\kappa$ too. Specifically, the prove computes,

1. $Q_1(X) = \frac{\mathsf{Poly}_\mathsf{Vanish1}(X)}{X^\kappa - 1}$
2. $Q_2(X) = \frac{\mathsf{Poly}_\mathsf{Vanish2}(X)}{X^\kappa - 1}$
3. $Q_3(X) = \frac{\mathsf{Poly}_\mathsf{Vanish3}(X)}{X^\kappa - 1}$​​

We can replace polynomials $Q_1(X)$, $Q_2(X)$, and $Q_3(X)$ with a single polynomial $Q(X)$. We can do this because all three constraints have the same format: $\mathsf{Poly}_\mathsf{Vanish_i}(X)=0$. The batching technique is to create a new polynomial with all three $\mathsf{Poly}_\mathsf{Vanish_i}(X)$ values as coefficients. If and (overwhelmingly) only if all three are vanishing, then so will the new polynomial. This polynomial will be evaluated at a random challenge point $\rho$ selected after the commitments to the earlier polynomials are fixed. 

$Q(X) = \frac{\mathsf{Poly}_\mathsf{Vanish1}(X) + \mathsf{Poly}_\mathsf{Vanish2}(X) \rho  +  \mathsf{Poly}_\mathsf{Vanish3}(X)\rho^2}{X^n - 1}$

By rearranging, we can get $\mathsf{Poly}_\mathsf{Zero}(X)$ as a true zero polynomial (zero at every value both in $\mathcal{H}_\kappa$ and outside of it):

$\mathsf{Poly}_\mathsf{Zero}(X)=\mathsf{Poly}_\mathsf{Vanish1}(X) + \rho \mathsf{Poly}_\mathsf{Vanish2}(X) + \rho^2 \mathsf{Poly}_\mathsf{Vanish3}(X) - Q(X)\cdot (X^n - 1)=0$​​​

Ultimately the mult2 argument will satisfy the following constraints at the Commitment Level:

1. Show $Q(X)$​ exists (as a polynomial that evenly divides the divisor)
2. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is correctly constructed from $\mathsf{Poly}_\mathsf{Acc}(X)$,  $\mathsf{Poly}_\mathsf{Acc}(\omega X)$, $\mathsf{Poly}_\mathsf{Arr}(X)$, and $\mathsf{Prod}_\mathsf{Arr}$
3. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is the zero polynomial

### Commitment Level

The verifier will never see the arrays or polynomials themselves. They are undisclosed because they either (i) contain private data or (ii) they are too large to examine and maintain a succinct proof system. Instead the prover will use commitments.

The prover will write the following commitments to the transcript:

* $K_\mathsf{Arr}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr}(X))$​
* $K_\mathsf{Acc}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Acc}(X))$​​

The prover will generate a random challenge evaluation point (using strong Fiat-Shamir) on the polynomial that is outside of $\mathcal{H}_\kappa$. Call this point $\rho$. It will be used by the prover to create polynomial $Q(X)$ (see above) and the prover will write to the transcript: 

* $\rho$
* $K_Q=\mathsf{KZG.Commit}(Q(X))$

The prover will generate a second random challenge evaluation point (using strong Fiat-Shamir) on the polynomial that is outside of $\mathcal{H}_\kappa$. Call this point $\zeta$. The prover will write the three points and opening proofs to the transcript:

* $\zeta$
* $\mathsf{Poly}_\mathsf{Arr}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr},\zeta)$​
* $\mathsf{Poly}_\mathsf{Arr}(\zeta\omega)=\mathsf{KZG.Open}(K_\mathsf{Arr},\zeta\omega)$
* $\mathsf{Poly}_\mathsf{Acc}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Acc},\zeta)$​
* $Q(\zeta)=\mathsf{KZG.Open}(K_Q,\zeta)$

To check the proof, the verifier uses the transcript to construct the value $Y_\mathsf{Zero}$ as follows:

* $Y_\mathsf{Vanish1}=(\mathsf{Poly}_\mathsf{Acc}(\zeta)-\mathsf{Poly}_\mathsf{Arr}(\zeta))\cdot\frac{(\zeta^\kappa-1)}{(\zeta-\omega^{\kappa-1})}$
* $Y_\mathsf{Vanish2}=(\mathsf{Poly}_\mathsf{Acc}(\zeta)-\mathsf{Poly}_\mathsf{Arr}(\zeta)\cdot\mathsf{Poly}_\mathsf{Acc}(\omega\cdot \zeta))\cdot(\zeta-\omega^{\kappa-1})$ 
* $Y_\mathsf{Vanish3}=(\mathsf{Poly}_\mathsf{Acc}(\zeta)-\mathsf{Prod}_\mathsf{Arr})\cdot\frac{(\zeta^\kappa-1)}{(\zeta-\omega^0)}$​
* $Y_\mathsf{Zero}=Y_\mathsf{Vanish1} + \rho Y_\mathsf{Vanish2} + \rho^2 Y_\mathsf{Vanish3} - Q(\zeta)\cdot (\zeta^n - 1)$​​

Finally, if the constraint system is true, the following constraint will be true (and will be false otherwise with overwhelming probability, due to the Schwartz-Zippel lemma on $\rho$ and $\zeta$) :

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
