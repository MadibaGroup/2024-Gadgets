# Addition (Type 2)

## Recap of types

| Type           | Description                                                  | Recap                                                        | This |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| [add1](./add1) | $\mathsf{Arr}_3=\mathsf{Arr}_1 + \mathsf{Arr}_2$             | $\mathsf{Arr}_3$ is the element-wise addition of $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$. |      |
| [add2](./add2) | $\mathsf{Sum}_\mathsf{Arr}=\sum_{i = 0}^{n-1} \mathsf{Arr}[i]$ | $\mathsf{Sum}_\mathsf{Arr}$ is the disclosed sum of all the elements in $\mathsf{Arr}$. | ✅    |
| [add3](./add3) | $\sum_{i = 0}^{n-1} \mathsf{Arr}_1[i]=\sum_{i = 0}^{n-1} \mathsf{Arr}_2[i]$ | $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$ have the same undisclosed sum. |      |

## Relation

$ \mathcal{R}_{\mathtt{add2}} := \left\{ \begin{array}{l} (K_\mathsf{Arr},\mathsf{Sum}_\mathsf{Arr}) \end{array} \middle | \begin{array}{l} \mathsf{Arr} = [a_0, a_1, a_2, \dots, a_{n-1}], \\ \mathsf{Sum}_\mathsf{Arr} = \sum_{i = 0}^{n-1} a_i, \\ \mathsf{Poly}_\mathsf{Arr}(X)=\mathsf{FFT.Interp}(\omega,\mathsf{Arr}), \\ K_\mathsf{Arr}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr}(X)) \end{array} \right\} $

## Intuition

The prover ($\mathcal{P}$) holds an array $\mathsf{Arr} = [a_0, a_1, a_2, \dots, a_{n-1}]$ of $n$ integers (from $\mathbb{Z}_q$) and a disclosed integer $\mathsf{Sum}_\mathsf{Arr}$. It will produce a succinct (independent of $n$) proof that $\mathsf{Sum}_\mathsf{Arr}$ is the sum of all the elements in the array. The prover will encode the array into a polynomial $\mathsf{Poly}_\mathsf{Arr}$ (using [evaluation points](../background/poly-iop) on the domain $\mathcal{H}_\kappa$) and commit to the polynomial $K_\mathsf{Arr}$. The verifier ($\mathcal{V}$) cannot check $\textsf{Arr}$ or $\mathsf{Poly}_\mathsf{Arr}$ directly (they may contain secret information, and even if they do not, it is too long to check) so the verifier only sees $K_\mathsf{Arr}$ and the asserted value $\mathsf{Sum_\mathsf{Arr}}$.

In order to prove $K_\mathsf{Arr}$ and  $\mathsf{Sum}_\mathsf{Arr}$ are consistent, the prover will build a helper array $\mathsf{Acc}_\mathsf{Arr}$ called an accumulator (or accumulating array or incremental array). This should not be confused with accumulators from cryptography, which are a concept related to succinct proofs but are distinct. As with $\mathsf{Arr}$, the prover will also encode $\mathsf{Acc}$ as a polynomial and provide a commitment of it to the verifier. The idea is that the prover will prove a relation between $\mathsf{Arr}$ and $\mathsf{Acc}$; and a relation between $\mathsf{Acc}$ and $\mathsf{Sum_\mathsf{Arr}}$. Put together, it will imply the correct relation between $\mathsf{Arr}$ and $\mathsf{Sum_\mathsf{Arr}}$.

Consider a small numeric example in $\mathbb{Z}_{97}$ where  $\mathsf{Arr}= [84,67,11,92,36,67]$ and $\mathsf{Sum}_\mathsf{Arr}=72$. The first idea is to get $\mathsf{Sum}_\mathsf{Arr}$ into an array. Say, we just append it: $\mathsf{Arr}''= [84,67,11,92,36,67,72] $. How does the prover show $\mathsf{Arr}''$ is correct? The last value of the array depends on every single element that precedes it, which will be a complex constraint to prove. 

