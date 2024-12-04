# Addition (Type 3)

## Recap of types

| Type           | Description                                                  | Recap                                                        | This |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| [add1](../add1) | $\mathsf{Arr}_3=\mathsf{Arr}_1 + \mathsf{Arr}_2$             | $\mathsf{Arr}_3$ is the element-wise addition of $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$. |      |
| [add2](../add2) | $\mathsf{Sum}_\mathsf{Arr}=\sum_{i = 0}^{n-1} \mathsf{Arr}[i]$ | $\mathsf{Sum}_\mathsf{Arr}$ is the disclosed sum of all the elements in $\mathsf{Arr}$. |      |
| [add3](#) | $\sum_{i = 0}^{n-1} \mathsf{Arr}_1[i]=\sum_{i = 0}^{n-1} \mathsf{Arr}_2[i]$ | $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$ have the same undisclosed sum. | ✅    |

## Relation

$ \mathcal{R}_{\mathtt{add3}} := \left\{ \begin{array}{l} (K_\mathsf{Arr_1},K_\mathsf{Arr2}) \end{array} \middle | \begin{array}{l} \mathsf{Arr_1} = [a_{(1,0)}, a_{(1,1)}, a_{(1,2)}, \dots, a_{(1,n-1)}],\\ \mathsf{Arr_2} = [a_{(2,0)}, a_{(2,1)}, a_{(2,2)}, \dots, a_{(2,n-1)}], \\  \mathsf{Poly}_\mathsf{Arr_1}=\mathsf{FFT.Interp}(\omega,\mathsf{Arr_1}), \\ \mathsf{Poly}_\mathsf{Arr_2}=\mathsf{FFT.Interp}(\omega,\mathsf{Arr_2}), \\ K_\mathsf{Arr_1}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_1}),\\ K_\mathsf{Arr_2}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_2}), \end{array} \right\} $

## Intuition

The prover ($\mathcal{P}$) holds two arrays $\mathsf{Arr_1}$ and $\mathsf{Arr_2}$ of $n$ integers from $\mathbb{Z}_q$: $[a_0, a_1, a_2, \dots, a_{n-1}]$. It will produce a succinct (independent of $n$) proof that they have the same undisclosed sum. The prover will encode the two arrays into polynomials, $\mathsf{Poly}_\mathsf{Arr_1}$ and $\mathsf{Poly}_\mathsf{Arr_2}$ (using [evaluation points](../background/poly-iop) on the domain $\mathcal{H}_\kappa$) and commit to them as $K_\mathsf{Arr_1}$ and $K_\mathsf{Arr_2}$. The verifier ($\mathcal{V}$) cannot check either array directly (they may contain secret information, and even if they do not, it is too long to check) so the verifier only sees $K_\mathsf{Arr_1}$ and $K_\mathsf{Arr_2}$.

In order to prove $K_\mathsf{Arr_1}$ and  $K_\mathsf{Arr_2}$ are consistent, the prover will build two helper arrays $\mathsf{Acc_1}$ and $\mathsf{Acc_2}$ called accumulators (or accumulating arrays or incremental arrays). This should not be confused with accumulators from cryptography, which are a concept related to succinct proofs but are distinct. As with $\mathsf{Arr_1}$ and $\mathsf{Arr_2}$, the prover will also encode $\mathsf{Acc_1}$ and $\mathsf{Acc_2}$ as a polynomials and provide a commitment to the verifier of each one. The idea is that the prover will prove a relation between each $\mathsf{Arr}$ and its $\mathsf{Acc}$; and a relation between $\mathsf{Acc_1}$ and $\mathsf{Acc_2}$. Put together, it will imply the correct relation between $\mathsf{Arr_1}$ and $\mathsf{Arr_2}$.

We will illustrate a small numerical example in $\mathbb{Z}_{97}$ of constructing an accumulator $\mathsf{Acc}$ for the array $\mathsf{Arr}= [84,67,11,92,36,67]$. The idea is to create a new array, $\mathsf{Acc}$, that starts the same as $\mathsf{Arr}$ and ends with the sum of all entries of $\mathsf{Arr}$, by folding in the integers from $\mathsf{Arr}$ one-by-one with multiplication. Then each value in the new array will depend on only two values, as below. 

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

Last, while it is not necessary to do, it is often convenient to hold the the value $\mathsf{Sum}_\mathsf{Arr}$ at the start of the array $\mathsf{Acc}$ instead of the end. For this reason, the mathematical explaination below will construct $\mathsf{Acc}$ "backwards" (or right-to-left) from the above example, where the last value of $\mathsf{Acc}$ matches the last value of $\mathsf{Arr}$, the values are folded in from right to left, and the first (leftmost) value of $\mathsf{Acc}$ is $\mathsf{Sum}_\mathsf{Arr}$:

* $\mathsf{Arr}= [84,67,11,92,36,67]$
* $ \mathsf{Acc} = [\bot, \bot, \bot, \bot, \bot, 67]$ 
* $ \mathsf{Acc} = [\bot, \bot, \bot, \bot, 6, 67]$ 
* $\ldots$
* $ \mathsf{Acc} = [66, 79, 12, 1, 6, 67]$ 
* and the sum of all entries in $\mathsf{Acc}$ is 66

## Protocol Details

### Array Level

* $\mathcal{P}$ holds an array $\mathsf{Arr_1} = [a_{(1,0)}, a_{(1,1)}, a_{(1,2)}, \dots, a_{(1,n-1)}]$ of $n$ integers ($a_{(1,i)} \in \mathbb{Z}_q$)
* $\mathcal{P}$ holds an array $\mathsf{Arr_2} = [a_{(2,0)}, a_{(2,1)}, a_{(2,2)}, \dots, a_{(2,n-1)}]$ of $n$ integers ($a_{(2,i)} \in \mathbb{Z}_q$)
* $\mathcal{P}$ computes array $\mathsf{Acc_j}$ as follows for $j \in [1,2]$:
  * $\mathsf{Acc_j}[n-1]\leftarrow\mathsf{Arr_j}[n-1]$
  * $\mathsf{Acc_j}[i]\leftarrow\mathsf{Arr_j}[i]+\mathsf{Acc_j}[i+1]$ for $i$ from $n-2$ to 0

### Polynomial Level

We assume arrays $\mathsf{Arr_1}$, $\mathsf{Arr_2}$, $\mathsf{Acc_1}$ and $\mathsf{Acc_2}$ are encoded as the y-coordinates into a univariant polynomial where the x-coordinates (called the domain $\mathcal{H}_\kappa$) are chosen as the multiplicative group of order $\kappa$ with generator $\omega\in\mathbb{G}_\kappa$ (see [Background](../../background/poly-iop) for more). In short, $\omega^0$ is the first element and $\omega^{\kappa-1}$ is the last element of $\mathcal{H}_\kappa$. If $\kappa$ is larger than the length of the array, the array can be padded with elements of value 0 (which will not change the sum).

Recall the five constraints we want to prove (now adjusted to fit with an $\mathsf{Acc}$ that is constructed "backwards," as noted above): 

1. The last value in $\mathsf{Acc_1}$ matches the last value in $\mathsf{Arr_1}$ 
2. The last value in $\mathsf{Acc_2}$ matches the last value in $\mathsf{Arr_2}$
3. The rest of the values in $\mathsf{Acc}_1$ are of the form $\mathsf{Acc_1}[i]=\mathsf{Arr_1}[i]+\mathsf{Acc_1}[i-1]$ 
4. The rest of the values in $\mathsf{Acc}_2$ are of the form $\mathsf{Acc_2}[i]=\mathsf{Arr_2}[i]+\mathsf{Acc_2}[i-1]$ 
5. The first value in $\mathsf{Acc_1}$ matches the first value in $\mathsf{Acc_2}$

In polynomial form, the constraints are:

1. For $X=w^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc_1}(X)=\mathsf{Poly}_\mathsf{Arr_1}(X)$,
2. For $X=w^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc_2}(X)=\mathsf{Poly}_\mathsf{Arr_2}(X)$,
3. For all $X$ except $X=\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc_1}(X)=\mathsf{Poly}_\mathsf{Arr_1}(X)+\mathsf{Poly}_\mathsf{Acc_1}(\omega\cdot X)$ 
4. For all $X$ except $X=\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc_2}(X)=\mathsf{Poly}_\mathsf{Arr_2}(X)+\mathsf{Poly}_\mathsf{Acc_2}(\omega\cdot X)$ 
5. For $X=w^0$: $\mathsf{Poly}_\mathsf{Acc_1}(X)=\mathsf{Poly}_\mathsf{Acc_2}(X)$

In constraints 2 and 3, $\mathsf{Poly}_\mathsf{Acc}(\omega\cdot X)$ can also be conceptualized as <span style="border-style:dotted;border-width: 2px;"> [rotate](../rotate)</span> applied to $\mathsf{Poly}_\mathsf{Acc}(X)$ by one element (rightward in the array view). Also note that constraints 2 and 3 do not hold at $X=\omega^{\kappa-1}$ because this value is defined by constraint 1 (for the last value of $X$, the "next" value, $\omega X$, wraps back to the first element of the array which is a boundary condition).

