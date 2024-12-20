# Addition (Type 2)

## Recap of types

| Type           | Description                                                  | Recap                                                        | This |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| [add1](../add1) | $\mathsf{Arr}_3=\mathsf{Arr}_1 + \mathsf{Arr}_2$             | $\mathsf{Arr}_3$ is the element-wise addition of $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$. |      |
| [add2](#) | $\mathsf{Sum}_\mathsf{Arr}=\sum_{i = 0}^{n-1} \mathsf{Arr}[i]$ | $\mathsf{Sum}_\mathsf{Arr}$ is the disclosed sum of all the elements in $\mathsf{Arr}$. | ✅    |
| [add3](../add3) | $\sum_{i = 0}^{n-1} \mathsf{Arr}_1[i]=\sum_{i = 0}^{n-1} \mathsf{Arr}_2[i]$ | $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$ have the same undisclosed sum. |      |

## Relation

$ \mathcal{R}_{\mathtt{add2}} := \left\{ \begin{array}{l} (K_\mathsf{Arr},\mathsf{Sum}_\mathsf{Arr}) \end{array} \middle | \begin{array}{l} \mathsf{Arr} = [a_0, a_1, a_2, \dots, a_{n-1}], \\ \mathsf{Sum}_\mathsf{Arr} = \sum_{i = 0}^{n-1} a_i, \\ \mathsf{Poly}_\mathsf{Arr}(X)=\mathsf{FFT.Interp}(\omega,\mathsf{Arr}), \\ K_\mathsf{Arr}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr}(X)) \end{array} \right\} $

## Intuition

The prover ($\mathcal{P}$) holds an array $\mathsf{Arr} = [a_0, a_1, a_2, \dots, a_{n-1}]$ of $n$ integers (from $\mathbb{Z}_q$) and a disclosed integer $\mathsf{Sum}_\mathsf{Arr}$. It will produce a succinct (independent of $n$) proof that $\mathsf{Sum}_\mathsf{Arr}$ is the sum of all the elements in the array. The prover will encode the array into a polynomial $\mathsf{Poly}_\mathsf{Arr}$ (using [evaluation points](../../background/poly-iop) on the domain $\mathcal{H}_\kappa$) and commit to the polynomial $K_\mathsf{Arr}$. The verifier ($\mathcal{V}$) cannot check $\textsf{Arr}$ or $\mathsf{Poly}_\mathsf{Arr}$ directly (they may contain secret information, and even if they do not, it is too long to check) so the verifier only sees $K_\mathsf{Arr}$ and the asserted value $\mathsf{Sum_\mathsf{Arr}}$.

In order to prove $K_\mathsf{Arr}$ and  $\mathsf{Sum}_\mathsf{Arr}$ are consistent, the prover will build a helper array $\mathsf{Acc}_\mathsf{Arr}$ called an accumulator (or accumulating array or incremental array). This should not be confused with accumulators from cryptography, which are a concept related to succinct proofs but are distinct. As with $\mathsf{Arr}$, the prover will also encode $\mathsf{Acc}$ as a polynomial and provide a commitment of it to the verifier. The idea is that the prover will prove a relation between $\mathsf{Arr}$ and $\mathsf{Acc}$; and a relation between $\mathsf{Acc}$ and $\mathsf{Sum_\mathsf{Arr}}$. Put together, it will imply the correct relation between $\mathsf{Arr}$ and $\mathsf{Sum_\mathsf{Arr}}$.

Consider a small numeric example in $\mathbb{Z}_{97}$ where  $\mathsf{Arr}= [84,67,11,92,36,67]$ and $\mathsf{Sum}_\mathsf{Arr}=66$. The first idea is to get $\mathsf{Sum}_\mathsf{Arr}$ into an array. Say, we just append it: $\mathsf{Arr}''= [84,67,11,92,36,67,66] $. How does the prover show $\mathsf{Arr}''$ is correct? The last value of the array depends on every single element that precedes it, which will be a complex constraint to prove. 