An alternative idea is to create a new array that starts the same as $\mathsf{Arr}$ and ends up at $\mathsf{Sum}_\mathsf{Arr}$ by folding in the integers from $\mathsf{Arr}$ one-by-one with addition. Then each value in the new array will depend on only two values, as below. 

The first value in $\mathsf{Acc}$ will be a copy of the first value from $\mathsf{Arr}$:

* $\mathsf{Arr}= [84,67,11,92,36,67]$

* $\mathsf{Acc}= [84, \bot,\bot,\bot,\bot,\bot] $

The next value will be the addition (mod 97) of: 67 (the value at the same index in $\mathsf{Arr}$) and 84 (the previous value in $\mathsf{Acc}$):

* $\mathsf{Arr}= [84,67,11,92,36,67]$

* $ \mathsf{Acc} = [84, (67+84),\bot,\bot,\bot,\bot] = [84, 2,\bot,\bot,\bot,\bot]$ 

The next value will be the addition of: 11 (the value at the same index in $\mathsf{Arr}$) and 2 (the previous value in $\mathsf{Acc}$):

* $\mathsf{Arr}= [84,67,11,92,36,67]$
* $ \mathsf{Acc} = [84, 2,(11+2),\bot,\bot,\bot] = [84,2,22,\bot,\bot,\bot]$ 
* $ \mathsf{Acc} = [84, 2,22,(92+22),\bot,\bot] = [84,2,22,84,\bot,\bot]$ 
* $ \mathsf{Acc} = [84, 2,22,84,(36+84),\bot] = [84,2,22,84,17,\bot]$ 
* $ \mathsf{Acc} = [84,2,22,84,17,(67+17)] = [84, 2, 22, 84, 17, 72]$ 
* $\mathsf{Sum}_\mathsf{Arr}=72$

Notice the last value in $\mathsf{Acc}$ is $\mathsf{Sum_\mathsf{Arr}}$. The prover wants to show three constraints:

1. The first value in $\mathsf{Acc}$ matches the first value in $\mathsf{Arr}$,
2. The rest of the values in $\mathsf{Acc}$ are of the form $\mathsf{Acc}[i]=\mathsf{Arr}[i]+\mathsf{Acc}[i-1]$,
3. The last value in $\mathsf{Acc}$ matches $\mathsf{Sum}_\mathsf{Arr}$.

If all three constraints are true, then $\mathsf{Sum}_\mathsf{Arr}$ is the sum of the elements of $\mathsf{Arr}$. 

Last, while it is not necessary to do, it is often convenient to hold the the value $\mathsf{Sum}_\mathsf{Arr}$ at the start of the array $\mathsf{Acc}$ instead of the end. For this reason, the mathematical explaination below will construct $\mathsf{Acc}$ "backwards" (or right-to-left) from the above example, where the last value of $\mathsf{Acc}$ matches the last value of $\mathsf{Arr}$, the values are folded in from right to left, and the first (leftmost) value of $\mathsf{Acc}$ is $\mathsf{Sum}_\mathsf{Arr}$:

* $\mathsf{Arr}= [84,67,11,92,36,67]$
* $ \mathsf{Acc} = [\bot, \bot, \bot, \bot, \bot, 67]$ 
* $ \mathsf{Acc} = [\bot, \bot, \bot, \bot, 84, 67]$ 
* $\ldots$
* $ \mathsf{Acc} = [72, 84, 36, 65, 84, 67]$ 
* $\mathsf{Sum}_\mathsf{Arr}=72$

## Protocol Details

### Array Level

* $\mathcal{P}$ holds an array $\mathsf{Arr} = [a_0, a_1, a_2, \dots, a_{n-1}]$ of $n$ integers ($a_i \in \mathbb{Z}_q$)
* $\mathcal{P}$ computes array $\mathsf{Acc}$ as follows:
  * $\mathsf{Acc}[n-1]\leftarrow\mathsf{Arr}[n-1]$
  * $\mathsf{Acc}[i]\leftarrow\mathsf{Arr}[i]+\mathsf{Acc}[i+1]$ for $i$ from $n-2$ to 0
* $\mathcal{P}$ computes $\mathsf{Sum}_\mathsf{Arr}\leftarrow\mathsf{Acc}[0]$

