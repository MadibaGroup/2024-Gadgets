# Addition (Type 1)

## Recap of types

  | Type           | Description                                                  | Recap                                                        | This |
  | -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
  | [add1](#) | $\mathsf{Arr}_3=\mathsf{Arr}_1 + \mathsf{Arr}_2$             | $\mathsf{Arr}_3$ is the element-wise addition of $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$. | ✅    |
  | [add2](../add2) | $\mathsf{Sum}_\mathsf{Arr}=\sum_{i = 0}^{n-1} \mathsf{Arr}[i]$ | $\mathsf{Sum}_\mathsf{Arr}$ is the disclosed sum of all the elements in $\mathsf{Arr}$. |      |
  | [add3](../add3) | $\sum_{i = 0}^{n-1} \mathsf{Arr}_1[i]=\sum_{i = 0}^{n-1} \mathsf{Arr}_2[i]$ | $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$ have the same undisclosed sum. |      |

## Relation

  $ \mathcal{R}_{\mathtt{add1}} := \left\{ \begin{array}{l} (K_\mathsf{Arr_1},K_\mathsf{Arr_2},K_\mathsf{Arr_3}) \end{array} \middle | \begin{array}{l} \mathsf{Arr_3}[i]=\mathsf{Arr_1}[i]+\mathsf{Arr_2}[i], 0\leq i \leq n, \\ \mathsf{Poly}_\mathsf{Arr_j}=\mathsf{FFT.Interp}(\omega,\mathsf{Arr_j}), 1\leq j \leq 3, \\ K_\mathsf{Arr_j}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_j}), 1\leq j \leq 3, \end{array} \right\} $

## Intuition

  The prover ($\mathcal{P}$) holds two arrays $\mathsf{Arr_1}$ and $\mathsf{Arr_2}$ of $n$ integers from $\mathbb{Z}_q$: $[a_0, a_1, a_2, \dots, a_{n-1}]$. It will produce a succinct (independent of $n$) proof that $\mathsf{Arr_3}$ is the element-wise sum of all the elements in the array: $\mathsf{Arr_3}[i]=\mathsf{Arr_1}[i]+\mathsf{Arr_2}[i]$. The prover will encode the three arrays into three polynomials: $\mathsf{Poly}_\mathsf{Arr_1}$, $\mathsf{Poly}_\mathsf{Arr_2}$, and $\mathsf{Poly}_\mathsf{Arr_3}$ (using [evaluation points]() on the domain $\mathcal{H}_\kappa$). It will commit to each polynomial: $K_\mathsf{Arr_1}$,$K_\mathsf{Arr_2}$, and $K_\mathsf{Arr_3}$. The verifier ($\mathcal{V}$) cannot check any of the $\mathsf{Arr_i}$ or $\mathsf{Poly}_\mathsf{Arr_i}$ values directly (they may contain secret information, and even if they do not, they are too long to check) so the verifier only sees $K_\mathsf{Arr_1}$,$K_\mathsf{Arr_2}$, and $K_\mathsf{Arr_3}$.

  In order to prove $K_\mathsf{Arr_1}$,$K_\mathsf{Arr_2}$, and $K_\mathsf{Arr_3}$ are consistent, the prover can use one of two methods. 

  The most straight-forward method is to use the additive homomorphic property of the KZG polynomial commitment scheme which states that for equal-sized polynomials on the same domain:

  * $K_\mathsf{Arr_1}\otimes K_\mathsf{Arr_2}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_1}\oplus \mathsf{Poly}_\mathsf{Arr_2})$

  Here $\otimes$ is multiplication in the KZG group (*e.g.,* $\mathbb{G}_1$ in BLS12-381) while $\oplus$ is addition in $\mathbb{Z}_q$ of each evaluation point in $ \mathsf{Poly}_\mathsf{Arr_1}$ with each evaluation point in $\mathsf{Poly}_\mathsf{Arr_2}$. If the prover ($\mathcal{P}$) can set $K_\mathsf{Arr_3}\leftarrow K_\mathsf{Arr_1}\otimes K_\mathsf{Arr_2}$, then the verifier can check $K_\mathsf{Arr_3}\stackrel{?}{=}K_\mathsf{Arr_1}\otimes K_\mathsf{Arr_2}$.

  However a second method is needed in other cases. Note that there are many different polynomials that interpolate $\mathsf{Arr_3}$ on the domain $\mathcal{H}_\kappa$ (but are different elsewhere in the polynomial outside of $\mathcal{H}_\kappa)$. Each of these polynomials will have a unique commitment value. So it is possible that $K_\mathsf{Arr_3}\neq K_\mathsf{Arr_1}\otimes K_\mathsf{Arr_2}$, and yet $\mathsf{Arr_3}[i]=\mathsf{Arr_1}[i]+\mathsf{Arr_2}[i]$ for all $i$. This arrises when $\mathsf{Poly}_\mathsf{Arr_3}$ comes from a different part of the protocol than $\mathsf{Poly}_\mathsf{Arr_1}$  and $\mathsf{Poly}_\mathsf{Arr_2}$.

  The second method is more general so it can be used in place of the first method (but is more expensive), as well as covering all cases where $\mathsf{Arr_3}[i]=\mathsf{Arr_1}[i]+\mathsf{Arr_2}[i]$. The idea is to show $\mathsf{Arr_3}[i]-\mathsf{Arr_1}[i]+\mathsf{Arr_2}[i]=0$ for each evaluation point in the domain $\mathcal{H}_\kappa$. Showing a polynomial is zero on the domain (a "vanishing polynomial") is a common sub-protocol used by many gadgets.

   

