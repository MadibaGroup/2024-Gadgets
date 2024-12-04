# Shuffle (Type 1)

## Recap of types

| Type                   | Description                                            | Recap                                                        | This |
| ---------------------- | ------------------------------------------------------ | :----------------------------------------------------------- | ---- |
| [shuffle1](./shuffle1) | $\mathsf{Arr}_2=\mathsf{Permute}(\mathsf{Arr}_1)$      | Array $\mathsf{Arr}_2$ is a shuffle of $\mathsf{Arr}_1$ for some undisclosed permutation $\pi$. | ✅    |
| [shuffle2](./shuffle2) | $\mathsf{Arr}_2=\mathsf{Permute}(\mathsf{Arr}_1 ,\pi)$ | Array $\mathsf{Arr}_2$ is a shuffle of $\mathsf{Arr}_1$ under a disclosed permutation $\pi$. |      |

## Relation

$ \mathcal{R}_{\mathtt{shuffle1}} := \left\{ \begin{array}{l} (K_\mathsf{Arr_1},K_\mathsf{Arr2}) \end{array} \middle | \begin{array}{l} \mathsf{Arr_1} = [a_{(1,0)}, a_{(1,1)}, a_{(1,2)}, \dots, a_{(1,n-1)}],\\ \mathsf{Arr_2} = [a_{(2,0)}, a_{(2,1)}, a_{(2,2)}, \dots, a_{(2,n-1)}], \\  \mathsf{Poly}_\mathsf{Arr_1}=\mathsf{FFT.Interp}(\omega,\mathsf{Arr_1}), \\ \mathsf{Poly}_\mathsf{Arr_2}=\mathsf{FFT.Interp}(\omega,\mathsf{Arr_2}), \\ K_\mathsf{Arr_1}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_1}),\\ K_\mathsf{Arr_2}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_2}), \end{array} \right\} $

## Intuition

The prover ($\mathcal{P}$) holds 2 arrays, $\mathsf{Arr_1 }$ and $\mathsf{Arr_2}$, of $n$ integers from $\mathbb{Z}_q$: $[a_0, a_1, a_2, \dots, a_{n-1}]$. It will produce a succinct (independent of $n$) proof that $\mathsf{Arr}_2$ is a shuffle of $\mathsf{Arr}_1$ for some undisclosed permutation $\pi$. The prover will encode the two arrays into polynomials, $\mathsf{Poly}_\mathsf{Arr_1}$ and $\mathsf{Poly}_\mathsf{Arr_2}$ (using [evaluation points](../background/poly-iop) on the domain $\mathcal{H}_\kappa$) and commit to them as $K_\mathsf{Arr_1}$ and $K_\mathsf{Arr_2}$.  The verifier ($\mathcal{V}$) cannot check either array directly (they may contain secret information, and even if they do not, it is too long to check) so the verifier only sees $K_\mathsf{Arr_1}$ and $K_\mathsf{Arr_2}$.

One idea to check that $\mathsf{Arr_2}$ is a permutation of $\mathsf{Arr_1}$ might be to perform a product check on the two arrays. If the permutation relation holds, then the products will be equal; however, many arrays can have their entries multiply to the same number, without necessarily containing the same elements. 

Instead, the prover constructs two new arrays, $\mathsf{Arr_1}'$ and $\mathsf{Arr_2}'$, where $\mathsf{Arr_j}'$ contains the points $ \{r - \mathsf{Arr_j}[i]  \}_{i \in [0, n-1]}$ for $r$ a random field element. Then, a product check is run on these two arrays. One way to understand why this works is to think of it as creating two auxiliary polynomials, ${\mathsf{Poly}}_\mathsf{Arr_1'}$ and ${\mathsf{Poly}}_\mathsf{Arr_2'}$, where $\mathsf{Poly}_\mathsf{Arr_j'}(X) = \prod^{n-1}_{i = 1}(X - \mathsf{Arr_j}[i])$. If ${\mathsf{Poly}}_\mathsf{Arr_1'}$ = $\mathsf{Poly}_\mathsf{Arr_2'}$, then they have the same factorization. This means that $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$ must contain the same elements (in possibly different orders); in other words, $\mathsf{Arr}_2$ is a permutation of $\mathsf{Arr}_1$. To check this equality, a random challenge point $r$ is generated and the products are checked at that point. If they are equal at that point then (with overwhelming probability) the polynomials are equal.