An alternative idea is to create a new array that starts the same as $\mathsf{Arr}$ and ends up at $\mathsf{Sum}_\mathsf{Arr}$ by folding in the integers from $\mathsf{Arr}$ one-by-one with addition. Then each value in the new array will depend on only two values, as below. 

The first value in $\mathsf{Acc}$ will be a copy of the first value from $\mathsf{Arr}$:

* $\mathsf{Arr}= [84,67,11,92,36,67]$

* $\mathsf{Acc}= [84, \bot,\bot,\bot,\bot,\bot] $

The next value will be the addition (mod 97) of: 67 (the value at the same index in $\mathsf{Arr}$) and 84 (the previous value in $\mathsf{Acc}$):

* $\mathsf{Arr}= [84,67,11,92,36,67]$

* $ \mathsf{Acc} = [84, (67+84),\bot,\bot,\bot,\bot] = [84, 54,\bot,\bot,\bot,\bot]$ 

The next value will be the addition of: 11 (the value at the same index in $\mathsf{Arr}$) and 54 (the previous value in $\mathsf{Acc}$):

* $\mathsf{Arr}= [84,67,11,92,36,67]$
* $ \mathsf{Acc} = [84, 54,(11+54),\bot,\bot,\bot] = [84,54,65,\bot,\bot,\bot]$ 
* $ \mathsf{Acc} = [84, 54, 65,(92+65),\bot,\bot] = [84,54,65, 60,\bot,\bot]$ 
* $ \mathsf{Acc} = [84,54,65, 60,(36 + 60),\bot] = [84,54,65, 60, 96,\bot]$ 
* $ \mathsf{Acc} = [84,54,65, 60, 96, (67+96)] = [84,54,65, 60, 96, 66]$ 
* $\mathsf{Sum}_\mathsf{Arr}=66$

Notice the last value in $\mathsf{Acc}$ is $\mathsf{Sum_\mathsf{Arr}}$. The prover wants to show three constraints:

1. The first value in $\mathsf{Acc}$ matches the first value in $\mathsf{Arr}$,
2. The rest of the values in $\mathsf{Acc}$ are of the form $\mathsf{Acc}[i]=\mathsf{Arr}[i]+\mathsf{Acc}[i-1]$,
3. The last value in $\mathsf{Acc}$ matches $\mathsf{Sum}_\mathsf{Arr}$.

If all three constraints are true, then $\mathsf{Sum}_\mathsf{Arr}$ is the sum of the elements of $\mathsf{Arr}$. 

Last, while it is not necessary to do, it is often convenient to hold the the value $\mathsf{Sum}_\mathsf{Arr}$ at the start of the array $\mathsf{Acc}$ instead of the end. For this reason, the mathematical explanation below will construct $\mathsf{Acc}$ "backwards" (or right-to-left) from the above example, where the last value of $\mathsf{Acc}$ matches the last value of $\mathsf{Arr}$, the values are folded in from right to left, and the first (leftmost) value of $\mathsf{Acc}$ is $\mathsf{Sum}_\mathsf{Arr}$:

* $\mathsf{Arr}= [84,67,11,92,36,67]$
* $ \mathsf{Acc} = [\bot, \bot, \bot, \bot, \bot, 67]$ 
* $ \mathsf{Acc} = [\bot, \bot, \bot, \bot, 6, 67]$ 
* $\ldots$
* $ \mathsf{Acc} = [66, 79, 12, 1, 6, 67]$ 
* $\mathsf{Sum}_\mathsf{Arr}=66$

## Protocol Details

### Array Level

* $\mathcal{P}$ holds an array $\mathsf{Arr} = [a_0, a_1, a_2, \dots, a_{n-1}]$ of $n$ integers ($a_i \in \mathbb{Z}_q$)
* $\mathcal{P}$ computes array $\mathsf{Acc}$ as follows:
  * $\mathsf{Acc}[n-1]\leftarrow\mathsf{Arr}[n-1]$
  * $\mathsf{Acc}[i]\leftarrow\mathsf{Arr}[i]+\mathsf{Acc}[i+1]$ for $i$ from $n-2$ to 0