## Protocol Details

### Array Level

  * $\mathcal{P}$ holds an array $\mathsf{Arr_1} = [a_{(1,0)}, a_{(1,1)}, a_{(1,2)}, \dots, a_{(1,n-1)}]$ of $n$ integers ($a_{(1,i)} \in \mathbb{Z}_q$)
  * $\mathcal{P}$ holds an array $\mathsf{Arr_2} = [a_{(2,0)}, a_{(2,1)}, a_{(2,2)}, \dots, a_{(2,n-1)}]$ of $n$ integers ($a_{(2,i)} \in \mathbb{Z}_q$)
  * $\mathcal{P}$ computes or holds an array $\mathsf{Arr_3} = [a_{(3,0)}, a_{(3,1)}, a_{(3,2)}, \dots, a_{(3,n-1)}]$ of $n$ integers ($a_{(3,i)} \in \mathbb{Z}_q$) such that:
    * $\mathsf{Arr_3}[i]=\mathsf{Arr_1}[i]+\mathsf{Arr_2}[i]$ for $i$ from 0 to $n-1$

### Polynomial Level

  We assume the three arrays $\mathsf{Arr_1}$, $\mathsf{Arr_2}$, and $\mathsf{Arr_3}$ are encoded as the y-coordinates into a univariant polynomial where the x-coordinates (called the domain $\mathcal{H}_\kappa$) are chosen as the multiplicative group of order $\kappa$ with generator $\omega\in\mathbb{G}_\kappa$ (see [Background](../../background/poly-iop/) for more). In short, $\omega^0$ is the first element and $\omega^{\kappa-1}$ is the last element of $\mathcal{H}_\kappa$. If $\kappa$ is larger than the length of the array, the array can be padded with elements of value 0 (which will not change the sum).

  Recall the constraint we want to prove: 

  1. $\mathsf{Arr_3}[i]=\mathsf{Arr_1}[i]+\mathsf{Arr_2}[i]$ for $i$ from 0 to $n-1$

  In polynomial form, the constraint is:

  1. For all $X$ from $\omega^0$ to $\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Arr_3}(X)=\mathsf{Poly}_\mathsf{Arr_1}(X)+\mathsf{Poly}_\mathsf{Arr_2}(X)$ 

  We adjust the constraints to show an equality with 0 and label it:

  1. For all $X$ from $\omega^0$ to $\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Vanish}(X)= \mathsf{Poly}_\mathsf{Arr_1}(X)+\mathsf{Poly}_\mathsf{Arr_2}(X) - \mathsf{Poly}_\mathsf{Arr_3}(X)=0$ 

  This equation is true for every value of $X \in \mathcal{H}_\kappa$ (but not necessarily true outside of these values). To show this, we divide the polynomial by  $X^\kappa - 1$, which is a minimal vanishing polynomial for $\mathcal{H}_\kappa$ that does not require interpolation to create. If the quotient is polynomial (and not a rational function), then $\mathsf{Poly}_\mathsf{Vanish}(X)$ must be vanishing on $\mathcal{H}_\kappa$ too. Specifically, the prover computes:

  1. $Q(X) = \frac{\mathsf{Poly}_\mathsf{Vanish}(X)}{X^\kappa - 1}$

  By rearranging, we can get $\mathsf{Poly}_\mathsf{Zero}(X)$ as a true zero polynomial (zero at every value both in $\mathcal{H}_\kappa$ and outside of it):

  1. $\mathsf{Poly}_\mathsf{Zero}(X)=\mathsf{Poly}_\mathsf{Vanish}(X) - Q(X)\cdot (X^\kappa - 1)=0$

  Ultimately the add1 argument will satisfy the following constraints at the Commitment Level:

  1. Show $Q(X)$ exists (as a polynomial that evenly divides the divisor)
  2. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is correctly constructed from $\mathsf{Poly}_\mathsf{Arr_1}(X)$, $\mathsf{Poly}_\mathsf{Arr_2}(X)$, and $\mathsf{Poly}_\mathsf{Arr_3}(X)$.
  3. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is the zero polynomial