We adjust each of these constraints to show an equality with 0:

1. For $X=w^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc_1}(X)-\mathsf{Poly}_\mathsf{Arr_1}(X)=0$,
2. For $X=w^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc_2}(X)-\mathsf{Poly}_\mathsf{Arr_2}(X)=0$,
3. For all $X$ except $X=\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc_1}(X)-\mathsf{Poly}_\mathsf{Arr_1}(X)+\mathsf{Poly}_\mathsf{Acc_1}(\omega\cdot X)=0$ 
4. For all $X$ except $X=\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc_2}(X)-\mathsf{Poly}_\mathsf{Arr_2}(X)+\mathsf{Poly}_\mathsf{Acc_2}(\omega\cdot X)=0$ 
5. For $X=w^0$: $\mathsf{Poly}_\mathsf{Acc_1}(X)-\mathsf{Poly}_\mathsf{Acc_2}(X)=0$

Next we take care of the "for $X$" conditions by zeroing out the rest of the polynomial that is not zero. See the gadget <span style="border-style:dotted;border-width: 2px;"> [zero1](../zero1)</span> for more on why this works.

1. $\mathsf{Poly}_\mathsf{Vanish1}(X)=(\mathsf{Poly}_\mathsf{Acc_1}(X)-\mathsf{Poly}_\mathsf{Arr_1}(X))\cdot\frac{(X^\kappa-1)}{(X-\omega^{\kappa-1})}=0$,
2. $\mathsf{Poly}_\mathsf{Vanish2}(X)=(\mathsf{Poly}_\mathsf{Acc_2}(X)-\mathsf{Poly}_\mathsf{Arr_2}(X))\cdot\frac{(X^\kappa-1)}{(X-\omega^{\kappa-1})}=0$,
3. $\mathsf{Poly}_\mathsf{Vanish3}(X)=(\mathsf{Poly}_\mathsf{Acc_1}(X)-(\mathsf{Poly}_\mathsf{Arr_1}(X)+\mathsf{Poly}_\mathsf{Acc_1}(\omega\cdot X)))\cdot(X-\omega^{\kappa-1})=0$ 
4. $\mathsf{Poly}_\mathsf{Vanish4}(X)=(\mathsf{Poly}_\mathsf{Acc_2}(X)-(\mathsf{Poly}_\mathsf{Arr_2}(X)+\mathsf{Poly}_\mathsf{Acc_2}(\omega\cdot X)))\cdot(X-\omega^{\kappa-1})=0$ 
5. $\mathsf{Poly}_\mathsf{Vanish5}(X)=(\mathsf{Poly}_\mathsf{Acc_1}(X)-\mathsf{Poly}_\mathsf{Acc_2}(X)\cdot\frac{(X^\kappa-1)}{(X-\omega^0)}=0$

These equations are true for every value of $X \in \mathcal{H}_\kappa$ (but not necessarily true outside of these values). To show this, we divide each polynomial by  $X^\kappa - 1$, which is a minimal vanishing polynomial for $\mathcal{H}_\kappa$ that does not require interpolation to create. If the quotients are polynomials (and not rational functions), then $\mathsf{Poly}_\mathsf{Vanish1}(X)$, $\mathsf{Poly}_\mathsf{Vanish2}(X)$,  $\mathsf{Poly}_\mathsf{Vanish3}(X)$,  $\mathsf{Poly}_\mathsf{Vanish4}(X)$, and $\mathsf{Poly}_\mathsf{Vanish5}(X)$ must be vanishing on $\mathcal{H}_\kappa$ too. Specifically, the prove computes,

1. $Q_1(X) = \frac{\mathsf{Poly}_\mathsf{Vanish1}(X)}{X^\kappa - 1}$
2. $Q_2(X) = \frac{\mathsf{Poly}_\mathsf{Vanish2}(X)}{X^\kappa - 1}$
3. $Q_3(X) = \frac{\mathsf{Poly}_\mathsf{Vanish3}(X)}{X^\kappa - 1}$
4. $Q_4(X) = \frac{\mathsf{Poly}_\mathsf{Vanish4}(X)}{X^\kappa - 1}$
5. $Q_5(X) = \frac{\mathsf{Poly}_\mathsf{Vanish5}(X)}{X^\kappa - 1}$

We can replace polynomials $Q_1(X)$, $Q_2(X)$,  $Q_3(X)$,  $Q_4(X)$, and $Q_5(X)$ with a single polynomial $Q(X)$. We can do this because all three constraints have the same format: $\mathsf{Poly}_\mathsf{Vanish_i}(X)=0$. The batching technique is to create a new polynomial with all five $\mathsf{Poly}_\mathsf{Vanish_i}(X)$ values as coefficients. If and (overwhelmingly) only if all five are vanishing, then so will the new polynomial. This polynomial will be evaluated at a random challenge point $\rho$ selected after the commitments to the earlier polynomials are fixed. 

$Q(X) = \frac{\mathsf{Poly}_\mathsf{Vanish1}(X) + \mathsf{Poly}_\mathsf{Vanish2}(X) \rho  +  \mathsf{Poly}_\mathsf{Vanish3}(X)\rho^2 + \mathsf{Poly}_\mathsf{Vanish4}(X)\rho^3 + \mathsf{Poly}_\mathsf{Vanish5}(X)\rho^4}{X^\kappa - 1}$

By rearranging, we can get $\mathsf{Poly}_\mathsf{Zero}(X)$ as a true zero polynomial (zero at every value both in $\mathcal{H}_\kappa$ and outside of it):

$\mathsf{Poly}_\mathsf{Zero}(X)=\mathsf{Poly}_\mathsf{Vanish1}(X) + \mathsf{Poly}_\mathsf{Vanish2} (X) \rho + \mathsf{Poly}_\mathsf{Vanish3}(X) \rho^2 + \mathsf{Poly}_\mathsf{Vanish4}(X)\rho^3 + \mathsf{Poly}_\mathsf{Vanish5}(X)\rho^4 - Q(X)\cdot (X^\kappa - 1)=0$

Ultimately the add3 argument will satisfy the following constraints at the Commitment Level:

1. Show $Q(X)$ exists (as a polynomial that evenly divides the divisor)
2. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is correctly constructed from $\mathsf{Poly}_\mathsf{Acc_1}(X)$,  $\mathsf{Poly}_\mathsf{Acc_1}(\omega X)$, $\mathsf{Poly}_\mathsf{Arr_1}(X)$, $\mathsf{Poly}_\mathsf{Acc_2}(X)$,  $\mathsf{Poly}_\mathsf{Acc_2}(\omega X)$, and $\mathsf{Poly}_\mathsf{Arr_2}(X)$
3. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is the zero polynomial

### Commitment Level

The verifier will never see the arrays or polynomials themselves. They are undisclosed because they either (i) contain private data or (ii) they are too large to examine and maintain a succinct proof system. Instead the prover will use commitments.

The prover will write the following commitments to the transcript:

* $K_\mathsf{Arr_1}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_1}(X))$
* $K_\mathsf{Acc_1}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Acc_1}(X))$
* $K_\mathsf{Arr_2}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_2}(X))$
* $K_\mathsf{Acc_2}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Acc_2}(X))$