In addition to demonstrating the equality of the product of $\mathsf{Arr_1}'$ and $\mathsf{Arr_2}'$, it must also be shown that these two arrays are defined correctly in terms of the original arrays. In other words, it must be shown that $\mathsf{Arr_j}'[i]= r - \mathsf{Arr_j}[i]$ for $i \in [0, n-1]$. Once this, in addition to the product check, has been done, we have shown that $\mathsf{Arr}_2$ is a shuffle of $\mathsf{Arr}_1$ for some undisclosed permutation $\pi$. 

## Protocol Details

### Array Level

* $\mathcal{P}$ holds an array $\mathsf{Arr_1} = [a_{(1,0)}, a_{(1,1)}, a_{(1,2)}, \dots, a_{(1,n-1)}]$ of $n$ integers ($a_{(1,i)} \in \mathbb{Z}_q$)
* $\mathcal{P}$ holds an array $\mathsf{Arr_2} = [a_{(2,0)}, a_{(2,1)}, a_{(2,2)}, \dots, a_{(2,n-1)}]$ of $n$ integers ($a_{(2,i)} \in \mathbb{Z}_q$)
* $\mathcal{P}$ generates the random challenge $r$ and computes $\mathsf{Arr_j}'$ as follows for $j \in [1,2]$:
  * $\mathsf{Arr_j}'[i]= r - \mathsf{Arr_j}[i]$

### Polynomial Level

We assume that $\mathsf{Arr_1}$, $\mathsf{Arr_2}$, $\mathsf{Arr_1'}$, and $\mathsf{Arr_2'}$ are encoded as the y-coordinates into a univariant polynomial where the x-coordinates (called the domain $\mathcal{H}_\kappa$) are chosen as the multiplicative group of order $\kappa$ with generator $\omega\in\mathbb{G}_\kappa$ (see [Background](../background/poly-iop.md) for more). In short, $\omega^0$ is the first element and $\omega^{\kappa-1}$ is the last element of $\mathcal{H}_\kappa$. If $\kappa$ is larger than the length of the arrays, the arrays can be padded with elements all of value 1 (or any other value, as long as it is the same for both arrays).

Recall the two steps we want to prove: 

1. $\prod^{n-1}_{i=0}\{r - \mathsf{Arr_1[i]}\} = \prod^{n-1}_{i=0}\{r - \mathsf{Arr_2[i]}\}$ 
2. $\mathsf{Arr_j}'[i]= r - \mathsf{Arr_j}[i]$ for $j \in [1,2]$, $0 \leq 1 \leq n-1$

The first step is done as a [mult3](./mult3) product check, and we write the second step as two constraints in polynomial form. From this point on we focus on the polynomial details of the second step.

1. For all $X$ from $\omega^0$ to $\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Arr_1'}(X) = (r - \mathsf{Poly}_\mathsf{Arr_1}(X))$ 
2. For all $X$ from $\omega^0$ to $\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Arr_2'}(X) = (r - \mathsf{Poly}_\mathsf{Arr_2}(X))$ 

We adjust each of these constraints to show an equality with 0 and label them:

1. $\mathsf{Poly}_\mathsf{Vanish1}(X)= \mathsf{Poly}_\mathsf{Arr_1'}(X) - (r - \mathsf{Poly}_\mathsf{Arr_1}(X)) = 0$
2. $\mathsf{Poly}_\mathsf{Vanish2}(X)= \mathsf{Poly}_\mathsf{Arr_2'}(X) - (r - \mathsf{Poly}_\mathsf{Arr_2}(X)) = 0$

This equation is true for every value of $X \in \mathcal{H}_\kappa$ (but not necessarily true outside of these values). To show this, we divide each polynomial by  $X^\kappa - 1$, which is a minimal vanishing polynomial for $\mathcal{H}_\kappa$ that does not require interpolation to create. If the quotients are polynomials (and not rational functions), then $\mathsf{Poly}_\mathsf{Vanish}(X)$ must be vanishing on $\mathcal{H}_\kappa$ too. Specifically, the prover computes:

1. $Q_1(X) = \frac{\mathsf{Poly}_\mathsf{Vanish1}(X)}{X^\kappa - 1}$
2. $Q_2(X) = \frac{\mathsf{Poly}_\mathsf{Vanish2}(X)}{X^\kappa - 1}$

We can replace polynomials $Q_1(X)$ and $Q_2(X)$ with a single polynomial $Q(X)$. We can do this because both constraints have the same format: $\mathsf{Poly}_\mathsf{Vanish_i}(X)=0$. The batching technique is to create a new polynomial with both $\mathsf{Poly}_\mathsf{Vanish_i}(X)$ values as coefficients. If and (overwhelmingly) only if both are vanishing, then so will the new polynomial. This polynomial will be evaluated at a random challenge point $\rho$ selected after the commitments to the earlier polynomials are fixed. 

$Q(X) = \frac{\mathsf{Poly}_\mathsf{Vanish1}(X) + \mathsf{Poly}_\mathsf{Vanish2}(X) \rho}{X^\kappa - 1}$

By rearranging, we can get $\mathsf{Poly}_\mathsf{Zero}(X)$ as a true zero polynomial (zero at every value both in $\mathcal{H}_\kappa$ and outside of it):

$\mathsf{Poly}_\mathsf{Zero}(X)=\mathsf{Poly}_\mathsf{Vanish1}(X) + \rho \mathsf{Poly}_\mathsf{Vanish2}(X) - Q(X)\cdot (X^\kappa - 1)=0$

Ultimately the shuffle1 argument will satisfy the following constraints at the Commitment Level:

1. Show $Q(X)$ exists (as a polynomial that evenly divides the divisor)
2. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is correctly constructed from $\mathsf{Poly}_\mathsf{Arr_1}(X)$, $\mathsf{Poly}_\mathsf{Arr_2}(X)$, $\mathsf{Poly}_\mathsf{Arr_1'}(X)$, and $\mathsf{Poly}_\mathsf{Arr_2'}(X)$
3. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is the zero polynomial

In addition, it will show that $\prod^{n-1}_{i=0}(r - \mathsf{Arr_1}[i]) = \prod^{n-1}_{i=0}(r - \mathsf{Arr_2}[i])$  using a [mult3](./mult3) product check.

### Commitment Level

The verifier will never see the arrays or polynomials themselves. They are undisclosed because they either (i) contain private data or (ii) they are too large to examine and maintain a succinct proof system. Instead the prover will use commitments.

The prover will create a transcript for the product check, as described in [mult3](./mult3). Below, we give details specific to the second step, showing that $\mathsf{Arr_j}'[i]= r - \mathsf{Arr_j}[i]$ for $j \in [1,2]$, $0 \leq 1 \leq n-1$.

The prover will write the following commitments to the transcript:

* $K_\mathsf{Arr_1}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_1}(X))$
* $K_\mathsf{Arr_2}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_2}(X))$

The prover will generate a random challenge evaluation point (using strong Fiat-Shamir) on the polynomials that is outside of $\mathcal{H}_\kappa$. Call this point $r$. It will use this point to define the sets $ \{r - \mathsf{Poly}_\mathsf{Arr_j}(a)  \}_{a \in \mathcal{H}_\kappa}$ and run the product check. It will write the product check into the transcript. However, here we focus only on what is relevant to the second step, the point $r$ and the following polynomials, which it also writes to the transcript:

* $r$
* $K_\mathsf{Arr_1'}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_1'}(X))$
* $K_\mathsf{Arr_2'}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_2'}(X))$​

The prover will generate a random challenge evaluation point (using strong Fiat-Shamir) on the polynomial that is outside of $\mathcal{H}_\kappa$. Call this point $\rho$. It will be used by the prover to create polynomial $Q(X)$ (see above) and the prover will write to the transcript: 

* $\rho$
* $K_Q=\mathsf{KZG.Commit}(Q(X))$

The prover will generate a random challenge evaluation point (using strong Fiat-Shamir) on the polynomials that is outside of $\mathcal{H}_\kappa$. Call this point $\zeta$. The prover will write the point and opening proofs to the transcript:

* $\zeta$
* $\mathsf{Poly}_\mathsf{Arr_1}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr_1},\zeta)$
* $\mathsf{Poly}_\mathsf{Arr_2}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr_2},\zeta)$
* $\mathsf{Poly}_\mathsf{Arr_1'}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr_1'},\zeta)$
* $\mathsf{Poly}_\mathsf{Arr_2'}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr_2'},\zeta)$
* $Q(\zeta)=\mathsf{KZG.Open}(K_Q,\zeta)$

To check the proof, the verifier uses the transcript to construct the value $Y_\mathsf{Zero}$ as follows:

* $Y_\mathsf{Vanish1}= \mathsf{Poly}_\mathsf{Arr_1'}(\zeta) - (r - \mathsf{Poly}_\mathsf{Arr_1}(\zeta))$ 
* $Y_\mathsf{Vanish2}= \mathsf{Poly}_\mathsf{Arr_2'}(\zeta) - (r - \mathsf{Poly}_\mathsf{Arr_2}(\zeta))$
* $Y_\mathsf{Zero}=Y_\mathsf{Vanish1} + \rho Y_\mathsf{Vanish2} - Q(\zeta)\cdot (\zeta^\kappa - 1)$

Finally, if the constraint system is true, the following constraint will be true (and will be false otherwise with overwhelming probability, due to the Schwartz-Zippel lemma on $\zeta$) :

* $Y_\mathsf{Zero}\overset{?}{=}0$

## Implementations



## Security Proof

### Completeness

We assume completeness of the product check (it is proven in [mult3](./mult3)) and conduct a proof of completeness for the rest of the protocol.

If $Y_\mathsf{Zero}$ is zero, then $\mathcal{V}$ will accept. Therefore, to show completeness, we show that any prover who holds $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$ such that  $\mathsf{Arr}_2=\mathsf{Permute}(\mathsf{Arr}_1)$, can follow the steps outlined in the above protocol and the resulting $Y_\mathsf{Zero}$ will be equal to zero.  To see this, observed that $Y_\mathsf{Zero}$

$= Y_\mathsf{Vanish1} + \rho Y_\mathsf{Vanish2} - Q(\zeta)\cdot (\zeta^\kappa - 1)$

$= [\mathsf{Poly}_\mathsf{Arr_1'}(\zeta) - (r - \mathsf{Poly}_\mathsf{Arr_1}(\zeta))] + \rho [\mathsf{Poly}_\mathsf{Arr_2'}(\zeta) - (r - \mathsf{Poly}_\mathsf{Arr_2}(\zeta))] - Q(\zeta) \cdot (\zeta^\kappa - 1)$

$= [\mathsf{Poly}_\mathsf{Arr_1'}(\zeta) - (r - \mathsf{Poly}_\mathsf{Arr_1}(\zeta))] + \rho [\mathsf{Poly}_\mathsf{Arr_2'}(\zeta) - (r - \mathsf{Poly}_\mathsf{Arr_2}(\zeta))] - \frac{\mathsf{Poly}_\mathsf{Vanish1}(\zeta) + \rho \mathsf{Poly}_\mathsf{Vanish2}(\zeta)}{X^\kappa - 1} \cdot (\zeta^\kappa - 1)$

$= [\mathsf{Poly}_\mathsf{Arr_1'}(\zeta) - (r - \mathsf{Poly}_\mathsf{Arr_1}(\zeta))] + \rho [\mathsf{Poly}_\mathsf{Arr_2'}(\zeta) - (r - \mathsf{Poly}_\mathsf{Arr_2}(\zeta))] -  [\mathsf{Poly}_\mathsf{Arr_1'}(\zeta) - (r - \mathsf{Poly}_\mathsf{Arr_1}(\zeta)) + \rho[\mathsf{Poly}_\mathsf{Arr_2'}(\zeta) - (r - \mathsf{Poly}_\mathsf{Arr_2}(\zeta))]]$

$= 0$

Where the third equality relies on the fact that $\mathsf{Poly}_\mathsf{Vanish1}(X) + \rho \mathsf{Poly}_\mathsf{Vanish2}(X)$ is divisible by $X^\kappa -1$. This is true if $\mathsf{Poly_{Vanish1}}(\zeta)$ and $\mathsf{Poly_{Vanish2}}(\zeta)$ are vanishing on $\mathcal{H}_\kappa$, i.e. if $\mathsf{Poly}_\mathsf{Arr_1'}(X) - (r - \mathsf{Poly}_\mathsf{Arr_1}(X)) = 0$ and $\mathsf{Poly}_\mathsf{Arr_2'}(X) - (r - \mathsf{Poly}_\mathsf{Arr_2}(X)) = 0$, $X \in \mathcal{H}_\kappa$. This is true if $\mathsf{Arr}_1'[i] - (r - \mathsf{Arr}_1[i]) = 0$ and $\mathsf{Arr}_2'[i] - (r - \mathsf{Arr}_1[i]) = 0$, $\forall 0 \leq i\leq \kappa$, since $\mathsf{Poly_j}(\omega^i) = \mathsf{Arr_j}[i] \space \forall i \in [0, \kappa - 1]$ and $\mathsf{Poly_j}'(\omega^i) = \mathsf{Arr_j}'[i] \space \forall i \in [0, \kappa - 1]$. But this is precisely how the honest prover defines $\mathsf{Arr}_1'$ and $\mathsf{Arr}_2'$, so the $Y_\mathsf{Zero}$ it creates by following the protocol is zero, and the transcript will be accepted.

### Soundness

We prove knowledge soundness in the Algebraic Group Model (AGM). We assume soundness of the product check (it is proven in [mult3](./mult3)) and conduct a proof of soundness for the rest of the protocol. To do so, we must prove that there exists an efficient extractor $\mathcal{E}$ such that for any algebraic adversary $\mathcal{A}$, the probability of $\mathcal{A}$ winning the following game is $\mathsf{negl}(\lambda)$.

1. Given $[g, g^\tau, g^{\tau^2}, \dots,g^{\tau^{n-1}}]$ $\mathcal{A}$ outputs commitments to $\mathsf{Poly}_\mathsf{Arr_1}(X)$, $\mathsf{Poly}_\mathsf{Arr_2}(X)$, $\mathsf{Poly}_\mathsf{Arr_1'}(X)$, $\mathsf{Poly}_\mathsf{Arr_2'}(X)$, $Q(X)$

2. $\mathcal{E}$, given access to $\mathcal{A}$'s outputs from the previous step, outputs $\mathsf{Poly}_\mathsf{Arr_1}(X)$, $\mathsf{Poly}_\mathsf{Arr_2}(X)$, $\mathsf{Poly}_\mathsf{Arr_1'}(X)$, $\mathsf{Poly}_\mathsf{Arr_2'}(X)$, $Q(X)$

3. $\mathcal{A}$ plays the part of the prover in showing that $Y_{\mathsf{Zero}}$ is zero at a random challenge $\zeta$

4. $\mathcal{A}$ wins if: 

   i) $\mathcal{V}$ accepts at the end of the protocol

   ii) $\mathsf{Arr}_2 \neq \mathsf{Permute}(\mathsf{Arr}_1)$