* $\mathcal{P}$ computes $\mathsf{Sum}_\mathsf{Arr}\leftarrow\mathsf{Acc}[0]$

### Polynomial Level

We assume arrays $\mathsf{Arr}$ and $\mathsf{Acc}$ are encoded as the y-coordinates into a univariant polynomial where the x-coordinates (called the domain $\mathcal{H}_\kappa$) are chosen as the multiplicative group of order $\kappa$ with generator $\omega\in\mathbb{G}_\kappa$ (see [Background](../../background/poly-iop) for more). In short, $\omega^0$ is the first element and $\omega^{\kappa-1}$ is the last element of $\mathcal{H}_\kappa$. If $\kappa$ is larger than the length of the array, the array can be padded with elements of value 0 (which will not change the sum).

Recall the three constraints we want to prove (now adjusted to fit with an $\mathsf{Acc}$ that is constructed "backwards," as noted above): 

1. The last value in $\mathsf{Acc}$ matches the last value in $\mathsf{Arr}$,
2. The rest of the values in $\mathsf{Acc}$ are of the form $\mathsf{Acc}[i]=\mathsf{Arr}[i]+\mathsf{Acc}[i-1]$,
3. The first value in $\mathsf{Acc}$ matches $\mathsf{Sum}_\mathsf{Arr}$.

In polynomial form, the constraints are:

1. For $X=w^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc}(X)=\mathsf{Poly}_\mathsf{Arr}(X)$,
2. For all $X$ except $X=\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc}(X)=\mathsf{Poly}_\mathsf{Arr}(X)+\mathsf{Poly}_\mathsf{Acc}(\omega\cdot X)$ 
3. For $X=w^0$: $\mathsf{Poly}_\mathsf{Acc}(X)=\mathsf{Sum}_\mathsf{Arr}$

In constraint 2, $\mathsf{Poly}_\mathsf{Acc}(\omega\cdot X)$ can also be conceptualized as <span style="border-style:dotted;border-width: 2px;"> [rotate](../rotate)</span> applied to $\mathsf{Poly}_\mathsf{Acc}(X)$ by one element (rightward in the array view). Also note that constraint 2 does not hold at $X=\omega^{\kappa-1}$ because this value is defined by constraint 1 (for the last value of $X$, the "next" value, $\omega X$, wraps back to the first element of the array which is a boundary condition).

We adjust each of these constraints to show an equality with 0:

1. For $X=w^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc}(X)-\mathsf{Poly}_\mathsf{Arr}(X)=0$,
2. For all $X$ except $X=\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc}(X)-\mathsf{Poly}_\mathsf{Arr}(X)+\mathsf{Poly}_\mathsf{Acc}(\omega\cdot X)=0$ 
3. For $X=w^0$: $\mathsf{Poly}_\mathsf{Acc}(X)-\mathsf{Sum}_\mathsf{Arr}=0$

Next we take care of the "for $X$" conditions by zeroing out the rest of the polynomial that is not zero. See the gadget <span style="border-style:dotted;border-width: 2px;"> [zero1](../zero1)</span> for more on why this works.

1. $\mathsf{Poly}_\mathsf{Vanish1}(X)=(\mathsf{Poly}_\mathsf{Acc}(X)-\mathsf{Poly}_\mathsf{Arr}(X))\cdot\frac{(X^\kappa-1)}{(X-\omega^{\kappa-1})}=0$,
2. $\mathsf{Poly}_\mathsf{Vanish2}(X)=(\mathsf{Poly}_\mathsf{Acc}(X)-\mathsf{Poly}_\mathsf{Arr}(X)+\mathsf{Poly}_\mathsf{Acc}(\omega\cdot X))\cdot(X-\omega^{\kappa-1})=0$ 
3. $\mathsf{Poly}_\mathsf{Vanish3}(X)=(\mathsf{Poly}_\mathsf{Acc}(X)-\mathsf{Sum}_\mathsf{Arr})\cdot\frac{(X^\kappa-1)}{(X-\omega^0)}=0$