The prover will generate a random challenge evaluation point (using strong Fiat-Shamir) on the polynomial that is outside of $\mathcal{H}_\kappa$. Call this point $\rho$. It will be used by the prover to create polynomial $Q(X)$ (see above) and the prover will write to the transcript: 

* $\rho$
* $K_Q=\mathsf{KZG.Commit}(Q(X))$

The prover will generate a second random challenge evaluation point (using strong Fiat-Shamir) on the polynomial that is outside of $\mathcal{H}_\kappa$. Call this point $\zeta$. The prover will write the point and opening proofs to the transcript:

* $\zeta$
* $\mathsf{Poly}_\mathsf{Arr_1}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr_1},\zeta)$
* $\mathsf{Poly}_\mathsf{Acc_1}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr_1},\zeta)$
* $\mathsf{Poly}_\mathsf{Acc_1}(\zeta\omega)=\mathsf{KZG.Open}(K_\mathsf{Acc_1},\zeta\omega)$
* $\mathsf{Poly}_\mathsf{Arr_2}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr_2},\zeta)$
* $\mathsf{Poly}_\mathsf{Acc_2}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr_2},\zeta)$
* $\mathsf{Poly}_\mathsf{Acc_2}(\zeta\omega)=\mathsf{KZG.Open}(K_\mathsf{Acc_2},\zeta\omega)$
* $Q(\zeta)=\mathsf{KZG.Open}(K_Q,\zeta)$

To check the proof, the verifier uses the transcript to construct the value $Y_\mathsf{Zero}$ as follows:

* $Y_\mathsf{Vanish1}= (\mathsf{Poly}_\mathsf{Acc_1}(\zeta)-\mathsf{Poly}_\mathsf{Arr_1}(\zeta))\cdot\frac{(\zeta^\kappa-1)}{(\zeta-\omega^{\kappa-1})}$
* $Y_\mathsf{Vanish2}= (\mathsf{Poly}_\mathsf{Acc_2}(\zeta)-\mathsf{Poly}_\mathsf{Arr_2}(\zeta))\cdot\frac{(\zeta^\kappa-1)}{(\zeta-\omega^{\kappa-1})}$
* $Y_\mathsf{Vanish3}=(\mathsf{Poly}_\mathsf{Acc_1}(\zeta)-(\mathsf{Poly}_\mathsf{Arr_1}(\zeta)+\mathsf{Poly}_\mathsf{Acc_1}(\omega\cdot \zeta)))\cdot(\zeta-\omega^{\kappa-1})$
* $Y_\mathsf{Vanish4}= (\mathsf{Poly}_\mathsf{Acc_2}(\zeta)-(\mathsf{Poly}_\mathsf{Arr_2}(\zeta)+\mathsf{Poly}_\mathsf{Acc_2}(\omega\cdot \zeta)))\cdot(\zeta-\omega^{\kappa-1})$
* $Y_\mathsf{Vanish5}= (\mathsf{Poly}_\mathsf{Acc_1}(\zeta)-\mathsf{Poly}_\mathsf{Acc_2}(\zeta)\cdot\frac{(\zeta^\kappa-1)}{(\zeta-\omega^0)}$
* $Y_\mathsf{Zero}=Y_\mathsf{Vanish1} + \rho Y_\mathsf{Vanish2} + \rho^2 Y_\mathsf{Vanish3} + \rho^3 Y_\mathsf{Vanish4} + \rho^4 Y_\mathsf{Vanish5}- Q(\zeta)\cdot (\zeta^\kappa - 1)$

Finally, if the constraint system is true, the following constraint will be true (and will be false otherwise with overwhelming probability, due to the Schwartz-Zippel lemma on $\rho$ and $\zeta$) :

* $Y_\mathsf{Zero}\overset{?}{=}0$

## Implementations



## Security Proof

### Completeness

If $Y_\mathsf{Zero}$ is zero, then $\mathcal{V}$ will accept. Therefore, to show completeness, we show that any prover who holds $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$ such that $\sum_{i = 0}^{n-1} \mathsf{Arr}_1[i]=\sum_{i = 0}^{n-1} \mathsf{Arr}_2[i] \space \forall i \in [0, n - 1]$ can follow the steps outlined in the above protocol and the resulting $Y_\mathsf{Zero}$ will be equal to zero.  To see this, observed that $Y_\mathsf{Zero}$

$ = Y_\mathsf{Vanish1} + \rho Y_\mathsf{Vanish2} + \rho^2 Y_\mathsf{Vanish3} + \rho^3 Y_\mathsf{Vanish4} + \rho^4 Y_\mathsf{Vanish5}- Q(\zeta)\cdot (\zeta^\kappa - 1)$ 

$= Y_\mathsf{Vanish1} + \rho Y_\mathsf{Vanish2} + \rho^2 Y_\mathsf{Vanish3} + \rho^3 Y_\mathsf{Vanish4} + \rho^4 Y_\mathsf{Vanish5} - (\frac{\mathsf{Poly}_\mathsf{Vanish1}(X) + \mathsf{Poly}_\mathsf{Vanish2}(X) \rho  +  \mathsf{Poly}_\mathsf{Vanish3}(X)\rho^2 + \mathsf{Poly}_\mathsf{Vanish4}(X)\rho^3 + \mathsf{Poly}_\mathsf{Vanish5}(X)\rho^4}{X^\kappa - 1})(\zeta^\kappa - 1)$

$= Y_\mathsf{Vanish1} + \rho Y_\mathsf{Vanish2} + \rho^2 Y_\mathsf{Vanish3} + \rho^3 Y_\mathsf{Vanish4} + \rho^4 Y_\mathsf{Vanish5} - {\mathsf{Poly}_\mathsf{Vanish1}(X) - \mathsf{Poly}_\mathsf{Vanish2}(X) \rho -  \mathsf{Poly}_\mathsf{Vanish3}(X)\rho^2 - \mathsf{Poly}_\mathsf{Vanish4}(X)\rho^3 - \mathsf{Poly}_\mathsf{Vanish5}(X)\rho^4}$

$= 0$

Where the finally equality holds because $Y_{\mathsf{Vanish_j}}=\mathsf{Poly_{Vanish_j}}(\zeta)$.

Note also that the second equality relies on the fact that $\mathsf{Poly}_\mathsf{Vanish1}(X) + \mathsf{Poly}_\mathsf{Vanish2}(X) \rho  +  \mathsf{Poly}_\mathsf{Vanish3}(X)\rho^2 + \mathsf{Poly}_\mathsf{Vanish4}(X)\rho^3 + \mathsf{Poly}_\mathsf{Vanish5}(X)\rho^4$ is divisible by $X^\kappa - 1$. This is true if $\mathsf{Poly_{Vanish_1}}(X), \mathsf{Poly_{Vanish_2}}(X)$, $\mathsf{Poly_{Vanish_3}}(X),$ $\mathsf{Poly_{Vanish_4}}(X)$, and $\mathsf{Poly_{Vanish_5}}(X),$ are all vanishing on $\mathcal{H_\kappa}$, i.e. if all three of the following conditions hold for all $X \in \mathcal{H}_\kappa$:

1. $ (\mathsf{Poly}_\mathsf{Acc_1}(X)-\mathsf{Poly}_\mathsf{Arr_1}(X))\cdot\frac{(X^\kappa-1)}{(X-\omega^{\kappa-1})}=0$
2. $ (\mathsf{Poly}_\mathsf{Acc_2}(X)-\mathsf{Poly}_\mathsf{Arr_2}(X))\cdot\frac{(X^\kappa-1)}{(X-\omega^{\kappa-1})}=0$
3. $(\mathsf{Poly}_\mathsf{Acc_1}(X)-(\mathsf{Poly}_\mathsf{Arr_1}(X)+\mathsf{Poly}_\mathsf{Acc_1}(\omega\cdot X)))\cdot(X-\omega^{\kappa-1})=0$ 
4. $ (\mathsf{Poly}_\mathsf{Acc_2}(X)-(\mathsf{Poly}_\mathsf{Arr_2}(X)+\mathsf{Poly}_\mathsf{Acc_2}(\omega\cdot X)))\cdot(X-\omega^{\kappa-1})=0$ 
5. $ (\mathsf{Poly}_\mathsf{Acc_1}(X)-\mathsf{Poly}_\mathsf{Acc_2}(X)\cdot\frac{(X^\kappa-1)}{(X-\omega^0)}=0$

These conditions, in turn, hold if:

1. For $X=w^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc_1}(X)=\mathsf{Poly}_\mathsf{Arr_1}(X)$
2. For $X=w^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc_2}(X)=\mathsf{Poly}_\mathsf{Arr_2}(X)$
3. For all $X$ except $X=\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc_1}(X)=\mathsf{Poly}_\mathsf{Arr_1}(X)+\mathsf{Poly}_\mathsf{Acc_1}(\omega\cdot X)$ 
4. For all $X$ except $X=\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc_2}(X)=\mathsf{Poly}_\mathsf{Arr_2}(X)+\mathsf{Poly}_\mathsf{Acc_2}(\omega\cdot X)$ 
5. For $X=w^0$: $\mathsf{Poly}_\mathsf{Acc_1}(X)=\mathsf{Poly}_\mathsf{Acc_2}(X)$

