# Shuffle (Type 1)

## Recap of types

| Type                   | Description                                            | Recap                                                        | This |
| ---------------------- | ------------------------------------------------------ | :----------------------------------------------------------- | ---- |
| [shuffle1](./shuffle1) | $\mathsf{Arr}_2=\mathsf{Permute}(\mathsf{Arr}_1)$      | Array $\mathsf{Arr}_2$ is a shuffle of $\mathsf{Arr}_1$ for some undisclosed permutation $\pi$. | ✅    |
| [shuffle2](./shuffle2) | $\mathsf{Arr}_2=\mathsf{Permute}(\mathsf{Arr}_1 ,\pi)$ | Array $\mathsf{Arr}_2$ is a shuffle of $\mathsf{Arr}_1$ under a disclosed permutation $\pi$. |      |

## Relation

$ \mathcal{R}_{\mathtt{lookup1}} := \left\{ \begin{array}{l} (K_\mathsf{Arr}) \end{array} \middle | \begin{array}{l} \mathsf{Arr} = [a_0, a_1, a_2, \dots, a_{n-1}], \\ \mathsf{Poly}_\mathsf{Arr}=\mathsf{FFT.Interp}(\omega,\mathsf{Arr}), \\ K_\mathsf{Arr}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr}), \end{array} \right\} $

## Intuition

The prover ($\mathcal{P}$) holds 2 arrays, $\mathsf{Arr_1 }$ and $\mathsf{Arr_2}$, of $n$ integers from $\mathbb{Z}_q$: $[a_0, a_1, a_2, \dots, a_{n-1}]$. It will produce a succinct (independent of $n$) proof that $\mathsf{Arr}_2$ is a shuffle of $\mathsf{Arr}_1$ for some undisclosed permutation $\pi$. The prover will encode the two arrays into polynomials, $\mathsf{Poly}_\mathsf{Arr_1}$ and $\mathsf{Poly}_\mathsf{Arr_2}$ (using [evaluation points](../background/poly-iop) on the domain $\mathcal{H}_\kappa$) and commit to them as $K_\mathsf{Arr_1}$ and $K_\mathsf{Arr_2}$.  The verifier ($\mathcal{V}$) cannot check either array directly (they may contain secret information, and even if they do not, it is too long to check) so the verifier only sees $K_\mathsf{Arr_1}$ and $K_\mathsf{Arr_2}$.

One idea to check that $\mathsf{Arr_2}$ is a permutation of $\mathsf{Arr_1}$​ might be to compare the product of the entries of the two arrays. If the permutation relation holds, then the products will be equal; however, many arrays can have their entries multiply to the same number, without necessarily containing the same elements. 

Instead, the prover constructs two new arrays, $\mathsf{Arr_1}'$ and $\mathsf{Arr_2}'$, where $\mathsf{Arr_j}'$ contains the points $ \{r - \mathsf{Poly}_\mathsf{Arr_j}(a)  \}_{a \in \mathcal{H}_\kappa}$, for $r$ a random field element. Then, a product check is run on these two arrays. One way to understand why this works is to think of it as creating two auxilary polynomials, ${\mathsf{Poly}}_\mathsf{Arr_1'}$ and ${\mathsf{Poly}}_\mathsf{Arr_2'}$, where $\mathsf{Poly}_\mathsf{Arr_j'}(X) = \prod^{}_{}(X - \mathsf{Poly}_\mathsf{Arr_j}(a))$ for $a \in \mathcal{H}_\kappa$. If ${\mathsf{Poly}}_\mathsf{Arr_1'}$ = $\mathsf{Poly}_\mathsf{Arr_2'}$, then they have the same roots. This means that $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$ must contain the same elements (in possibly different orders); in other words, $\mathsf{Arr}_2$ is a permutation of $\mathsf{Arr}_1$. To check this equality, generate a random challenge point $r$ and check the products are equal at that point.

In addition to demontrasting the equality of the product of $\mathsf{Arr_1}'$ and $\mathsf{Arr_2}'$, it must also be shown that these two arrays are defined correctly in terms of the original arrays. In other words, it must be shown that $\mathsf{Arr_j}'[i]= r - \mathsf{Arr_j}[i]$. Once this, in addition to the product check, has been done, it has been shown that $\mathsf{Arr}_2$ is a shuffle of $\mathsf{Arr}_1$ for some undisclosed permutation $\pi$. 