These equations are true for every value of $X \in \mathcal{H}_\kappa$ (but not necessarily true outside of these values). To show this, we divide each polynomial by  $X^\kappa - 1$, which is a minimal vanishing polynomial for $\mathcal{H}_\kappa$ that does not require interpolation to create. If the quotients are polynomials (and not rational functions), then $\mathsf{Poly}_\mathsf{Vanish1}(X)$, $\mathsf{Poly}_\mathsf{Vanish2}(X)$, and $\mathsf{Poly}_\mathsf{Vanish3}(X)$ must be vanishing on $\mathcal{H}_\kappa$ too. Specifically, the prove computes,

1. $Q_1(X) = \frac{\mathsf{Poly}_\mathsf{Vanish1}(X)}{X^\kappa - 1}$
2. $Q_2(X) = \frac{\mathsf{Poly}_\mathsf{Vanish2}(X)}{X^\kappa - 1}$
3. $Q_3(X) = \frac{\mathsf{Poly}_\mathsf{Vanish3}(X)}{X^\kappa - 1}$

We can replace polynomials $Q_1(X)$, $Q_2(X)$, and $Q_3(X)$ with a single polynomial $Q(X)$. We can do this because all three constraints have the same format: $\mathsf{Poly}_\mathsf{Vanish_i}(X)=0$. The batching technique is to create a new polynomial with all three $\mathsf{Poly}_\mathsf{Vanish_i}(X)$ values as coefficients. If and (overwhelmingly) only if all three are vanishing, then so will the new polynomial. This polynomial will be evaluated at a random challenge point $\rho$ selected after the commitments to the earlier polynomials are fixed. 

$Q(X) = \frac{\mathsf{Poly}_\mathsf{Vanish1}(X) + \mathsf{Poly}_\mathsf{Vanish2}(X) \rho  +  \mathsf{Poly}_\mathsf{Vanish3}(X)\rho^2}{X^\kappa - 1}$

By rearranging, we can get $\mathsf{Poly}_\mathsf{Zero}(X)$ as a true zero polynomial (zero at every value both in $\mathcal{H}_\kappa$ and outside of it):

$\mathsf{Poly}_\mathsf{Zero}(X)=\mathsf{Poly}_\mathsf{Vanish1}(X) + \rho \mathsf{Poly}_\mathsf{Vanish2}(X) + \rho^2 \mathsf{Poly}_\mathsf{Vanish3}(X) - Q(X)\cdot (X^\kappa - 1)=0$

Ultimately the add2 argument will satisfy the following constraints at the Commitment Level:

1. Show $Q(X)$ exists (as a polynomial that evenly divides the divisor)
2. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is correctly constructed from $\mathsf{Poly}_\mathsf{Acc}(X)$,  $\mathsf{Poly}_\mathsf{Acc}(\omega X)$, $\mathsf{Poly}_\mathsf{Arr}(X)$, and $\mathsf{Sum}_\mathsf{Arr}$
3. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is the zero polynomial

### Commitment Level

The verifier will never see the arrays or polynomials themselves. They are undisclosed because they either (i) contain private data or (ii) they are too large to examine and maintain a succinct proof system. Instead the prover will use commitments.

The prover will write the following commitments to the transcript:

* $K_\mathsf{Arr}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr}(X))$
* $K_\mathsf{Acc}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Acc}(X))$

The prover will generate a random challenge evaluation point (using strong Fiat-Shamir) on the polynomial that is outside of $\mathcal{H}_\kappa$. Call this point $\rho$. It will be used by the prover to create polynomial $Q(X)$ (see above) and the prover will write to the transcript: 

* $\rho$
* $K_Q=\mathsf{KZG.Commit}(Q(X))$

The prover will generate a second random challenge evaluation point (using strong Fiat-Shamir) on the polynomial that is outside of $\mathcal{H}_\kappa$. Call this point $\zeta$. The prover will write the point and opening proofs to the transcript:

* $\zeta$
* $\mathsf{Poly}_\mathsf{Arr}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr},\zeta)$
* $\mathsf{Poly}_\mathsf{Arr}(\zeta\omega)=\mathsf{KZG.Open}(K_\mathsf{Arr},\zeta\omega)$
* $\mathsf{Poly}_\mathsf{Acc}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Acc},\zeta)$
* $Q(\zeta)=\mathsf{KZG.Open}(K_Q,\zeta)$

To check the proof, the verifier uses the transcript to construct the value $Y_\mathsf{Zero}$ as follows:

* $Y_\mathsf{Vanish1}=(\mathsf{Poly}_\mathsf{Acc}(\zeta)-\mathsf{Poly}_\mathsf{Arr}(\zeta))\cdot\frac{(\zeta^\kappa-1)}{(\zeta-\omega^{\kappa-1})}$
* $Y_\mathsf{Vanish2}=(\mathsf{Poly}_\mathsf{Acc}(\zeta)-\mathsf{Poly}_\mathsf{Arr}(\zeta)+\mathsf{Poly}_\mathsf{Acc}(\omega\cdot \zeta))\cdot(\zeta-\omega^{\kappa-1})$ 
* $Y_\mathsf{Vanish3}=(\mathsf{Poly}_\mathsf{Acc}(\zeta)-\mathsf{Sum}_\mathsf{Arr})\cdot\frac{(\zeta^\kappa-1)}{(\zeta-\omega^0)}$
* $Y_\mathsf{Zero}=Y_\mathsf{Vanish1} + \rho Y_\mathsf{Vanish2} + \rho^2 Y_\mathsf{Vanish3} - Q(\zeta)\cdot (\zeta^\kappa - 1)$

Finally, if the constraint system is true, the following constraint will be true (and will be false otherwise with overwhelming probability, due to the Schwartz-Zippel lemma on $\rho$ and $\zeta$) :

* $Y_\mathsf{Zero}\overset{?}{=}0$

## Implementations

* [Rust](https://github.com/MadibaGroup/2024-Gadgets-Code/tree/main/src)
* [Mathematica](https://github.com/MadibaGroup/2024-Gadgets-Code/tree/main/Mathematica) (Toy Example)

## Security Proof

### Completeness

If $Y_\mathsf{Zero}$ is zero, then $\mathcal{V}$ will accept. Therefore, to show completeness, we show that any prover who holds $\mathsf{Arr}$ such that $\mathsf{Sum}_\mathsf{Arr}=\sum_{i = 0}^{n-1} \mathsf{Arr}[i] \space \forall i \in [0, n - 1]$ can follow the steps outlined in the above protocol and the resulting $Y_\mathsf{Zero}$ will be equal to zero.  To see this, observed that $Y_\mathsf{Zero}$

$= Y_\mathsf{Vanish1} + \rho Y_\mathsf{Vanish2} + \rho^2 Y_\mathsf{Vanish3} - Q(\zeta)\cdot (\zeta^\kappa - 1)$ 

$= (\mathsf{Poly}_\mathsf{Acc}(\zeta)-\mathsf{Poly}_\mathsf{Arr}(\zeta))\cdot\frac{(\zeta^\kappa-1)}{(\zeta-\omega^{\kappa-1})}) + \rho(\mathsf{Poly}_\mathsf{Acc}(\zeta)-\mathsf{Poly}_\mathsf{Arr}(\zeta)+\mathsf{Poly}_\mathsf{Acc}(\omega\cdot \zeta))\cdot(\zeta-\omega^{\kappa-1}) + \rho^2 (\mathsf{Poly}_\mathsf{Acc}(\zeta)-\mathsf{Sum}_\mathsf{Arr})\cdot\frac{(\zeta^\kappa-1)}{(\zeta-\omega^0)} - Q(\zeta)\cdot (\zeta^\kappa - 1)$

$= (\mathsf{Poly}_\mathsf{Acc}(\zeta)-\mathsf{Poly}_\mathsf{Arr}(\zeta))\cdot\frac{(\zeta^\kappa-1)}{(\zeta-\omega^{\kappa-1})}) + \rho(\mathsf{Poly}_\mathsf{Acc}(\zeta)-\mathsf{Poly}_\mathsf{Arr}(\zeta)+\mathsf{Poly}_\mathsf{Acc}(\omega\cdot \zeta))\cdot(\zeta-\omega^{\kappa-1}) + \rho^2 (\mathsf{Poly}_\mathsf{Acc}(\zeta)-\mathsf{Sum}_\mathsf{Arr})\cdot\frac{(\zeta^\kappa-1)}{(\zeta-\omega^0)} \newline - \frac{\mathsf{Poly}_\mathsf{Vanish1}(\zeta) + \mathsf{Poly}_\mathsf{Vanish2}(\zeta) \rho  +  \mathsf{Poly}_\mathsf{Vanish3}(\zeta)\rho^2}{\zeta^\kappa - 1} \cdot(\zeta^\kappa - 1)$