Where we get the "For $X$" due to zeroing parts of the polynomials (see [zero1](../zero1.md)). Since $\mathsf{Poly_j}(\omega^i) = \mathsf{Arr_j}[i] \space \forall i \in [0, \kappa - 1]$, the above conditions are true if:

1. The last value in $\mathsf{Acc_1}$ matches the last value in $\mathsf{Arr_1}$ 
2. The last value in $\mathsf{Acc_2}$ matches the last value in $\mathsf{Arr_2}$
3. The rest of the values in $\mathsf{Acc}_1$ are of the form $\mathsf{Acc_1}[i]=\mathsf{Arr_1}[i]+\mathsf{Acc_1}[i-1]$ 
4. The rest of the values in $\mathsf{Acc}_2$ are of the form $\mathsf{Acc_2}[i]=\mathsf{Arr_2}[i]+\mathsf{Acc_2}[i-1]$ 
5. The first value in $\mathsf{Acc_1}$ matches the first value in $\mathsf{Acc_2}$

Which are precisely the conditions the Intuitions sections explains will hold if the prover constructs their Accumulators by following the protocol and using $\mathsf{Arr}_1$ and $\mathsf{Arr_2}$ such that $\sum_{i = 0}^{n-1} \mathsf{Arr}_1[i] =\sum_{i = 0}^{n-1} \mathsf{Arr}_2[i] \space \forall i \in [0, n - 1]$. This is what we assumed true about the prover, thus the $Y_\mathsf{Zero}$ it creates by following the protocol is zero, and its transcript will be accepted.