## Protocol Details

### Array Level

* $\mathcal{P}$ holds an array $\mathsf{Arr_1} = [a_{(1,0)}, a_{(1,1)}, a_{(1,2)}, \dots, a_{(1,n-1)}]$ of $n$ integers ($a_{(1,i)} \in \mathbb{Z}_q$)
* $\mathcal{P}$ holds an array $\mathsf{Arr_2} = [a_{(2,0)}, a_{(2,1)}, a_{(2,2)}, \dots, a_{(2,n-1)}]$ of $n$ integers ($a_{(2,i)} \in \mathbb{Z}_q$​)
* $\mathcal{P}$ computes $\mathsf{Arr_j}'$ as follows for $j \in [1,2]$:
  * $\mathsf{Arr_j}'[i]= r - \mathsf{Arr_j}[i]$

### Polynomial Level

We assume that $\mathsf{Arr_1}$ and $\mathsf{Arr_2}$ are encoded as the y-coordinates into a univariant polynomial where the x-coordinates (called the domain $\mathcal{H}_\kappa$) are chosen as the multiplicative group of order $\kappa$ with generator $\omega\in\mathbb{G}_\kappa$ (see [Background](../background/poly-iop.md) for more). In short, $\omega^0$ is the first element and $\omega^{\kappa-1}$ is the last element of $\mathcal{H}_\kappa$. If $\kappa$ is larger than the length of the array, the array can be padded with elements of value 0 or 1.

Recall the two steps we want to prove: 

1. $\prod_{a \in \mathcal{H}_\kappa}\{r - \mathsf{Poly}_\mathsf{Arr_1}(a)\} = \prod_{a \in \mathcal{H}_\kappa}\{r - \mathsf{Poly}_\mathsf{Arr_2}(a)\}$ 
2. $\mathsf{Arr_j}'[i]= r - \mathsf{Arr_j}[i]$ for $j \in [1,2]$, $0 \leq 1 \leq n-1$

The first step is done as a [mult3](./mult3) product check, and we write the second step as two constraints in polynomial form. From this point on we focus on the polynomial details of the second step.

1. For all $X$ from $\omega^0$ to $\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Arr_1'}(X) = (r - \mathsf{Poly}_\mathsf{Arr_1}(X))$​ 
2. For all $X$ from $\omega^0$ to $\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Arr_2'}(X) = (r - \mathsf{Poly}_\mathsf{Arr_2}(X))$ 

We adjust each of these constraints to show an equality with 0 and label them:

1. $\mathsf{Poly}_\mathsf{Vanish1}(X)= \mathsf{Poly}_\mathsf{Arr_1'}(X) - (r - \mathsf{Poly}_\mathsf{Arr_1}(X)) = 0$​
2. $\mathsf{Poly}_\mathsf{Vanish2}(X)= \mathsf{Poly}_\mathsf{Arr_2'}(X) - (r - \mathsf{Poly}_\mathsf{Arr_2}(X)) = 0$

This equation is true for every value of $X \in \mathcal{H}_\kappa$ (but not necessarily true outside of these values). To show this, we divide each polynomial by  $X^\kappa - 1$, which is a minimal vanishing polynomial for $\mathcal{H}_\kappa$ that does not require interpolation to create. If the quotients are polynomials (and not rational functions), then $\mathsf{Poly}_\mathsf{Vanish}(X)$ must be vanishing on $\mathcal{H}_\kappa$ too. Specifically, the prover computes:

1. $Q_1(X) = \frac{\mathsf{Poly}_\mathsf{Vanish1}(X)}{X^\kappa - 1}$​
2. $Q_2(X) = \frac{\mathsf{Poly}_\mathsf{Vanish2}(X)}{X^\kappa - 1}$

We can replace polynomials $Q_1(X)$ and $Q_2(X)$ with a single polynomial $Q(X)$. We can do this because both constraints have the same format: $\mathsf{Poly}_\mathsf{Vanish_i}(X)=0$. The batching technique is to create a new polynomial with both $\mathsf{Poly}_\mathsf{Vanish_i}(X)$ values as coefficients. If and (overwhelmingly) only if all three are vanishing, then so will the new polynomial. This polynomial will be evaluated at a random challenge point $\rho$ selected after the commitments to the earlier polynomials are fixed. 

$Q(X) = \frac{\mathsf{Poly}_\mathsf{Vanish1}(X) + \mathsf{Poly}_\mathsf{Vanish2}(X) \rho}{X^n - 1}$

By rearranging, we can get $\mathsf{Poly}_\mathsf{Zero}(X)$ as a true zero polynomial (zero at every value both in $\mathcal{H}_\kappa$ and outside of it):

$\mathsf{Poly}_\mathsf{Zero}(X)=\mathsf{Poly}_\mathsf{Vanish1}(X) + \rho \mathsf{Poly}_\mathsf{Vanish2}(X) - Q(X)\cdot (X^n - 1)=0$

Ultimately the shuffle1 argument will satisfy the following constraints at the Commitment Level:

1. Show $Q(X)$ exists (as a polynomial that evenly divides the divisor)
2. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is correctly constructed from $\mathsf{Poly}_\mathsf{Arr}(X)$
3. Show $\mathsf{Poly}_\mathsf{Zero}(X)$​ is the zero polynomial

In addition, it will show that $\prod_{a \in \mathcal{H}_\kappa}\{r - \mathsf{Poly}_\mathsf{Arr_1}(a)\} = \prod_{a \in \mathcal{H}_\kappa}\{r - \mathsf{Poly}_\mathsf{Arr_2}(a)\}$  using a [mult3](./mult3) product check.

### Commitment Level

The verifier will never see the arrays or polynomials themselves. They are undisclosed because they either (i) contain private data or (ii) they are too large to examine and maintain a succinct proof system. Instead the prover will use commitments.

The prover will create a transcript for the product check, as decribed in [mult3](./mult3). Below, we give detail specific to the second step, $\mathsf{Arr_j}'[i]= r - \mathsf{Arr_j}[i]$ for $j \in [1,2]$, $0 \leq 1 \leq n-1$.

The prover will write the following commitments to the transcript:

* $K_\mathsf{Arr_1}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_1}(X))$​
* $K_\mathsf{Arr_2}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_2}(X))$​​

The prover will generate a random challenge evaluation point (using strong Fiat-Shamir) on the polynomials that is outside of $\mathcal{H}_\kappa$. Call this point $r$. It will use this point to define the sets $ \{r - \mathsf{Poly}_\mathsf{Arr_j}(a)  \}_{a \in \mathcal{H}_\kappa}$ and run the product check. It will write the product check into the transcript, as well as the point $r$ and the following polynomials:

* $r$
* $K_\mathsf{Arr_1'}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_1'}(X))$​
* $K_\mathsf{Arr_2'}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_2'}(X))$
* $K_Q=\mathsf{KZG.Commit}(Q(X))$​

The prover will generate a random challenge evaluation point (using strong Fiat-Shamir) on the polynomials that is outside of $\mathcal{H}_\kappa$. Call this point $\zeta$. The prover will write the point and opening proofs to the transcript:

* $\zeta$
* $\mathsf{Poly}_\mathsf{Arr_1}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr_1},\zeta)$​
* $\mathsf{Poly}_\mathsf{Arr_2}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr_2},\zeta)$​
* $\mathsf{Poly}_\mathsf{Arr_1'}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr_1'},\zeta)$​
* $\mathsf{Poly}_\mathsf{Arr_2'}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr_2'},\zeta)$
* $Q(\zeta)=\mathsf{KZG.Open}(K_Q,\zeta)$​

To check the proof, the verifier uses the transcript to construct the value $Y_\mathsf{Zero}$ as follows:

* $Y_\mathsf{Vanish}=\mathsf{Poly}_\mathsf{Arr}(\zeta) \cdot (\mathsf{Poly}_\mathsf{Arr}(\zeta) - 1)$
* $Y_\mathsf{Zero}=Y_\mathsf{Vanish1} - Q(\zeta)\cdot (\zeta^n - 1)$

