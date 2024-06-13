# Multiplication (Type 2)

## Recap of types

| Type      | Description                                                  | Recap                                                        | This |
| --------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| [mult1]() | $\mathsf{Arr}_3=\mathsf{Arr}_1 \cdot \mathsf{Arr}_2$         | $\mathsf{Arr}_3$ is the element-wise multiplication of $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$. |      |
| mult2     | $\mathsf{Prod}_\mathsf{Arr}=\prod_{i = 0}^{n-1} \mathsf{Arr}[i]$ | $\mathsf{Prod}_\mathsf{Arr}$ is the disclosed product of all the elements in $\mathsf{Arr}$. | ✅    |
| mult3     | $\prod_{i = 0}^{n-1} \mathsf{Arr}_1[i]=\prod_{i = 0}^{n-1} \mathsf{Arr}_2[i]$ | $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$ have the same undisclosed product. |      |

## Relation

$ \mathcal{R}_{\mathtt{mult3}} := \left\{ \begin{array}{l} (K_\mathsf{Arr_1},K_\mathsf{Arr2}) \end{array} \middle | \begin{array}{l} \mathsf{Arr_1} = [a_{(1,0)}, a_{(1,1)}, a_{(1,2)}, \dots, a_{(1,n-1)}],\\ \mathsf{Arr_2} = [a_{(2,0)}, a_{(2,1)}, a_{(2,2)}, \dots, a_{(2,n-1)}], \\  \mathsf{Poly}_\mathsf{Arr_1}=\mathsf{FFT.Interp}(\omega,\mathsf{Arr_1}), \\ \mathsf{Poly}_\mathsf{Arr_2}=\mathsf{FFT.Interp}(\omega,\mathsf{Arr_2}), \\ K_\mathsf{Arr_1}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_1}),\\ K_\mathsf{Arr_2}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_2}), \end{array} \right\} $

## Intuition

The prover ($\mathcal{P}$) holds two arrays $\mathsf{Arr_1}$ and $\mathsf{Arr_2}$ of $n$ integers from $\mathbb{Z}_q$: $[a_0, a_1, a_2, \dots, a_{n-1}]$. It will produce a succinct (independent of $n$) proof that they have the same undisclosed product. The prover will encode the two arrays into polynomials, $\mathsf{Poly}_\mathsf{Arr_1}$ and $\mathsf{Poly}_\mathsf{Arr_2}$ (using [evaluation points](../background/poly-iop) on the domain $\mathcal{H}_\kappa$) and commit to them as $K_\mathsf{Arr_1}$ and $K_\mathsf{Arr_2}$. The verifier ($\mathcal{V}$) cannot check either array directly (they may contain secret information, and even if they do not, it is too long to check) so the verifier only sees $K_\mathsf{Arr_1}$ and $K_\mathsf{Arr_2}$​.

In order to prove $K_\mathsf{Arr_1}$ and  $K_\mathsf{Arr_2}$ are consistent, the prover will build two helper arrays $\mathsf{Acc_1}$ and $\mathsf{Acc_2}$ called accumulators (or accumulating arrays or incremental arrays). This should not be confused with accumulators from cryptography, which are a concept related to succinct proofs but are distinct. As with $\mathsf{Arr_1}$ and $\mathsf{Arr_2}$, the prover will also encode $\mathsf{Acc_1}$ and $\mathsf{Acc_2}$ as a polynomials and provide a commitment to the verifier of each one. The idea is that the prover will prove a relation between each $\mathsf{Arr}$ and its $\mathsf{Acc}$; and a relation between $\mathsf{Acc_1}$ and $\mathsf{Acc_2}$. Put together, it will imply the correct relation between $\mathsf{Arr_1}$ and $\mathsf{Arr_2}$.

We will illustrate a small numerical example n $\mathbb{Z}_{97}$ of constructing an accumulator $\mathsf{Acc}$ for the array $\mathsf{Arr}= [84,67,11,92,36,67]$. The idea is to create a new array, $\mathsf{Acc}$, that starts the same as $\mathsf{Arr}$ and ends with the product of all entries of $\mathsf{Arr}$, by folding in the integers from $\mathsf{Arr}$ one-by-one with multiplication. Then each value in the new array will depend on only two values, as below. 