### Polynomial Level

We assume arrays $\mathsf{Arr}$ and $\mathsf{Acc}$ are encoded as the y-coordinates into a univariant polynomial where the x-coordinates (called the domain $\mathcal{H}_\kappa$) are chosen as the multiplicative group of order $\kappa$ with generator $\omega\in\mathbb{G}_\kappa$ (see [Background](../background/poly-iop.md) for more). In short, $\omega^0$ is the first element and $\omega^{\kappa-1}$ is the last element of $\mathcal{H}_\kappa$. If $\kappa$ is larger than the length of the array, the array can be padded with elements of value 0 (which will not change the sum).

Recall the three constraints we want to prove: 

1. The first value in $\mathsf{Acc}$ matches the first value in $\mathsf{Arr}$,
2. The rest of the values in $\mathsf{Acc}$ are of the form $\mathsf{Acc}[i]=\mathsf{Arr}[i]+\mathsf{Acc}[i-1]$,
3. The last value in $\mathsf{Acc}$ matches $\mathsf{Sum}_\mathsf{Arr}$.

In polynomial form, the constraints are:

1. For $X=w^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc}(X)=\mathsf{Poly}_\mathsf{Arr}(X)$,
2. For all $X$ except $X=\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc}(X)=\mathsf{Poly}_\mathsf{Arr}(X)+\mathsf{Poly}_\mathsf{Acc}(\omega\cdot X)$ 
3. For $X=w^0$: $\mathsf{Poly}_\mathsf{Acc}(X)=\mathsf{Sum}_\mathsf{Arr}$

In constraint 2, $\mathsf{Poly}_\mathsf{Acc}(\omega\cdot X)$ can also be conceptualized as <span style="border-style:dotted;border-width: 2px;"> [rotate](./rotate)</span> applied to $\mathsf{Poly}_\mathsf{Acc}(X)$ by one element (rightward in the array view). Also note that constraint 2 does not hold at $X=\omega^{\kappa-1}$ because this value is defined by constraint 1 (for the last value of $X$, the "next" value, $\omega X$, wraps back to the first element of the array which is a boundary condition).

We adjust each of these constraints to show an equality with 0:

1. For $X=w^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc}(X)-\mathsf{Poly}_\mathsf{Arr}(X)=0$,
2. For all $X$ except $X=\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc}(X)-\mathsf{Poly}_\mathsf{Arr}(X)+\mathsf{Poly}_\mathsf{Acc}(\omega\cdot X)=0$ 
3. For $X=w^0$: $\mathsf{Poly}_\mathsf{Acc}(X)-\mathsf{Sum}_\mathsf{Arr}=0$

Next we take care of the "for $X$" conditions by zeroing out the rest of the polynomial that is not zero. See the gadget <span style="border-style:dotted;border-width: 2px;"> [zero1](./zero1)</span> for more on why this works.

1. $\mathsf{Poly}_\mathsf{Vanish1}(X)=(\mathsf{Poly}_\mathsf{Acc}(X)-\mathsf{Poly}_\mathsf{Arr}(X))\cdot\frac{(X^\kappa-1)}{(X-\omega^{\kappa-1})}=0$,
2. $\mathsf{Poly}_\mathsf{Vanish2}(X)=(\mathsf{Poly}_\mathsf{Acc}(X)-\mathsf{Poly}_\mathsf{Arr}(X)+\mathsf{Poly}_\mathsf{Acc}(\omega\cdot X))\cdot(X-\omega^{\kappa-1})=0$ 
3. $\mathsf{Poly}_\mathsf{Vanish3}(X)=(\mathsf{Poly}_\mathsf{Acc}(X)-\mathsf{Sum}_\mathsf{Arr})\cdot\frac{(X^\kappa-1)}{(X-\omega^0)}=0$