$= (\mathsf{Poly}_\mathsf{Acc}(\zeta)-\mathsf{Poly}_\mathsf{Arr}(\zeta))\cdot\frac{(\zeta^\kappa-1)}{(\zeta-\omega^{\kappa-1})}) + \rho(\mathsf{Poly}_\mathsf{Acc}(\zeta)-\mathsf{Poly}_\mathsf{Arr}(\zeta)+\mathsf{Poly}_\mathsf{Acc}(\omega\cdot \zeta))\cdot(\zeta-\omega^{\kappa-1}) + \rho^2 (\mathsf{Poly}_\mathsf{Acc}(\zeta)-\mathsf{Sum}_\mathsf{Arr})\cdot\frac{(\zeta^\kappa-1)}{(\zeta-\omega^0)} \newline - ((\mathsf{Poly}_\mathsf{Acc}(\zeta)-\mathsf{Poly}_\mathsf{Arr}(\zeta))\cdot\frac{(\zeta^\kappa-1)}{(\zeta-\omega^{\kappa-1})}) + \rho(\mathsf{Poly}_\mathsf{Acc}(\zeta)-\mathsf{Poly}_\mathsf{Arr}(\zeta)+\mathsf{Poly}_\mathsf{Acc}(\omega\cdot \zeta))\cdot(\zeta-\omega^{\kappa-1}) \newline + \rho^2 (\mathsf{Poly}_\mathsf{Acc}(\zeta)-\mathsf{Sum}_\mathsf{Arr})\cdot\frac{(\zeta^\kappa-1)}{(\zeta-\omega^0)})$ 

$= 0$

Where the third equality relies on the fact that $\mathsf{Poly}_\mathsf{Vanish1}(\zeta) + \mathsf{Poly}_\mathsf{Vanish2}(\zeta) \rho  +  \mathsf{Poly}_\mathsf{Vanish3}(\zeta)\rho^2$ is divisible by $X^\kappa - 1$. This is true if $\mathsf{Poly_{Vanish_1}}(X), \mathsf{Poly_{Vanish_2}}(X)$ and $\mathsf{Poly_{Vanish_3}}(X),$ are all vanishing on $\mathcal{H_\kappa}$, i.e. if all three of the following conditions hold for all $X \in \mathcal{H}_\kappa$:

1. $(\mathsf{Poly}_\mathsf{Acc}(X)-\mathsf{Poly}_\mathsf{Arr}(X))\cdot\frac{(X^\kappa-1)}{(X-\omega^{\kappa-1})}=0$
2. $ (\mathsf{Poly}_\mathsf{Acc}(X)-\mathsf{Poly}_\mathsf{Arr}(X)+\mathsf{Poly}_\mathsf{Acc}(\omega\cdot X))\cdot(X-\omega^{\kappa-1})=0$ 
3. $ (\mathsf{Poly}_\mathsf{Acc}(X)-\mathsf{Sum}_\mathsf{Arr})\cdot\frac{(X^\kappa-1)}{(X-\omega^0)}=0$