The first value in $\mathsf{Acc}$ will be a copy of the first value from $\mathsf{Arr}$:

* $\mathsf{Arr}= [84,67,11,92,36,67]$

* $\mathsf{Acc}= [84, \bot,\bot,\bot,\bot,\bot] $

The next value will be the multiplication (mod 97) of: 67 (the value at the same index in $\mathsf{Arr}$) and 84 (the previous value in $\mathsf{Acc}$):

* $\mathsf{Arr}= [84,67,11,92,36,67]$

* $ \mathsf{Acc} = [84, (67\cdot84),\bot,\bot,\bot,\bot] = [84, 2,\bot,\bot,\bot,\bot]$ 

The next value will be the multiplication of: 11 (the value at the same index in $\mathsf{Arr}$) and 2 (the previous value in $\mathsf{Acc}$):

* $\mathsf{Arr}= [84,67,11,92,36,67]$
* $ \mathsf{Acc} = [84, 2,(11\cdot2),\bot,\bot,\bot] = [84,2,22,\bot,\bot,\bot]$ 
* $ \mathsf{Acc} = [84, 2,22,(92\cdot22),\bot,\bot] = [84,2,22,84,\bot,\bot]$ 
* $ \mathsf{Acc} = [84, 2,22,84,(36\cdot84),\bot] = [84,2,22,84,17,\bot]$ 
* $ \mathsf{Acc} = [84,2,22,84,17,(67\cdot17)] = [84, 2, 22, 84, 17, 72]$ 

Notice the last value in $\mathsf{Acc}$ is the product of all the entries in $\mathsf{Arr}$. The prover wants to show five constraints:

1. The first value in $\mathsf{Acc_1}$ matches the first value in $\mathsf{Arr_1}$ 
2. The first value in $\mathsf{Acc_2}$ matches the first value in $\mathsf{Arr_2}$
3. The rest of the values in $\mathsf{Acc}_1$ are of the form $\mathsf{Acc_1}[i]=\mathsf{Arr_1}[i]\cdot\mathsf{Acc_1}[i-1]$​ 
4. The rest of the values in $\mathsf{Acc}_2$ are of the form $\mathsf{Acc_2}[i]=\mathsf{Arr_2}[i]\cdot\mathsf{Acc_2}[i-1]$ 
5. The last value in $\mathsf{Acc_1}$ matches the last value in $\mathsf{Acc_2}$

If all five constraints are true, then entries of $\mathsf{Arr_1}$ and $\mathsf{Arr_2}$ have the same product.

Last, while it is not necessary to do, it is often convenient to hold the the value of the product at the start of the array $\mathsf{Acc}$ instead of the end. For this reason, the mathematical explaination below will construct $\mathsf{Acc}$ "backwards" (or right-to-left) from the above example, where the last value of $\mathsf{Acc}$ matches the last value of $\mathsf{Arr}$, the values are folded in from right to left, and the first (leftmost) value of $\mathsf{Acc}$ is the product of all entries:

* $\mathsf{Arr}= [84,67,11,92,36,67]$
* $ \mathsf{Acc} = [\bot, \bot, \bot, \bot, \bot, 67]$ 
* $ \mathsf{Acc} = [\bot, \bot, \bot, \bot, 84, 67]$ 
* $\ldots$
* $ \mathsf{Acc} = [72, 84, 36, 65, 84, 67]$ 
* and the product of all entries in $\mathsf{Arr}$ is 72

## Protocol Details

### Array Level