Finally, if the constraint system is true, the following constraint will be true (and will be false otherwise with overwhelming probability, due to the Schwartz-Zippel lemma on $\zeta$) :

* $Y_\mathsf{Zero}\overset{?}{=}0$

## Implementations



## Security Proof

### Completeness

Any honest prover can do the computations explained above and create an accepting proof.

### Soundness

We prove knowledge soundness in the Algebraic Group Model (AGM). We assume soundness of the product check (it is proven in [mult3](./mult3)) and conduct a proof of soundness for the rest of the protocol. To do so, we must prove that there exists an efficient extractor $E$ such that for any algebraic adversary $A$ the probability of $A$ winning the following game is $\mathsf{negl}(\lambda)$.

1. Given $[g, g^\tau, g^{\tau^2}, \dots,g^{\tau^{n-1}}]$ $A$ outputs commitments to $\mathsf{Poly}_\mathsf{Arr_1}(X)$, $\mathsf{Poly}_\mathsf{Arr_2}(X)$, $\mathsf{Poly}_\mathsf{Arr_1'}(X)$, $\mathsf{Poly}_\mathsf{Arr_2'}(X)$, $Q$

2. $E$, given access to $A$'s outputs from the previous step, outputs $\mathsf{Poly}_\mathsf{Arr_1}(X)$, $\mathsf{Poly}_\mathsf{Arr_2}(X)$, $\mathsf{Poly}_\mathsf{Arr_1'}(X)$, $\mathsf{Poly}_\mathsf{Arr_2'}(X)$, $Q$

3. $A$ plays the part of the prover in showing that $Y_{\mathsf{Zero}}$ is zero at a random challenge $\zeta$

4. $A$ wins if: 

   i) $V$ accepts at the end of the protocol

   ii) $\mathsf{Arr}_2 \neq \mathsf{Permute}(\mathsf{Arr}_1)$

Our proof is as follows:

For the second win condition to be fulfilled, there must be some $a \in \mathsf{Arr_2}, a \notin \mathsf{Arr_1}$, or vice versa. WLOG, we assume the first case. Since $\mathsf{Arr_1}$ and $\mathsf{Arr_2}$ have different entries, $\prod^{}_{}(X - \mathsf{Poly}_\mathsf{Arr_1}(a)$ and $\prod^{}_{}(X - \mathsf{Poly}_\mathsf{Arr_2} (a)$ are different polynomials. By the Schwartz-Zippel lemma, there is thus negligible probability that they are equal at $r$. Thus to pass the product check with greater than negligible probability, $A$ must pass in $\mathsf{Arr_j'}$ such that $\mathsf{Arr_j'}[i] \neq r - \mathsf{Arr_j}[i]$ for some index $i$ and $j \in [1, 2]$. But then $\mathsf{Poly}_\mathsf{Vanish}(X)$ is not vanishing on $\mathcal{H}_\kappa$, so $Q(X)$ is not a polynomial (it is a rational function). This means that $A$ cannot calcuated the correct commitment value $g^{Q(\tau)}$ without solving the t-SDH. Thus, $A$ chooses an arbitrary value for $Q(\tau)$ and sends $K_Q = g^{Q(\tau)}$. It also sends a commitment to  $\mathsf{Poly}_\mathsf{Arr}(X)$. Both commitments $A$ has outputted is are linear combinations of the elements in $[g, g^\tau, g^{\tau^2}, \dots,g^{\tau^{n-1}}]$. $E$ is given these coefficients (since $A$ is an algebraic adversary) so $E$ can output the original polynomials.

$A$ then obtains the random challenge $\zeta$ (using strong Fiat-Shamir). By the binding property of KZG commitments, $\mathsf{Poly}_\mathsf{Arr}(\zeta)$, can only feasibliy be opened to one value. For $A$ to have the verifier accept, they must send a proof that $Q(\zeta)$ opens to $Q(\zeta) = \frac{Y_\mathsf{Vanish1}}{(\zeta^n - 1)}$. This means being able to send $g^{q(\tau)}$ where $q(\tau) = \frac{Q(\tau) - Q(\zeta)}{\tau - \zeta}$ and $Q(\zeta) = \frac{Y_\mathsf{Vanish1}}{(\zeta^n - 1)}$. Since $Q(\tau)$ and $Q(\zeta)$ are known, this implies knowing $g^{\frac{1}{\tau - \zeta}}$. Thus the prover would have found $\langle\zeta,g^{\frac{1}{\tau - \zeta}}\rangle$, which is the t-SDH problem. We have shown that creating an accepting proof reduces to the t-SDH, so $A$'s probability of success is negligible.

### Zero-Knowledge

We prove that the above protocol is zero-knowledge when $\mathsf{PolyCommit}_\mathsf{Ped}$ (from the KZG paper) is used for the polynomial commitments. We assume the product check is zero-knowledge (it is proven in [mult3](./mult3)), and conduct a proof for the rest of the protocol. We do so by constructing a probabilistic polynomial time simulator $S$ that knows the trapdoor $\tau$, which, for every (possibly malicious) verifier $V$, can output a view of the execution of the protocol that is indistinguishable from the view produced by the real execution of the program.

The simulator $S$ choose arbitrary values for ${\mathsf{Poly}_\mathsf{Arr_1}(\tau)}$ and ${\mathsf{Poly}_\mathsf{Arr_2}(\tau)}$, then computes $g^{\mathsf{Poly}_\mathsf{Arr_1}(\tau)}$ and $g^{\mathsf{Poly}_\mathsf{Arr_2}(\tau)}$ to output as the commitments $ K_\mathsf{Arr_1}$ and $K_\mathsf{Arr_1}$. $S$ then generates the random challenge $r$ (by strong Fiat-Shamir). They then choose arbitrary values for ${\mathsf{Poly}_\mathsf{Arr_1'}(\tau)}$ and ${\mathsf{Poly}_\mathsf{Arr_2'}(\tau)}$, then computes $g^{\mathsf{Poly}_\mathsf{Arr_1'}(\tau)}$ and $g^{\mathsf{Poly}_\mathsf{Arr_2'}(\tau)}$ to output as the commitments $ K_\mathsf{Arr_1'}$ and $K_\mathsf{Arr_1'}$.

$S$ then generates the challenge evaluation pount $\rho$ (by strong Fiat-Shamir) and computes $Q(\tau)$ using $\rho$ and the values they chose for ${\mathsf{Poly}_\mathsf{Arr_1}(\tau)}$, ${\mathsf{Poly}_\mathsf{Arr_2}(\tau)}$, ${\mathsf{Poly}_\mathsf{Arr_1'}(\tau)}$, and ${\mathsf{Poly}_\mathsf{Arr_2'}(\tau)}$. $S$ outputs the commitment $K_Q = g^{Q(\tau)}$.

Now, $S$ generates the second random challenge point $\zeta$ (which we assume is not in $\mathcal{H}_\kappa$; if it is in $\mathcal{H}_\kappa$, $S$ simply restarts and runs from the beginning). This is once again by strong Fiat-Shamir. $S$ then create fake opening proofs for ${\mathsf{Poly}_\mathsf{Arr_1}(\zeta)}$, ${\mathsf{Poly}_\mathsf{Arr_2}(\zeta)}$, ${\mathsf{Poly}_\mathsf{Arr_1'}(\zeta)}$, and ${\mathsf{Poly}_\mathsf{Arr_2'}(\zeta)}$, to arbitrary values. This is done using the knowledge of $\tau$, calculating the respective witness $q(\tau) = \frac{{f(\tau) - f(\zeta)}}{\tau - \zeta}$ for each of the polynomials.

Finally, $S$ creates a fake opening proof for $Q(\zeta) = \frac{Y_\mathsf{Vanish1} + \rho Y_\mathsf{Vanish2}}{(\zeta^n - 1)}$. This is done using knowledge of $\tau$ to calculate an accepting witness $q(\tau)$, as above. This means that $Y_\mathsf{Zero}$ will be zero, and the transcript will be accepted by the verifier. It is indistinguishable from a transcript generates from a real execution, since $\mathsf{PolyCommit}_\mathsf{Ped}$ has the property of Indistinguishability of Commitments due to the randomization by $h^{\hat{\phi}(x)}$. 