### Commitment Level

  The verifier will never see the arrays or polynomials themselves. They are undisclosed because they either (i) contain private data or (ii) are too large to examine and maintain a succinct proof system. Instead, the prover will use commitments.

  The prover will write the following commitments to the transcript:

  * $K_\mathsf{Arr_1}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_1}(X))$
  * $K_\mathsf{Arr_2}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_2}(X))$
  * $K_\mathsf{Arr_3}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_3}(X))$

  * $K_Q=\mathsf{KZG.Commit}(Q(X))$

  The prover will generate a random challenge evaluation point (using strong Fiat-Shamir) on the polynomial that is outside of $\mathcal{H}_\kappa$. Call this point $\zeta$. The prover will write the point and opening proofs to the transcript:

  * $\zeta$
  * $\mathsf{Poly}_\mathsf{Arr_1}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr_1},\zeta)$
  * $\mathsf{Poly}_\mathsf{Arr_2}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr_2},\zeta)$
  * $\mathsf{Poly}_\mathsf{Arr_3}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr_3},\zeta)$

  * $Q(\zeta)=\mathsf{KZG.Open}(K_Q,\zeta)$

  To check the proof, the verifier uses the transcript to construct the value $Y_\mathsf{Zero}$ as follows:

  * $Y_\mathsf{Vanish}=\mathsf{Poly}_\mathsf{Arr_1}(\zeta)+\mathsf{Poly}_\mathsf{Arr_2}(\zeta)-\mathsf{Poly}_\mathsf{Arr_3}( \zeta)$ 
  * $Y_\mathsf{Zero}=Y_\mathsf{Vanish1} - Q(\zeta)\cdot (\zeta^\kappa - 1)$

  Finally, if the constraint system is true, the following constraint will be true (and will be false otherwise with overwhelming probability, due to the Schwartz-Zippel lemma on $\zeta$) :

  * $Y_\mathsf{Zero}\overset{?}{=}0$