These equations are true for every value of $X \in \mathcal{H}_\kappa$ (but not necessarily true outside of these values). To show this, we divide each polynomial by  $X^\kappa - 1$, which is a minimal vanishing polynomial for $\mathcal{H}_\kappa$ that does not require interpolation to create. If the quotients are polynomials (and not rational functions), then $\mathsf{Poly}_\mathsf{Vanish1}(X)$, $\mathsf{Poly}_\mathsf{Vanish2}(X)$, and $\mathsf{Poly}_\mathsf{Vanish2}(X)$ must be vanishing on $\mathcal{H}_\kappa$ too. Specifically, the prove computes,

1. $Q_1(X) = \frac{\mathsf{Poly}_\mathsf{Vanish1}(X)}{X^\kappa - 1}$
2. $Q_2(X) = \frac{\mathsf{Poly}_\mathsf{Vanish2}(X)}{X^\kappa - 1}$
3. $Q_3(X) = \frac{\mathsf{Poly}_\mathsf{Vanish3}(X)}{X^\kappa - 1}$​​

We can replace polynomials $Q_1(X)$, $Q_2(X)$, and $Q_3(X)$ with a single polynomial $Q(X)$. We can do this because all three constraints have the same format: $\mathsf{Poly}_\mathsf{Vanish_i}(X)=0$. The batching technique is to create a new polynomial with all three $\mathsf{Poly}_\mathsf{Vanish_i}(X)$ values as coefficients. If and (overwhelmingly) only if all three are vanishing, then so will the new polynomial. This polynomial will be evaluated at a random challenge point $\rho$ selected after the commitments to the earlier polynomials are fixed. 

$Q(X) = \frac{\mathsf{Poly}_\mathsf{Vanish1}(X) + \mathsf{Poly}_\mathsf{Vanish2}(X) \rho  +  \mathsf{Poly}_\mathsf{Vanish3}(X)\rho^2}{X^n - 1}$

By rearranging, we can get $\mathsf{Poly}_\mathsf{Zero}(X)$ as a true zero polynomial (zero at every value both in $\mathcal{H}_\kappa$ and outside of it):

$\mathsf{Poly}_\mathsf{Zero}(X)=\mathsf{Poly}_\mathsf{Vanish1}(X) + \rho \mathsf{Poly}_\mathsf{Vanish2}(X) + \rho^2 \mathsf{Poly}_\mathsf{Vanish3}(X) - Q(X)\cdot (X^n - 1)=0$​​​

Ultimately the add2 argument will satisfy the following constraints at the Commitment Level:

1. Show $Q(X)$​ exists (as a polynomial that evenly divides the divisor)
2. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is correctly constructed from $\mathsf{Poly}_\mathsf{Acc}(X)$,  $\mathsf{Poly}_\mathsf{Acc}(\omega X)$, $\mathsf{Poly}_\mathsf{Arr}(X)$, and $\mathsf{Sum}_\mathsf{Arr}$
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
* $Y_\mathsf{Vanish2}=(\mathsf{Poly}_\mathsf{Acc}(\zeta)-\mathsf{Poly}_\mathsf{Arr}(\zeta)+\mathsf{Poly}_\mathsf{Acc}(\omega\cdot \zeta))\cdot(\zeta-\omega^{\kappa-1})$ 
* $Y_\mathsf{Vanish3}=(\mathsf{Poly}_\mathsf{Acc}(\zeta)-\mathsf{Sum}_\mathsf{Arr})\cdot\frac{(\zeta^\kappa-1)}{(\zeta-\omega^0)}$​
* $Y_\mathsf{Zero}=Y_\mathsf{Vanish1} + \rho Y_\mathsf{Vanish2} + \rho^2 Y_\mathsf{Vanish3} - Q(\zeta)\cdot (\zeta^n - 1)$​​

Finally, if the constraint system is true, the following constraint will be true (and will be false otherwise with overwhelming probability, due to the Schwartz-Zippel lemma on $\rho$ and $\zeta$) :

* $Y_\mathsf{Zero}\overset{?}{=}0$

## Implementations

* [Rust](https://github.com/MadibaGroup/2024-Gadgets-Code/tree/main/src)
* [Mathematica](https://github.com/MadibaGroup/2024-Gadgets-Code/tree/main/Mathematica) (Toy Example)

## Security Proof

### Completeness



### Soundness

- 

### Zero-Knowledge

- 