### Soundness

We prove knowledge soundness in the Algebraic Group Model (AGM). To do so, we must prove that there exists an efficient extractor $\mathcal{E}$ such that for any algebraic adversary $\mathcal{A}$ the probability of $\mathcal{A}$ winning the following game is $\mathsf{negl}(\lambda)$.

1. Given $[g, g^\tau, g^{\tau^2}, \dots,g^{\tau^{n-1}}]$ $\mathcal{A}$ outputs commitments to $\mathsf{Poly}_\mathsf{Acc_1}(X)$, $\mathsf{Poly}_\mathsf{Arr_1}(X)$, $\mathsf{Poly}_\mathsf{Acc_{2}}(X)$, $\mathsf{Poly}_\mathsf{Arr_{2}}(X)$, $Q(X)$

2. $\mathcal{E}$, given access to $\mathcal{A}$'s outputs from the previous step, outputs $\mathsf{Poly}_\mathsf{Acc_1}(X)$, $\mathsf{Poly}_\mathsf{Arr_1}(X)$, $\mathsf{Poly}_\mathsf{Acc_{2}}(X)$, $\mathsf{Poly}_\mathsf{Arr_{2}}(X)$, $Q(X)$

3. $\mathcal{A}$ plays the part of the prover in showing that $Y_\mathsf{Zero}$ is zero at a random challenge $\zeta$

4. $\mathcal{A}$ wins if: 

   i) $\mathcal{V}$ accepts at the end of the protocol

   ii) $\sum_{i = 0}^{n-1} \mathsf{Arr_1}[i]\neq\sum_{i = 0}^{n-1} \mathsf{Arr_2}[i]$

Our proof is as follows:

For the second win condition to be fulfilled, one of the five constraints must be false. But then the $\mathsf{Poly}_\mathsf{Vanish}$ corresponding to that constraint is not vanishing on $\mathcal{H}_\kappa$, so $Q(X)$ is not a polynomial (it is a rational function). This means that $\mathcal{A}$ cannot calculated the correct commitment value $g^{Q(\tau)}$ without solving the t-SDH. Thus, $\mathcal{A}$ chooses an arbitrary value for $Q(\tau)$ and writes $K_Q = g^{Q(\tau)}$ to the transcript. Before this, it also writes commitments to  $\mathsf{Poly}_\mathsf{Acc_1}(X)$, $\mathsf{Poly}_\mathsf{Arr_1}(X)$,  $\mathsf{Poly}_\mathsf{Acc_2}(X)$, and $\mathsf{Poly}_\mathsf{Arr_2}(X)$. Each commitment $\mathcal{A}$ has written is a linear combination of the elements in $[g, g^\tau, g^{\tau^2}, \dots,g^{\tau^{n-1}}]$. $\mathcal{E}$ is given these coefficients (since $\mathcal{A}$ is an algebraic adversary) so $\mathcal{E}$ can output the original polynomials.

$\mathcal{A}$ then obtains the random challenge $\zeta$ (using strong Fiat-Shamir). By the binding property of KZG commitments, $\mathsf{Poly}_\mathsf{Arr_1}(\zeta)$, $\mathsf{Poly}_\mathsf{Acc_1}(\zeta)$, $\mathsf{Poly}_\mathsf{Acc_1}(\zeta \cdot \omega)$, $\mathsf{Poly}_\mathsf{Arr_2}(\zeta)$, $\mathsf{Poly}_\mathsf{Acc_2}(\zeta)$, and $\mathsf{Poly}_\mathsf{Acc_2}(\zeta \cdot \omega)$ can each only feasibly be opened to one value. For $\mathcal{A}$ to have the verifier accept, it must produce a proof that $Q(\zeta)$ opens to $Q(\zeta) = \frac{Y_\mathsf{Vanish1} + \rho Y_\mathsf{Vanish2}  +  \rho^2 Y_\mathsf{Vanish3} + \rho^3 Y_\mathsf{Vanish4} + \rho^4Y_\mathsf{Vanish5}}{(\zeta^\kappa - 1)}$. This means being able to produce $g^{q(\tau)}$ where $q(\tau) = \frac{Q(\tau) - Q(\zeta)}{\tau - \zeta}$ and $Q(\zeta) = \frac{Y_\mathsf{Vanish1} + \rho Y_\mathsf{Vanish2}  +  \rho^2 Y_\mathsf{Vanish3} + \rho^3 Y_\mathsf{Vanish4} + \rho^4Y_\mathsf{Vanish5}}{(\zeta^\kappa - 1)}$. Since $Q(\tau)$ and $Q(\zeta)$ are known, this implies knowing $g^{\frac{1}{\tau - \zeta}}$. Thus $\mathcal{A}$ would have found $\langle\zeta,g^{\frac{1}{\tau - \zeta}}\rangle$, which is the t-SDH problem. We have shown that creating an accepting proof reduces to the t-SDH, so $\mathcal{A}$'s probability of success is negligible.