Our proof is as follows:

For the second win condition to be fulfilled, there must be some $a \in \mathsf{Arr_2}, a \notin \mathsf{Arr_1}$, or vice versa. Since $\mathsf{Arr_1}$ and $\mathsf{Arr_2}$ have different entries, $\prod^{n-1}_{i = 1}(X - \mathsf{Arr_1}[i])$ and $\prod^{n-1}_{i = 1}(X - \mathsf{Arr_2}[i])$ have different factorizations and are thus different polynomials. By the Schwartz-Zippel lemma, there is negligible probability that they are equal at $r$ (thus the product check will fail). Any strategy to increase this probability to greater than negligible means $\mathcal{A}$ must pass in $\mathsf{Arr_j'}$ such that $\mathsf{Arr_j'}[i] \neq r - \mathsf{Arr_j}[i]$ for some index $i$ and $j \in [1, 2]$. But then $\mathsf{Poly}_\mathsf{Vanish_j}(X)$ is not vanishing on $\mathcal{H}_\kappa$, so $Q(X)$ is not a polynomial (it is a rational function). This means that $\mathcal{A}$ cannot calculated the correct commitment value $g^{Q(\tau)}$ without solving the t-SDH. Thus, $\mathcal{A}$ chooses an arbitrary value for $Q(\tau)$ and writes $K_Q = g^{Q(\tau)}$. Before this, it also writes commitments to  $\mathsf{Poly}_\mathsf{Arr_1}(X)$, $\mathsf{Poly}_\mathsf{Arr_2}(X)$, $\mathsf{Poly}_\mathsf{Arr_1'}(X)$, and $\mathsf{Poly}_\mathsf{Arr_2'}(X)$. All commitments $\mathcal{A}$ has written are linear combinations of the elements in $[g, g^\tau, g^{\tau^2}, \dots,g^{\tau^{n-1}}]$. $\mathcal{E}$ is given these coefficients (since $\mathcal{A}$ is an algebraic adversary) so $\mathcal{E}$ can output the original polynomials.

$\mathcal{A}$ then obtains the random challenge $\zeta$ (using strong Fiat-Shamir). By the binding property of KZG commitments, $\mathsf{Poly}_\mathsf{Arr_1}(\zeta)$, $\mathsf{Poly}_\mathsf{Arr_2}(\zeta)$, $\mathsf{Poly}_\mathsf{Arr_1'}(\zeta)$, and $\mathsf{Poly}_\mathsf{Arr_2'}(\zeta)$ can each only feasibly be opened to one value. For $\mathcal{A}$ to have the verifier accept, it must produce a proof that $Q(\zeta)$ opens to $Q(\zeta) = \frac{Y_\mathsf{Vanish1} + \rho Y_\mathsf{Vanish2}}{(\zeta^\kappa - 1)}$. This means being able to produce $g^{q(\tau)}$ where $q(\tau) = \frac{Q(\tau) - Q(\zeta)}{\tau - \zeta}$ and $Q(\zeta) = \frac{Y_\mathsf{Vanish1} + \rho Y_\mathsf{Vanish2}}{(\zeta^\kappa - 1)}$. Since $Q(\tau)$ and $Q(\zeta)$ are known, this implies knowing $g^{\frac{1}{\tau - \zeta}}$. Thus $\mathcal{A}$ would have found $\langle\zeta,g^{\frac{1}{\tau - \zeta}}\rangle$, which is the t-SDH problem. We have shown that creating an accepting proof reduces to the t-SDH, so $\mathcal{A}$'s probability of success is negligible.

### Zero-Knowledge