## Implementations

  * [Rust](https://github.com/MadibaGroup/2024-Gadgets-Code/tree/main/src)
  * [Mathematica](https://github.com/MadibaGroup/2024-Gadgets-Code/tree/main/Mathematica) (Toy Example)

## Security Proof

### Completeness

If $Y_\mathsf{Zero}$ is zero, then $\mathcal{V}$ will accept. Therefore, to show completeness, we show that any prover who holds $\mathsf{Arr_1}, \mathsf{Arr_2}$ and $\mathsf{Arr_3}$ such that $\mathsf{Arr_1}[i] + \mathsf{Arr_2}[i] - \mathsf{Arr_3}[i] = 0 \space \forall i \in [0, n - 1]$ can follow the steps outlined in the above protocol and the resulting $Y_\mathsf{Zero}$ will be equal to zero.  To see this, observed that $Y_\mathsf{Zero}$

$ = Y_\mathsf{Vanish} - Q(\zeta)(\zeta^\kappa - 1)$ 

$ = \mathsf{Poly}_\mathsf{Arr_1}(\zeta) + \mathsf{Poly}_\mathsf{Arr_2}(\zeta) - \mathsf{Poly}_\mathsf{Arr_3}( \zeta) - Q(\zeta)(\zeta^\kappa - 1)$

 $= \mathsf{Poly}_\mathsf{Arr_1}(\zeta) + \mathsf{Poly}_\mathsf{Arr_2}(\zeta) - \mathsf{Poly}_\mathsf{Arr_3}( \zeta) - \frac{\mathsf{Poly_{Vanish}}(\zeta)}{\zeta^\kappa - 1}\cdot(\zeta^\kappa - 1)$ 

$= \mathsf{Poly}_\mathsf{Arr_1}(\zeta) + \mathsf{Poly}_\mathsf{Arr_2}(\zeta) - \mathsf{Poly}_\mathsf{Arr_3}( \zeta) - (\mathsf{Poly}_\mathsf{Arr_1}(\zeta)+\mathsf{Poly}_\mathsf{Arr_2}(\zeta) - \mathsf{Poly}_\mathsf{Arr_3}(\zeta))$

$= 0$

Where the third equality relies on the fact that $\mathsf{Poly_{Vanish}}(X)$ is divisible by $X^\kappa - 1$. This is true if $\mathsf{Poly_{Vanish}}(X)$ is vanishing on $\mathcal{H_\kappa}$, i.e. if $\mathsf{Poly}_\mathsf{Arr_1}(X)+\mathsf{Poly}_\mathsf{Arr_2}(X) - \mathsf{Poly}_\mathsf{Arr_3}(X) =0 \space \forall X \in \mathcal{H}_\kappa$. This is true if if $\mathsf{Arr_1}[i] + \mathsf{Arr_2}[i] - \mathsf{Arr_3}[i] = 0 \space \forall i \in [0, \kappa - 1]$, since $\mathsf{Poly_j}(\omega^i) = \mathsf{Arr_j}[i] \space \forall i \in [0, \kappa - 1]$.  But $\mathsf{Arr_1}[i] + \mathsf{Arr_2}[i] - \mathsf{Arr_3}[i] = 0 \space \forall i \in [0, \kappa - 1]$ is precisely the relation tnat we assumed held for our prover (if $\kappa \gt n$ then the arrays get padded such that this relation still holds), thus the $Y_\mathsf{Zero}(X)$ it creates by following the protocol is zero, and its transcipt will be accepted.

### Soundness

We prove knowledge soundness in the Algebraic Group Model (AGM). To do so, we must prove that there exists an efficient extractor $\mathcal{E}$ such that for any algebraic adversary $\mathcal{A}$ the probability of $\mathcal{A}$ winning the following game is $\mathsf{negl}(\lambda)$.

1. Given $[g, g^\tau, g^{\tau^2}, \dots,g^{\tau^{n-1}}]$ $\mathcal{A}$ outputs commitments to $\mathsf{Poly}_\mathsf{Arr1}(X)$, $\mathsf{Poly}_\mathsf{Arr2}(X)$, $\mathsf{Poly}_\mathsf{Arr3}(X)$, $Q(X)$

2. $\mathcal{E}$, given access to $\mathcal{A}$'s outputs from the previous step, outputs $\mathsf{Poly}_\mathsf{Arr1}(X)$, $\mathsf{Poly}_\mathsf{Arr2}(X)$, $\mathsf{Poly}_\mathsf{Arr3}(X)$, $Q(X)$

3. $\mathcal{A}$ plays the part of the prover in showing that $Y_{\mathsf{Zero}}$ is zero at a random challenge $\zeta$

4. $\mathcal{A}$ wins if: 

   i) $\mathcal{\mathcal{V}}$ accepts at the end of the protocol

   ii) $\mathsf{Arr}_3\neq \mathsf{Arr}_1 + \mathsf{Arr}_2$

Our proof is as follows:

For the second win condition to be fulfilled, the constraint must not hold for at least one index of the arrays. But then $\mathsf{Poly}_\mathsf{Vanish}(X)$ is not vanishing on $\mathcal{H}_\kappa$, so $Q(X)$ is not a polynomial (it is a rational function). This means that $\mathcal{A}$ cannot calculate the correct commitment value $g^{Q(\tau)}$ without solving the t-SDH. Thus, $\mathcal{A}$ chooses an arbitrary value for $Q(\tau)$ and writes $K_Q = g^{Q(\tau)}$ to the transcript. Before this, it also writes commitments to  $\mathsf{Poly}_\mathsf{Arr1}(X)$, $\mathsf{Poly}_\mathsf{Arr2}(X)$, and $\mathsf{Poly}_\mathsf{Arr3}(X)$. Each commitment $\mathcal{A}$ has written is a linear combination of the elements in $[g, g^\tau, g^{\tau^2}, \dots,g^{\tau^{n-1}}]$. $\mathcal{E}$ is given these coefficients (since $\mathcal{A}$ is an algebraic adversary) so $\mathcal{E}$ can output the original polynomials.