* $\mathcal{P}$ holds an array $\mathsf{Arr_1} = [a_{(1,0)}, a_{(1,1)}, a_{(1,2)}, \dots, a_{(1,n-1)}]$ of $n$ integers ($a_{(1,i)} \in \mathbb{Z}_q$)
* $\mathcal{P}$ holds an array $\mathsf{Arr_2} = [a_{(2,0)}, a_{(2,1)}, a_{(2,2)}, \dots, a_{(2,n-1)}]$ of $n$ integers ($a_{(2,i)} \in \mathbb{Z}_q$)
* $\mathcal{P}$ computes array $\mathsf{Acc_j}$ as follows for $j \in [1,2]$:
  * $\mathsf{Acc_j}[n-1]\leftarrow\mathsf{Arr_j}[n-1]$
  * $\mathsf{Acc_j}[i]\leftarrow\mathsf{Arr_j}[i]\cdot\mathsf{Acc_j}[i+1]$ for $i$ from $n-2$ to 0

### Polynomial Level

We assume arrays $\mathsf{Arr_1}$, $\mathsf{Arr_2}$, $\mathsf{Acc_1}$ and $\mathsf{Acc_2}$ are encoded as the y-coordinates into a univariant polynomial where the x-coordinates (called the domain $\mathcal{H}_\kappa$) are chosen as the multiplicative group of order $\kappa$ with generator $\omega\in\mathbb{G}_\kappa$ (see [Background](../background/poly-iop.md) for more). In short, $\omega^0$ is the first element and $\omega^{\kappa-1}$ is the last element of $\mathcal{H}_\kappa$. If $\kappa$ is larger than the length of the array, the array can be padded with elements of value 1 (which will not change the product).

Recall the five constraints we want to prove: 

1. The first value in $\mathsf{Acc_1}$ matches the first value in $\mathsf{Arr_1}$ 
2. The first value in $\mathsf{Acc_2}$ matches the first value in $\mathsf{Arr_2}$
3. The rest of the values in $\mathsf{Acc}_1$ are of the form $\mathsf{Acc_1}[i]=\mathsf{Arr_1}[i]\cdot\mathsf{Acc_1}[i-1]$​ 
4. The rest of the values in $\mathsf{Acc}_2$ are of the form $\mathsf{Acc_2}[i]=\mathsf{Arr_2}[i]\cdot\mathsf{Acc_2}[i-1]$ 
5. The last value in $\mathsf{Acc_1}$ matches the last value in $\mathsf{Acc_2}$

In polynomial form, the constraints are:

1. For $X=w^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc_1}(X)=\mathsf{Poly}_\mathsf{Arr_1}(X)$​,
2. For $X=w^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc_2}(X)=\mathsf{Poly}_\mathsf{Arr_2}(X)$,
3. For all $X$ except $X=\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc_1}(X)=\mathsf{Poly}_\mathsf{Arr_1}(X)\cdot\mathsf{Poly}_\mathsf{Acc_1}(\omega\cdot X)$​ 
4. For all $X$ except $X=\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc_2}(X)=\mathsf{Poly}_\mathsf{Arr_2}(X)\cdot\mathsf{Poly}_\mathsf{Acc_2}(\omega\cdot X)$ 
5. For $X=w^0$: $\mathsf{Poly}_\mathsf{Acc_1}(X)=\mathsf{Poly}_\mathsf{Acc_2}(X)$

In constraint 2 and 3, $\mathsf{Poly}_\mathsf{Acc}(\omega\cdot X)$ can also be conceptualized as <span style="border-style:dotted;border-width: 2px;"> [rotate](./rotate)</span> applied to $\mathsf{Poly}_\mathsf{Acc}(X)$ by one element (rightward in the array view). Also note that constraint 2 and 3 do not hold at $X=\omega^{\kappa-1}$ because this value is defined by constraint 1 (for the last value of $X$, the "next" value, $\omega X$, wraps back to the first element of the array which is a boundary condition).

We adjust each of these constraints to show an equality with 0:

1. For $X=w^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc_1}(X)-\mathsf{Poly}_\mathsf{Arr_1}(X)=0$​,
2. For $X=w^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc_2}(X)-\mathsf{Poly}_\mathsf{Arr_2}(X)=0$,
3. For all $X$ except $X=\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc_1}(X)-\mathsf{Poly}_\mathsf{Arr_1}(X)\cdot\mathsf{Poly}_\mathsf{Acc_1}(\omega\cdot X)=0$​ 
4. For all $X$ except $X=\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc_2}(X)-\mathsf{Poly}_\mathsf{Arr_2}(X)\cdot\mathsf{Poly}_\mathsf{Acc_2}(\omega\cdot X)=0$ 
5. For $X=w^0$: $\mathsf{Poly}_\mathsf{Acc_1}(X)-\mathsf{Poly}_\mathsf{Acc_2}(X)=0$