We prove that the above protocol is zero-knowledge when $\mathsf{PolyCommit}_\mathsf{Ped}$ (from the KZG paper) is used for the polynomial commitments. We assume the product check is zero-knowledge (it is proven in [mult3](./mult3)), and conduct a proof for the rest of the protocol. We do so by constructing a probabilistic polynomial time simulator $\mathcal{S}$ that knows the trapdoor $\tau$, which, for every (possibly malicious) verifier $\mathcal{V}$, can output a view of the execution of the protocol that is indistinguishable from the view produced by the real execution of the program.

The simulator $\mathcal{S}$ chooses arbitrary values for ${\mathsf{Poly}_\mathsf{Arr_1}(\tau)}$ and ${\mathsf{Poly}_\mathsf{Arr_2}(\tau)}$, then computes $g^{\mathsf{Poly}_\mathsf{Arr_1}(\tau)}$ and $g^{\mathsf{Poly}_\mathsf{Arr_2}(\tau)}$ to write as the commitments $ K_\mathsf{Arr_1}$ and $K_\mathsf{Arr_1}$. $\mathcal{S}$ then generates the random challenge $r$ (by strong Fiat-Shamir). It chooses arbitrary values for ${\mathsf{Poly}_\mathsf{Arr_1'}(\tau)}$ and ${\mathsf{Poly}_\mathsf{Arr_2'}(\tau)}$, then computes $g^{\mathsf{Poly}_\mathsf{Arr_1'}(\tau)}$ and $g^{\mathsf{Poly}_\mathsf{Arr_2'}(\tau)}$ to write as the commitments $ K_\mathsf{Arr_1'}$ and $K_\mathsf{Arr_1'}$. It creates a view of the product check as described in the zero-knowledge proof for [mult3](./mutl3).

$\mathcal{S}$ generates the challenge evaluation point $\rho$ (by strong Fiat-Shamir) and computes $Q(\tau)$ using $\rho$ and the values it chose for ${\mathsf{Poly}_\mathsf{Arr_1}(\tau)}$, ${\mathsf{Poly}_\mathsf{Arr_2}(\tau)}$, ${\mathsf{Poly}_\mathsf{Arr_1'}(\tau)}$, and ${\mathsf{Poly}_\mathsf{Arr_2'}(\tau)}$. $\mathcal{S}$ outputs the commitment $K_Q = g^{Q(\tau)}$.

Now, $\mathcal{S}$ generates the random challenge point $\zeta$ (which we assume is not in $\mathcal{H}_\kappa$; if it is in $\mathcal{H}_\kappa$, $\mathcal{S}$ simply restarts and runs from the beginning). This is once again by strong Fiat-Shamir. $\mathcal{S}$ then create fake opening proofs for ${\mathsf{Poly}_\mathsf{Arr_1}(\zeta)}$, ${\mathsf{Poly}_\mathsf{Arr_2}(\zeta)}$, ${\mathsf{Poly}_\mathsf{Arr_1'}(\zeta)}$, and ${\mathsf{Poly}_\mathsf{Arr_2'}(\zeta)}$, to arbitrary values. This is done using the knowledge of $\tau$, calculating the respective witness $q(\tau) = \frac{{f(\tau) - f(\zeta)}}{\tau - \zeta}$ for each of the polynomials.

Finally, $\mathcal{S}$ creates a fake opening proof for $Q(\zeta) = \frac{Y_\mathsf{Vanish1} + \rho Y_\mathsf{Vanish2}}{(\zeta^\kappa - 1)}$. This is done using knowledge of $\tau$ to calculate an accepting witness $q(\tau)$, as above. This means that $Y_\mathsf{Zero}$ will be zero, and the transcript will be accepted by the verifier. It is indistinguishable from a transcript generates from a real execution, since $\mathsf{PolyCommit}_\mathsf{Ped}$ has the property of Indistinguishability of Commitments due to the randomization by $h^{\hat{\phi}(x)}$. 