These conditions, in turn, hold if:

1. For $X=w^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc}(X)=\mathsf{Poly}_\mathsf{Arr}(X)$
2. For all $X$ except $X=\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc}(X)=\mathsf{Poly}_\mathsf{Arr}(X)+\mathsf{Poly}_\mathsf{Acc}(\omega\cdot X)$ 
3. For $X=w^0$: $\mathsf{Poly}_\mathsf{Acc}(X)=\mathsf{Sum}_\mathsf{Arr}$

Where we get the "For $X$" due to zeroing parts of the polynomials (see [zero1](../zero1.md)). Since $\mathsf{Poly_j}(\omega^i) = \mathsf{Arr_j}[i] \space \forall i \in [0, \kappa - 1]$, the above conditions are true if:

1. The last value in $\mathsf{Acc}$ matches the last value in $\mathsf{Arr}$
2. The rest of the values in $\mathsf{Acc}$ are of the form $\mathsf{Acc}[i]=\mathsf{Arr}[i]+\mathsf{Acc}[i-1]$
3. The first value in $\mathsf{Acc}$ matches $\mathsf{Sum}_\mathsf{Arr}$

Which are precisely the conditions the Intuitions sections explains will hold if the prover contructs their Accumulator by following the protocol and using $\mathsf{Arr}$ such that $\mathsf{Sum}_\mathsf{Arr}=\sum_{i = 0}^{n-1} \mathsf{Arr}[i] \space \forall i \in [0, n - 1]$. This is what we assumed true about the prover, thus the $Y_\mathsf{Zero}$ it creates by following the protocol is zero, and its transcript will be accepted.

### Soundness

We prove knowledge soundness in the Algebraic Group Model (AGM). To do so, we must prove that there exists an efficient extractor $\mathcal{E}$ such that for any algebraic adversary $\mathcal{A}$ the probability of $\mathcal{A}$ winning the following game is $\mathsf{negl}(\lambda)$.

1. Given $[g, g^\tau, g^{\tau^2}, \dots,g^{\tau^{n-1}}]$ $\mathcal{A}$ outputs commitments to $\mathsf{Poly}_\mathsf{Acc}(X)$, $\mathsf{Poly}_\mathsf{Arr}(X)$, $Q(X)$

2. $\mathcal{E}$, given access to $\mathcal{A}$'s outputs from the previous step, outputs $\mathsf{Poly}_\mathsf{Acc}(X)$, $\mathsf{Poly}_\mathsf{Arr}(X)$, $Q(X)$

3. $\mathcal{A}$ plays the part of the prover in showing that $Y_\mathsf{Zero}$ is zero at a random challenge $\zeta$

4. $\mathcal{A}$ wins if: 

   i) $\mathcal{V}$ accepts at the end of the protocol

   ii) $\mathsf{Sum}_\mathsf{Arr}\neq\sum_{i = 0}^{n-1} \mathsf{Arr}[i]$

Our proof is as follows:

For the second win condition to be fulfilled, one of the three constraints must be false. But then the $\mathsf{Poly}_\mathsf{Vanish}$ corresponding to that constraint is not vanishing on $\mathcal{H}_\kappa$, so $Q(X)$ is not a polynomial (it is a rational function). This means that $\mathcal{A}$ cannot calculated the correct commitment value $g^{Q(\tau)}$ without solving the t-SDH. Thus, $\mathcal{A}$ chooses an arbitrary value for $Q(\tau)$ and writes $K_Q = g^{Q(\tau)}$ to the transcript. Before this, it also writes commitments to  $\mathsf{Poly}_\mathsf{Acc}(X)$, and $\mathsf{Poly}_\mathsf{Arr}(X)$. Each commitment $\mathcal{A}$ has written is a linear combination of the elements in $[g, g^\tau, g^{\tau^2}, \dots,g^{\tau^{n-1}}]$. $\mathcal{E}$ is given these coefficients (since $\mathcal{A}$ is an algebraic adversary) so $\mathcal{E}$ can output the original polynomials.