Next we take care of the "for $X$" conditions by zeroing out the rest of the polynomial that is not zero. See the gadget <span style="border-style:dotted;border-width: 2px;"> [zero1](./zero1)</span> for more on why this works.

1. $\mathsf{Poly}_\mathsf{Vanish1}(X)=(\mathsf{Poly}_\mathsf{Acc_1}(X)-\mathsf{Poly}_\mathsf{Arr_1}(X))\cdot\frac{(X^\kappa-1)}{(X-\omega^{\kappa-1})}=0$​,
2. $\mathsf{Poly}_\mathsf{Vanish2}(X)=(\mathsf{Poly}_\mathsf{Acc_2}(X)-\mathsf{Poly}_\mathsf{Arr_2}(X))\cdot\frac{(X^\kappa-1)}{(X-\omega^{\kappa-1})}=0$,
3. $\mathsf{Poly}_\mathsf{Vanish3}(X)=(\mathsf{Poly}_\mathsf{Acc_1}(X)-\mathsf{Poly}_\mathsf{Arr_1}(X)\cdot\mathsf{Poly}_\mathsf{Acc_1}(\omega\cdot X))\cdot(X-\omega^{\kappa-1})=0$​ 
4. $\mathsf{Poly}_\mathsf{Vanish4}(X)=(\mathsf{Poly}_\mathsf{Acc_2}(X)-\mathsf{Poly}_\mathsf{Arr_2}(X)\cdot\mathsf{Poly}_\mathsf{Acc_2}(\omega\cdot X))\cdot(X-\omega^{\kappa-1})=0$ 
5. $\mathsf{Poly}_\mathsf{Vanish5}(X)=(\mathsf{Poly}_\mathsf{Acc_1}(X)-\mathsf{Poly}_\mathsf{Acc_2}(X)\cdot\frac{(X^\kappa-1)}{(X-\omega^0)}=0$

These equations are true for every value of $X \in \mathcal{H}_\kappa$ (but not necessarily true outside of these values). To show this, we divide each polynomial by  $X^\kappa - 1$, which is a minimal vanishing polynomial for $\mathcal{H}_\kappa$ that does not require interpolation to create. If the quotients are polynomials (and not rational functions), then $\mathsf{Poly}_\mathsf{Vanish1}(X)$, $\mathsf{Poly}_\mathsf{Vanish2}(X)$,  $\mathsf{Poly}_\mathsf{Vanish3}(X)$,  $\mathsf{Poly}_\mathsf{Vanish4}(X)$, and $\mathsf{Poly}_\mathsf{Vanish5}(X)$ must be vanishing on $\mathcal{H}_\kappa$ too. Specifically, the prove computes,

1. $Q_1(X) = \frac{\mathsf{Poly}_\mathsf{Vanish1}(X)}{X^\kappa - 1}$
2. $Q_2(X) = \frac{\mathsf{Poly}_\mathsf{Vanish2}(X)}{X^\kappa - 1}$
3. $Q_3(X) = \frac{\mathsf{Poly}_\mathsf{Vanish3}(X)}{X^\kappa - 1}$​
4. $Q_4(X) = \frac{\mathsf{Poly}_\mathsf{Vanish4}(X)}{X^\kappa - 1}$
5. $Q_5(X) = \frac{\mathsf{Poly}_\mathsf{Vanish5}(X)}{X^\kappa - 1}$

We can replace polynomials $Q_1(X)$, $Q_2(X)$,  $Q_3(X)$,  $Q_4(X)$, and $Q_5(X)$ with a single polynomial $Q(X)$. We can do this because all three constraints have the same format: $\mathsf{Poly}_\mathsf{Vanish_i}(X)=0$. The batching technique is to create a new polynomial with all three $\mathsf{Poly}_\mathsf{Vanish_i}(X)$ values as coefficients. If and (overwhelmingly) only if all three are vanishing, then so will the new polynomial. This polynomial will be evaluated at a random challenge point $\rho$ selected after the commitments to the earlier polynomials are fixed. 

$Q(X) = \frac{\mathsf{Poly}_\mathsf{Vanish1}(X) + \mathsf{Poly}_\mathsf{Vanish2}(X) \rho  +  \mathsf{Poly}_\mathsf{Vanish3}(X)\rho^2 + \mathsf{Poly}_\mathsf{Vanish4}(X)\rho^3 + \mathsf{Poly}_\mathsf{Vanish5}(X)\rho^4}{X^n - 1}$

By rearranging, we can get $\mathsf{Poly}_\mathsf{Zero}(X)$ as a true zero polynomial (zero at every value both in $\mathcal{H}_\kappa$ and outside of it):

$\mathsf{Poly}_\mathsf{Zero}(X)=\mathsf{Poly}_\mathsf{Vanish1}(X) + \mathsf{Poly}_\mathsf{Vanish2} (X) \rho + \mathsf{Poly}_\mathsf{Vanish3}(X) \rho^2 + \mathsf{Poly}_\mathsf{Vanish4}(X)\rho^3 + \mathsf{Poly}_\mathsf{Vanish5}(X)\rho^4 - Q(X)\cdot (X^n - 1)=0$

Ultimately the mult3 argument will satisfy the following constraints at the Commitment Level:

1. Show $Q(X)$ exists (as a polynomial that evenly divides the divisor)
2. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is correctly constructed from $\mathsf{Poly}_\mathsf{Acc_1}(X)$,  $\mathsf{Poly}_\mathsf{Acc_1}(\omega X)$, $\mathsf{Poly}_\mathsf{Arr_1}(X)$, $\mathsf{Poly}_\mathsf{Acc_2}(X)$,  $\mathsf{Poly}_\mathsf{Acc_2}(\omega X)$, and $\mathsf{Poly}_\mathsf{Arr_2}(X)$
3. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is the zero polynomial

### Commitment Level

The verifier will never see the arrays or polynomials themselves. They are undisclosed because they either (i) contain private data or (ii) they are too large to examine and maintain a succinct proof system. Instead the prover will use commitments.

The prover will write the following commitments to the transcript:

* $K_\mathsf{Arr_1}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_1}(X))$
* $K_\mathsf{Acc_1}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Acc_1}(X))$​
* $K_\mathsf{Arr_2}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_2}(X))$
* $K_\mathsf{Acc_2}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Acc_2}(X))$