$\mathcal{A}$ then obtains the random challenge $\zeta$ (using strong Fiat-Shamir). By the binding property of KZG commitments, $\mathsf{Poly}_\mathsf{Arr1}(\zeta)$,  $\mathsf{Poly}_\mathsf{Arr2}(\zeta)$, and $\mathsf{Poly}_\mathsf{Arr3}(\zeta)$ can only feasibliy be opened to one value each. For $\mathcal{A}$ to have the verifier accept, it must produce a proof that $Q(\zeta)$ opens to $Q(\zeta) = \frac{Y_\mathsf{Vanish}}{(\zeta^\kappa - 1)}$. This means being able to produce $g^{q(\tau)}$ where $q(\tau) = \frac{Q(\tau) - Q(\zeta)}{\tau - \zeta}$ and $Q(\zeta) = \frac{Y_\mathsf{Vanish}}{(\zeta^\kappa - 1)}$. Since $Q(\tau)$ and $Q(\zeta)$ are known, this implies knowing $g^{\frac{1}{\tau - \zeta}}$. Thus $\mathcal{A}$ would have found $\langle\zeta,g^{\frac{1}{\tau - \zeta}}\rangle$, which is the t-SDH problem. We have shown that creating an accepting proof reduces to the t-SDH, so $\mathcal{A}$'s probability of success is negligible.

### Zero-Knowledge

We prove that the above protocol is zero-knowledge when $\mathsf{PolyCommit}_\mathsf{Ped}$ (from the KZG paper) is used for the polynomial commitments. We do so by constructing a probabilistic polynomial time simulator $\mathcal{S}$ that knows the trapdoor $\tau$, which, for every (possibly malicious) verifier $\mathcal{V}$, can output a view of the execution of the protocol that is indistinguishable from the view produced by the real execution of the program.

The simulator $\mathcal{S}$ chooses arbitrary values for ${\mathsf{Poly}_\mathsf{Arr1}(\tau)}$, ${\mathsf{Poly}_\mathsf{Arr2}(\tau)}$, and $\mathsf{Poly}_\mathsf{Arr3}(\tau)$, then computes $g^{\mathsf{Poly}_\mathsf{Arr1}(\tau)}$,  $g^{\mathsf{Poly}_\mathsf{Arr2}(\tau)}$, and $g^{\mathsf{Poly}_\mathsf{Arr3}(\tau)}$ to write as the commitments $K_\mathsf{Arr1}$, $ K_\mathsf{Arr2}$, and $ K_\mathsf{Arr3}$. $\mathcal{S}$ then generates the challenge evaluation point $\rho$ (by strong Fiat-Shamir) and computes $Q(\tau)$ using $\rho$ and the values it chose for ${\mathsf{Poly}_\mathsf{Arr1}(\tau)}$, ${\mathsf{Poly}_\mathsf{Arr2}(\tau)}$, and $\mathsf{Poly}_\mathsf{Arr3}(\tau)$. $\mathcal{S}$ writes the commitment $K_Q = g^{Q(\tau)}$ to the transcript.

Now, $\mathcal{S}$ generates the second random challenge point $\zeta$ (which we assume is not in $\mathcal{H}_\kappa$; if it is in $\mathcal{H}_\kappa$, $\mathcal{S}$ simply restarts and runs from the beginning). This is once again by strong Fiat-Shamir.  $\mathcal{S}$ then create fake opening proofs for ${\mathsf{Poly}_\mathsf{Arr1}(\zeta)}$, ${\mathsf{Poly}_\mathsf{Arr2}(\zeta)}$, and $\mathsf{Poly}_\mathsf{Arr3}(\zeta)$, to arbitrary values. This is done using the knowledge of $\tau$, calculating the respective witness $q(\tau) = \frac{{f(\tau) - f(\zeta)}}{\tau - \zeta}$ for each of the polynomials.

Finally, $\mathcal{S}$ creates a fake opening proof for $Q(\zeta) = \frac{Y_\mathsf{Vanish}}{(\zeta^\kappa - 1)}$. This is done using knowledge of $\tau$ to calculate an accepting witness $q(\tau)$, as above. This means that $Y_\mathsf{Zero}$ will be zero, and the transcript will be accepted by the verifier. It is indistinguishable from a transcript generated from a real execution, since $\mathsf{PolyCommit}_\mathsf{Ped}$ has the property of Indistinguishability of Commitments due to the randomization by $h^{\hat{\phi}(x)}$. 

- For add2, the proof is written with a simulator that doesn't know the trapdoor; however, with small alterations the proof for add2 should apply here and vice versa