$\mathcal{A}$ then obtains the random challenge $\zeta$ (using strong Fiat-Shamir). By the binding property of KZG commitments, $\mathsf{Poly}_\mathsf{Acc}(\zeta)$,  $\mathsf{Poly}_\mathsf{Arr}(\zeta)$, and $\mathsf{Poly}_\mathsf{Acc}(\zeta \cdot \omega)$ can only feasibly be opened to one value each. For $\mathcal{A}$ to have the verifier accept, it must produce a proof that $Q(\zeta)$ opens to $Q(\zeta) = \frac{Y_\mathsf{Vanish1} + \rho Y_\mathsf{Vanish2} + \rho^2Y_\mathsf{Vanish3}}{\zeta^\kappa - 1}$. This means being able to produce $g^{q(\tau)}$ where $q(\tau) = \frac{Q(\tau) - Q(\zeta)}{\tau - \zeta}$ and $Q(\zeta) = \frac{Y_\mathsf{Vanish1} + \rho Y_\mathsf{Vanish2} + \rho^2Y_\mathsf{Vanish3}}{\zeta^\kappa - 1}$. Since $Q(\tau)$ and $Q(\zeta)$ are known, this implies knowing $g^{\frac{1}{\tau - \zeta}}$. Thus $\mathcal{A}$ would have found $\langle\zeta,g^{\frac{1}{\tau - \zeta}}\rangle$, which is the t-SDH problem. We have shown that creating an accepting proof reduces to the t-SDH, so $A$'s probability of success is negligible.

### Zero-Knowledge

We prove that the above protocol is zero-knowledge when $\mathsf{PolyCommit}_\mathsf{Ped}$ (from the KZG paper) is used for the polynomial commitments. We do so by constructing a probabilistic polynomial time simulator $\mathcal{S}$ which, for every (possibly malicious) verifier $\mathcal{V}$, can output a view of the execution of the protocol that is indistinguishable from the view produced by the real execution of the program.

The simulator $\mathcal{S}$ generates an array $\mathsf{Arr'}$  whose product is equal to the disclosed value $\mathsf{Sum}_\mathsf{Arr}$ (this array could just have $\mathsf{Prod}_\mathsf{Sum}$ in one entry, and $0$'s elsewhere), then follows the same steps a prover would to prove the sum of this array. So, $\mathcal{S}$ computes the accumulator $\mathsf{Acc'}$ and interpolates the two arrays into their respective polynomials, $\mathsf{Poly}_\mathsf{Acc'}(X)$ and $\mathsf{Poly}_\mathsf{Arr'}(X)$. It computes $Q(X)'$ using  $\mathsf{Poly}_\mathsf{Acc'}(X)$ and $\mathsf{Poly}_\mathsf{Arr'}(X)$ and the random challenge point $\rho'$ (by strong Fiat-Shamir). $\mathcal{S}$ commits to each of these three polynomials (and writes the commitments to the transcript). Then, it generates the random challenge $\zeta'$ (once again this is by strong Fiat-Shamir). It creates opening proofs for $\mathsf{Poly}_\mathsf{Acc'}(\zeta'), \space \mathsf{Poly}_\mathsf{Arr'}(\zeta'), \space Q(\zeta')'$, and $\mathsf{Poly}_\mathsf{Acc'}(\zeta' \cdot \omega)$, and writes these to the transcript as well. Since $\mathcal{S}$ knows each of the above polynomials, it can honestly compute this step and the proof will be accepted by $\mathcal{V}$. The transcript it generates this way will be indistinguishable from a transcript generated from a real execution, since $\mathsf{PolyCommit}_\mathsf{Ped}$ has the property of Indistinguishability of Commitments due to the randomization by $h^{\hat{\phi}(x)}$. 

- This proof could also be done by defining a simulator that knows the trapdoor $\tau$ and can thus create a passing witness for any commitment. The proof for add1 is done in this style, but with small alterations would work here as well (and vice versa with this style of proof working for add1)