The prover will generate a random challenge evaluation point (using strong Fiat-Shamir) on the polynomial that is outside of $\mathcal{H}_\kappa$. Call this point $\rho$. It will be used by the prover to create polynomial $Q(X)$ (see above) and the prover will write to the transcript: 

* $\rho$
* $K_Q=\mathsf{KZG.Commit}(Q(X))$

The prover will generate a second random challenge evaluation point (using strong Fiat-Shamir) on the polynomial that is outside of $\mathcal{H}_\kappa$. Call this point $\zeta$. The prover will write the three points and opening proofs to the transcript:

* $\zeta$
* $\mathsf{Poly}_\mathsf{Arr_1}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr_1},\zeta)$
* $\mathsf{Poly}_\mathsf{Arr_1}(\zeta\omega)=\mathsf{KZG.Open}(K_\mathsf{Arr_1},\zeta\omega)$
* $\mathsf{Poly}_\mathsf{Acc_1}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Acc_1},\zeta)$​
* $\mathsf{Poly}_\mathsf{Arr_2}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr_2},\zeta)$
* $\mathsf{Poly}_\mathsf{Arr_2}(\zeta\omega)=\mathsf{KZG.Open}(K_\mathsf{Arr_2},\zeta\omega)$
* $\mathsf{Poly}_\mathsf{Acc_2}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Acc_2},\zeta)$
* $Q(\zeta)=\mathsf{KZG.Open}(K_Q,\zeta)$

To check the proof, the verifier uses the transcript to construct the value $Y_\mathsf{Zero}$ as follows:

* $Y_\mathsf{Vanish1}= (\mathsf{Poly}_\mathsf{Acc_1}(\zeta)-\mathsf{Poly}_\mathsf{Arr_1}(\zeta))\cdot\frac{(\zeta^\kappa-1)}{(\zeta-\omega^{\kappa-1})}$
* $Y_\mathsf{Vanish2}= (\mathsf{Poly}_\mathsf{Acc_2}(\zeta)-\mathsf{Poly}_\mathsf{Arr_2}(\zeta))\cdot\frac{(\zeta^\kappa-1)}{(\zeta-\omega^{\kappa-1})}$
* $Y_\mathsf{Vanish3}=(\mathsf{Poly}_\mathsf{Acc_1}(\zeta)-\mathsf{Poly}_\mathsf{Arr_1}(\zeta)\cdot\mathsf{Poly}_\mathsf{Acc_1}(\omega\cdot \zeta))\cdot(\zeta-\omega^{\kappa-1})$
* $Y_\mathsf{Vanish4}= (\mathsf{Poly}_\mathsf{Acc_2}(\zeta)-\mathsf{Poly}_\mathsf{Arr_2}(\zeta)\cdot\mathsf{Poly}_\mathsf{Acc_2}(\omega\cdot \zeta))\cdot(\zeta-\omega^{\kappa-1})$
* $Y_\mathsf{Vanish5}= (\mathsf{Poly}_\mathsf{Acc_1}(\zeta)-\mathsf{Poly}_\mathsf{Acc_2}(\zeta)\cdot\frac{(\zeta^\kappa-1)}{(\zeta-\omega^0)}$
* $Y_\mathsf{Zero}=Y_\mathsf{Vanish1} + \rho Y_\mathsf{Vanish2} + \rho^2 Y_\mathsf{Vanish3} + \rho^3 Y_\mathsf{Vanish4} + \rho^4 Y_\mathsf{Vanish5}- Q(\zeta)\cdot (\zeta^n - 1)$

Finally, if the constraint system is true, the following constraint will be true (and will be false otherwise with overwhelming probability, due to the Schwartz-Zippel lemma on $\rho$ and $\zeta$) :

* $Y_\mathsf{Zero}\overset{?}{=}0$

## Implementations



## Security Proof

### Completeness

Any honest prover can do the computations explained above and create an accepting proof.

### Soundness

We prove knowledge soundness in the Algebraic Group Model (AGM). To do so, we must prove that there exists an efficient extractor $E$ such that for any algebraic adversary $A$ the probability of $A$ winning the following game is $\mathsf{negl}(\lambda)$.

1. Given $[g, g^\tau, g^{\tau^2}, \dots,g^{\tau^{n-1}}]$ $A$ outputs commitments to $\mathsf{Poly}_\mathsf{Acc}(X)$, $\mathsf{Poly}_\mathsf{Arr}(X)$, $Q$

2. $E$, given access to $A$'s outputs from the previous step, outputs $\mathsf{Poly}_\mathsf{Acc}(X)$, $\mathsf{Poly}_\mathsf{Arr}(X)$, $Q$

3. $A$ plays the part of the prover in showing that $Y_\mathsf{Zero}$ is zero at a random challenge $\zeta$

4. $A$ wins if: 

   i) $V$ accepts at the end of the protocol

   ii) $\mathsf{Prod}_\mathsf{Arr}\neq\prod_{i = 0}^{n-1} \mathsf{Arr}[i]$

Our proof is as follows:

For the second win condition to be fulfilled, one of the three constraints must be false. But then the $\mathsf{Poly}_\mathsf{Vanish}$ corresponding to that constraint is not vanishing on $\mathcal{H}_\kappa$, so $Q(X)$ is not a polynomial (it is a rational function). This means that $A$ cannot calcuated the correct commitment value $g^{Q(\tau)}$ without solving the t-SDH. Thus, $A$ chooses an arbitrary value for $Q(\tau)$ and sends $K_Q = g^{Q(\tau)}$. It also sends commitments to  $\mathsf{Poly}_\mathsf{Acc}(X)$, and $\mathsf{Poly}_\mathsf{Arr}(X)$. Each commitment $A$ has outputted is a linear combination of the elements in $[g, g^\tau, g^{\tau^2}, \dots,g^{\tau^{n-1}}]$. $E$ is given these coefficients (since $A$ is an algebraic adversary) so $E$ can output the original polynomials.