### Zero-Knowledge

We prove that the above protocol is zero-knowledge when $\mathsf{PolyCommit}_\mathsf{Ped}$ (from the KZG paper) is used for the polynomial commitments. We do so by constructing a probabilistic polynomial time simulator $\mathcal{S}$ that knows the trapdoor $\tau$, which, for every (possibly malicious) verifier $\mathcal{V}$, can output a view of the execution of the protocol that is indistinguishable from the view produced by the real execution of the program.

The simulator $\mathcal{S}$ chooses arbitrary values for ${\mathsf{Poly}_\mathsf{Arr_1}(\tau)}$, ${\mathsf{Poly}_\mathsf{Acc_1}(\tau)}$, ${\mathsf{Poly}_\mathsf{Arr_2}(\tau)}$ and $\mathsf{Poly}_\mathsf{Acc_2}(\tau)$, then computes $g^{\mathsf{Poly}_\mathsf{Arr_1}(\tau)}$,  $g^{\mathsf{Poly}_\mathsf{Acc_1}(\tau)}$, $g^{\mathsf{Poly}_\mathsf{Arr_2}(\tau)}$, and $g^{\mathsf{Poly}_\mathsf{Acc_2}(\tau)}$ to write as the commitments $K_\mathsf{Arr_1}$, $ K_\mathsf{Acc_1}$, $K_\mathsf{Arr_2}$, and $ K_\mathsf{Acc_2}$. $\mathcal{S}$ then generates the challenge evaluation point $\rho$ (by strong Fiat-Shamir) and computes $Q(\tau)$ using $\rho$ and the values it chose for ${\mathsf{Poly}_\mathsf{Arr_1}(\tau)}$, ${\mathsf{Poly}_\mathsf{Acc_1}(\tau)}$, ${\mathsf{Poly}_\mathsf{Arr_2}(\tau)}$ and $\mathsf{Poly}_\mathsf{Acc_2}(\tau)$. $\mathcal{S}$ writes the commitment $K_Q = g^{Q(\tau)}$.

Now, $\mathcal{S}$ generates the second random challenge point $\zeta$ (which we assume is not in $\mathcal{H}_\kappa$; if it is in $\mathcal{H}_\kappa$, $\mathcal{S}$ simply restarts and runs from the beginning). This is once again by strong Fiat-Shamir. $\mathcal{S}$ then create fake opening proofs for ${\mathsf{Poly}_\mathsf{Arr_1}(\zeta)}$, ${\mathsf{Poly}_\mathsf{Acc_1}(\zeta)}$, ${\mathsf{Poly}_\mathsf{Acc_1}(\zeta \cdot \omega)}$, ${\mathsf{Poly}_\mathsf{Arr_2}(\zeta)}$, ${\mathsf{Poly}_\mathsf{Acc_2}(\zeta)}$, and $\mathsf{Poly}_\mathsf{Acc_2}(\zeta\cdot\omega)$, to arbitrary values. This is done using the knowledge of $\tau$, calculating the respective witness $q(\tau) = \frac{{f(\tau) - f(\zeta)}}{\tau - \zeta}$ for each of the polynomials.

Finally, $\mathcal{S}$ creates a fake opening proof for $Q(\zeta) = \frac{Y_\mathsf{Vanish1} + \rho Y_\mathsf{Vanish2}  +  \rho^2 Y_\mathsf{Vanish3} + \rho^3 Y_\mathsf{Vanish4} + \rho^4Y_\mathsf{Vanish5}}{(\zeta^\kappa - 1)}$. This is done using knowledge of $\tau$ to calculate an accepting witness $q(\tau)$, as above. This means that $Y_\mathsf{Zero}$ will be zero, and the transcript will be accepted by the verifier. It is indistinguishable from a transcript generates from a real execution, since $\mathsf{PolyCommit}_\mathsf{Ped}$ has the property of Indistinguishability of Commitments due to the randomization by $h^{\hat{\phi}(x)}$. 

- For mult2, the proof is written with a simulator that doesn't know the trapdoor; however, with small alterations the proof for mult2 should apply here and vice versa