$A$ then obtains the random challenge $\zeta$ (using strong Fiat-Shamir). By the binding property of KZG commitments, $\mathsf{Poly}_\mathsf{Acc}(\zeta)$,  $\mathsf{Poly}_\mathsf{Arr}(\zeta)$, and $\mathsf{Poly}_\mathsf{Acc}(\zeta \cdot \omega)$ can only feasibliy be opened to one value each. For $A$ to have the verifier accept, they must send a proof that $Q(\zeta)$ opens to $Q(\zeta) = \frac{Y_\mathsf{Vanish1} + \rho Y_\mathsf{Vanish2} + \rho^2Y_\mathsf{Vanish3}}{\zeta^n - 1}$. This means being able to send $g^{q(\tau)}$ where $q(\tau) = \frac{Q(\tau) - Q(\zeta)}{\tau - \zeta}$ and $Q(\zeta) = \frac{Y_\mathsf{Vanish1} + \rho Y_\mathsf{Vanish2} + \rho^2Y_\mathsf{Vanish3}}{\zeta^n - 1}$. Since $Q(\tau)$ and $Q(\zeta)$ are known, this implies knowing $g^{\frac{1}{\tau - \zeta}}$. Thus the prover would have found $\langle\zeta,g^{\frac{1}{\tau - \zeta}}\rangle$, which is the t-SDH problem. We have shown that creating an accepting proof reduces to the t-SDH, so $A$'s probability of success is negligible.



### Zero-Knowledge

We prove that the above protocol is zero-knowledge when $\mathsf{PolyCommit}_\mathsf{Ped}$ (from the KZG paper) is used for the polynomial commitments. We do so by constructing a probabilistic polynomial time simulator $S$ which, for every (possibly malicious) verifier $V, can output a view of the execution of the protocol that is indistinguishable from the view produced by the real execution of the program.

The simulator $S$ generates an array $\mathsf{Arr'}$  whose product is equal to the disclosed value $\mathsf{Prod}_\mathsf{Arr}$ (this array could just have $\mathsf{Prod}_\mathsf{Arr}$ in one entry, and $1$'s elsewhere), then follows the same steps a prover would to prove the product of this array. So, $S$ computes the accumulator $\mathsf{Acc'}$ and interpolates the two arrays into their respective polynomials, $\mathsf{Poly}_\mathsf{Acc}(X)$ and $\mathsf{Poly}_\mathsf{Arr}(X)$. It computes $Q(X)$ using  $\mathsf{Poly}_\mathsf{Acc}(X)$ and $\mathsf{Poly}_\mathsf{Arr}(X)$ and the random challenge point $\rho'$ (by strong Fiat-Shamir). $A$ commits to each of these three polynomials (and writes the commitments to the transcript). Then, it generates the random challenge $\zeta'$ (once again this is by strong Fiat-Shamir). It creates opening proofs for $\mathsf{Poly}_\mathsf{Acc}(\zeta'), \space \mathsf{Poly}_\mathsf{Arr}(\zeta'), \space Q(\zeta')$, and $\mathsf{Poly}_\mathsf{Acc}(\zeta' \cdot \omega)$, and writes these to the transcript as well. Since $S$ knows each of the above polynomials, it can honestly compute this step and the proof will be accepted by $V$. The transcript it generates this way will be indistinguishable from a transcript generated from a real execution, since $\mathsf{PolyCommit}_\mathsf{Ped}$ has the property of Indistinguishability of Commitments due to the randomization by $h^{\hat{\phi}(x)}$. 

- Could also define a simulator knows the trapdoor $\tau$ and thus can create a passing witness for any commitment. The proof for mult1 is done in this style, but with small alterations would work here as well (and vice versa for this style of proof working for mult1)