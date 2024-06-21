# Shuffle (Type 2)

## Recap of types

| Type                   | Description                                            | Recap                                                        | This |
| ---------------------- | ------------------------------------------------------ | :----------------------------------------------------------- | ---- |
| [shuffle1](./shuffle1) | $\mathsf{Arr}_2=\mathsf{Permute}(\mathsf{Arr}_1)$      | Array $\mathsf{Arr}_2$ is a shuffle of $\mathsf{Arr}_1$ for some undisclosed permutation $\pi$. |      |
| [shuffle2](./shuffle2) | $\mathsf{Arr}_2=\mathsf{Permute}(\mathsf{Arr}_1 ,\pi)$ | Array $\mathsf{Arr}_2$ is a shuffle of $\mathsf{Arr}_1$ under a disclosed permutation $\pi$. | ✅    |

## Relation

$\mathcal{R}_{\mathtt{shuffle2}} := \left\{ \begin{array}{l} (K_\mathsf{Arr_1},K_\mathsf{Arr2}, K_\pi) \end{array} \middle | \begin{array}{l} \mathsf{Arr_1} = [a_{(1,0)}, a_{(1,1)}, a_{(1,2)}, \dots, a_{(1,n-1)}],\\ \mathsf{Arr_2} = [a_{(2,0)}, a_{(2,1)}, a_{(2,2)}, \dots, a_{(2,n-1)}], \\ \mathsf{Arr_\pi} = [\pi(\omega^0), \pi(\omega^1), \pi(\omega^2), \dots, \pi(\omega^{n-1})], \\ \mathsf{Poly}_\mathsf{Arr_1}=\mathsf{FFT.Interp}(\omega,\mathsf{Arr_1}), \\ \mathsf{Poly}_\mathsf{Arr_2}=\mathsf{FFT.Interp}(\omega,\mathsf{Arr_2}), \\ \mathsf{Poly_\pi} = \mathsf{FFT.Interp}(\omega, \mathsf{Arr_\pi}), \\ K_\mathsf{Arr_1}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_1}),\\ K_\mathsf{Arr_2}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_2}),\\K_\pi = \mathsf{KZG.Commit(Poly_{\pi})} \end{array} \right\}$

## Intuition

The prover ($\mathcal{P}$) holds 2 arrays, $\mathsf{Arr_1 }$ and $\mathsf{Arr_2}$, of $n$ integers from $\mathbb{Z}_q$: $[a_0, a_1, a_2, \dots, a_{n-1}]$. It also holds an array $\mathsf{Arr_\pi}$, which represents the disclosed permutation $\pi$. It will produce a succinct (independent of $n$) proof that $\mathsf{Arr}_2$ is a shuffle of $\mathsf{Arr}_1$ under the disclosed permutation $\pi$. The prover will encode the three arrays into polynomials, $\mathsf{Poly}_\mathsf{Arr_1}$, $\mathsf{Poly}_\mathsf{Arr_2}$, and $\mathsf{Poly_\pi}$ (using [evaluation points](../background/poly-iop) on the domain $\mathcal{H}_\kappa$) and commit to them as $K_\mathsf{Arr_1}$, $K_\mathsf{Arr_2}$, and $K_\pi$.  The verifier ($\mathcal{V}$) cannot check any array directly ($\mathsf{Arr_1 }$ and $\mathsf{Arr_2}$ may contain secret information, and even if they do not, it is too long to check) so the verifier only sees $K_\mathsf{Arr_1}$, $K_\mathsf{Arr_2}$ and $K_\pi$.

The idea behind this check is that if $(\mathsf{Arr_\pi}[i], \mathsf{Arr_2}[i]) = (i, \mathsf{Arr_1}[i])$ for all $0 \leq i \leq n-1$, then $\mathsf{Arr}_2=\mathsf{Permute}(\mathsf{Arr}_1 ,\pi)$. To gain some intuition on why this is true, pair up tuples from the left and right hand sides of the equation by matching the first entries. Then, if each pair is equal, it means that $\mathsf{Arr_2}[i] = \mathsf{Arr_1}[\mathsf{Arr_\pi}[i]]$ for $0 \leq i \leq n-1$. In other words, $\mathsf{Arr}_2=\mathsf{Permute}(\mathsf{Arr}_1 ,\pi)$.

To check that $(\mathsf{Arr_\pi}[i], \mathsf{Arr_2}[i]) = (i, \mathsf{Arr_1}[i])$ for all $0 \leq j \leq n-1$, we use a similar trick to [shuffle1](./shuffle). The prover constructs two arrays:  $\mathsf{Arr_1'} = \{ r - s\cdot i - \mathsf{Arr_1}[i]\}_{i \in [0, n-1]}$ and $\mathsf{Arr_2'} = \{ r - s\cdot\mathsf{Arr_\pi}[i] - \mathsf{Arr_2}[i]\}_{i \in [0, n-1]}$ for random field elements $r, s$. Then, a product check is conducted on the two arrays. One way to understand why this works is to think of it as creating two auxilary polynomials, ${\mathsf{Poly}}_\mathsf{Arr_1'}$ and ${\mathsf{Poly}}_\mathsf{Arr_2'}$. Here, $\mathsf{Poly}_\mathsf{Arr_1'}(X) = \prod^{n-1}_{i = 1}(X - Y\cdot i - \mathsf{Arr_1}[i])$  and $\mathsf{Poly}_\mathsf{Arr_2'}(X) = \prod^{n-1}_{i = 1}(X - Y\cdot \mathsf{Arr_\pi}[i] - \mathsf{Arr_2}[i])$. If ${\mathsf{Poly}}_\mathsf{Arr_1'}$ = $\mathsf{Poly}_\mathsf{Arr_2'}$, then they have the same factorization. This means that $\mathsf{Arr}_1'$ and $\mathsf{Arr}_2'$ must contain the same elements (in possibly different orders). By the same intuition of matching first entries as above (in this case, we are pairing up factors where $i$ in $\mathsf{Poly}_\mathsf{Arr_1'}(X)$ equals $\mathsf{Arr_\pi}[i]$ in $\mathsf{Poly}_\mathsf{Arr_2'}(X)$) this shows that $\mathsf{Arr}_2$ is a permutation of $\mathsf{Arr}_1$ under $\pi$. To check the equality of the two auxilary polynomials, random challenge values $r$ and $s$ are generated and the products are checked at that point. If they are equal at that point then (with overwhelming probabiltiy) the polynomials are equal.

In addition to demontrasting the equality of the product of $\mathsf{Arr_1}'$ and $\mathsf{Arr_2}'$, it must also be shown that these two arrays are defined correctly in terms of the original arrays. In other words, it must be shown that $\mathsf{Arr_1}'[i]= (i, \mathsf{Arr_1}[i])$ and $\mathsf{Arr_2}'[i]= (\mathsf{Arr_\pi}[i], \mathsf{Arr_2}[i])$. Once this, in addition to the product check, has been done, it has been shown that $\mathsf{Arr}_2$ is a shuffle of $\mathsf{Arr}_1$ under the disclosed permutation $\pi$. 

## Protocol Details

### Array Level

* $\mathcal{P}$ holds an array $\mathsf{Arr_1} = [a_{(1,0)}, a_{(1,1)}, a_{(1,2)}, \dots, a_{(1,n-1)}]$ of $n$ integers ($a_{(1,i)} \in \mathbb{Z}_q$)
* $\mathcal{P}$ holds an array $\mathsf{Arr_2} = [a_{(2,0)}, a_{(2,1)}, a_{(2,2)}, \dots, a_{(2,n-1)}]$ of $n$ integers ($a_{(2,i)} \in \mathbb{Z}_q$​)
* $\mathcal{P}$ holds an array $\mathsf{Arr_\pi} = [\pi(\omega^0), \pi(\omega^1), \pi(\omega^2), \dots, \pi(\omega^{n-1})]$ of $n$ integers ($a_{(2,i)} \in \mathbb{Z}_q$)
* $\mathcal{P}$ computes $\mathsf{Arr_1}'$ as follows:
  * $\mathsf{Arr_1}'[i]= r - s\cdot i - \mathsf{Arr_1}[i]$
* $\mathcal{P}$ generates the random challenge $r, s$ and computes $\mathsf{Arr_2}'$ as follows:
  * $\mathsf{Arr_2}'[i]= r - s\cdot \mathsf{Arr_\pi}[i] - \mathsf{Arr_2}[i]$

### Polynomial Level

We assume that $\mathsf{Arr_1}$, $\mathsf{Arr_2}$, $\mathsf{Arr_\pi}[i]$, $\mathsf{Arr_1'}$, and $\mathsf{Arr_2'}$ are encoded as the y-coordinates into a univariant polynomial where the x-coordinates (called the domain $\mathcal{H}_\kappa$) are chosen as the multiplicative group of order $\kappa$ with generator $\omega\in\mathbb{G}_\kappa$ (see [Background](../background/poly-iop.md) for more). In short, $\omega^0$ is the first element and $\omega^{\kappa-1}$ is the last element of $\mathcal{H}_\kappa$. If $\kappa$ is larger than the length of the arrays, the arrays can be padded with elements all of value 1 (or any other value, as long as it is the same for both arrays).

Recall the two components we want to prove. First, the product check: 

$\prod^{n-1}_{i = 1}(r - s\cdot i - \mathsf{Arr_1}[i]) = \prod^{n-1}_{i = 1}(r - s\cdot \mathsf{Arr_\pi}[i] - \mathsf{Arr_2}[i])$ 

As well as the two constraints:

1. $\mathsf{Arr_1}'[i]= r - s\cdot i - \mathsf{Arr_1}[i]$
2. $\mathsf{Arr_2}'[i]= r - s\cdot \mathsf{Arr_\pi}[i] - \mathsf{Arr_2}[i]$

The first component is done as a [mult3](./mult3) product check, and we write the second component in polynomial form. From this point on we focus on the polynomial details of the second component.

1. For all $X$ from $\omega^0$ to $\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Arr_1'}(X) = r - s\cdot X -  \mathsf{Poly}_\mathsf{Arr_1}(X)$ 
2. For all $X$ from $\omega^0$ to $\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Arr_2'}(X) = r - s\cdot \mathsf{Poly_\pi}(X) -  \mathsf{Poly}_\mathsf{Arr_2}(X)$ 

We adjust each of these constraints to show an equality with 0 and label them:

1. $\mathsf{Poly}_\mathsf{Vanish1}(X)= \mathsf{Poly}_\mathsf{Arr_1'}(X) - (r - s\cdot X - \mathsf{Poly}_\mathsf{Arr_1}(X)) = 0$
2. $\mathsf{Poly}_\mathsf{Vanish2}(X)= \mathsf{Poly}_\mathsf{Arr_2'}(X) - (r - s\cdot \mathsf{Poly_\pi}(X) - \mathsf{Poly}_\mathsf{Arr_2}(X)) = 0$

This equation is true for every value of $X \in \mathcal{H}_\kappa$ (but not necessarily true outside of these values). To show this, we divide each polynomial by  $X^\kappa - 1$, which is a minimal vanishing polynomial for $\mathcal{H}_\kappa$ that does not require interpolation to create. If the quotients are polynomials (and not rational functions), then $\mathsf{Poly}_\mathsf{Vanish}(X)$ must be vanishing on $\mathcal{H}_\kappa$ too. Specifically, the prover computes:

1. $Q_1(X) = \frac{\mathsf{Poly}_\mathsf{Vanish1}(X)}{X^\kappa - 1}$
2. $Q_2(X) = \frac{\mathsf{Poly}_\mathsf{Vanish2}(X)}{X^\kappa - 1}$

We can replace polynomials $Q_1(X)$ and $Q_2(X)$ with a single polynomial $Q(X)$. We can do this because both constraints have the same format: $\mathsf{Poly}_\mathsf{Vanish_i}(X)=0$. The batching technique is to create a new polynomial with both $\mathsf{Poly}_\mathsf{Vanish_i}(X)$ values as coefficients. If and (overwhelmingly) only if both are vanishing, then so will the new polynomial. This polynomial will be evaluated at a random challenge point $\rho$ selected after the commitments to the earlier polynomials are fixed. 

$Q(X) = \frac{\mathsf{Poly}_\mathsf{Vanish1}(X) + \mathsf{Poly}_\mathsf{Vanish2}(X) \rho}{X^n - 1}$

By rearranging, we can get $\mathsf{Poly}_\mathsf{Zero}(X)$ as a true zero polynomial (zero at every value both in $\mathcal{H}_\kappa$ and outside of it):

$\mathsf{Poly}_\mathsf{Zero}(X)=\mathsf{Poly}_\mathsf{Vanish1}(X) + \rho \mathsf{Poly}_\mathsf{Vanish2}(X) - Q(X)\cdot (X^n - 1)=0$

Ultimately the shuffle1 argument will satisfy the following constraints at the Commitment Level:

1. Show $Q(X)$ exists (as a polynomial that evenly divides the divisor)
2. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is correctly constructed from $\mathsf{Poly}_\mathsf{Arr_1}(X)$, $\mathsf{Poly}_\mathsf{Arr_2}(X)$, $\mathsf{Poly_\pi(X)}$, $\mathsf{Poly}_\mathsf{Arr_1'}(X)$, $\mathsf{Poly}_\mathsf{Arr_2'}(X)$
3. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is the zero polynomial

In addition, it will show that $\prod^{n-1}_{i = 1}(X - Y\cdot i - \mathsf{Arr_1}[i]) = \prod^{n-1}_{i = 1}(X - Y\cdot \mathsf{Arr_\pi}[i] - \mathsf{Arr_2}[i])$ using a [mult3](./mult3) product check.

### Commitment Level

The verifier will never see the arrays or polynomials themselves. They are undisclosed because they either (i) contain private data or (ii) they are too large to examine and maintain a succinct proof system. Instead the prover will use commitments.

The prover will create a transcript for the product check, as decribed in [mult3](./mult3). Below, we give details specific to the second component, verifing the two constraints.

The prover will write the following commitments to the transcript:

* $K_\mathsf{Arr_1}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_1}(X))$
* $K_\mathsf{Arr_2}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_2}(X))$​
* $K_\pi = \mathsf{KZG.Commit(Poly_\pi}(X))$

The prover will generate two random challenge evaluation points (using strong Fiat-Shamir) on the polynomials that is outside of $\mathcal{H}_\kappa$. Call these points $r$ and $s$. It will use these points to define the arrays $\mathsf{Arr_1'} = \{ r - s\cdot i - \mathsf{Arr_1}[i]\}_{i \in [0, n-1]}$ and $\mathsf{Arr_2'} = \{ r - s\cdot\mathsf{Arr_\pi}[i] - \mathsf{Arr_2}[i]\}_{i \in [0, n-1]}$  and run the product check. It will write the product check into the transcript, but here we focus only on what is relevant to the second component, the points $r$ and $s$, and the following polynomials:

* $r$
* $K_\mathsf{Arr_1'}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_1'}(X))$
* $K_\mathsf{Arr_2'}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_2'}(X))$

The prover will generate a random challenge evaluation point (using strong Fiat-Shamir) on the polynomial that is outside of $\mathcal{H}_\kappa$. Call this point $\rho$. It will be used by the prover to create polynomial $Q(X)$ (see above) and the prover will write to the transcript: 

* $\rho$
* $K_Q=\mathsf{KZG.Commit}(Q(X))$

The prover will generate a random challenge evaluation point (using strong Fiat-Shamir) on the polynomials that is outside of $\mathcal{H}_\kappa$. Call this point $\zeta$. The prover will write the point and opening proofs to the transcript:

* $\zeta$
* $\mathsf{Poly}_\mathsf{Arr_1}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr_1},\zeta)$
* $\mathsf{Poly}_\mathsf{Arr_2}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr_2},\zeta)$​
* $\mathsf{Poly_\pi}(\zeta) = \mathsf{KZG.Open}(K_\pi, \zeta)$
* $\mathsf{Poly}_\mathsf{Arr_1'}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr_1'},\zeta)$
* $\mathsf{Poly}_\mathsf{Arr_2'}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr_2'},\zeta)$
* $Q(\zeta)=\mathsf{KZG.Open}(K_Q,\zeta)$

To check the proof, the verifier uses the transcript to construct the value $Y_\mathsf{Zero}$ as follows:

* $Y_\mathsf{Vanish1}= \mathsf{Poly}_\mathsf{Arr_1'}(\zeta) - (r - s\cdot \zeta - \mathsf{Poly}_\mathsf{Arr_1}(\zeta))$
* $Y_\mathsf{Vanish2}= \mathsf{Poly}_\mathsf{Arr_2'}(\zeta) - (r - s\cdot \mathsf{Poly_\pi}(\zeta) - \mathsf{Poly}_\mathsf{Arr_2}(\zeta))$
* $Y_\mathsf{Zero}=Y_\mathsf{Vanish1} + \rho Y_\mathsf{Vanish2} - Q(\zeta)\cdot (\zeta^n - 1)$

Finally, if the constraint system is true, the following constraint will be true (and will be false otherwise with overwhelming probability, due to the Schwartz-Zippel lemma on $\zeta$) :

* $Y_\mathsf{Zero}\overset{?}{=}0$

## Implementations



## Security Proof

### Completeness

Any honest prover can do the computations explained above and create an accepting proof.

### Soundness

We prove knowledge soundness in the Algebraic Group Model (AGM). We assume soundness of the product check (it is proven in [mult3](./mult3)) and conduct a proof of soundness for the rest of the protocol. To do so, we must prove that there exists an efficient extractor $\mathcal{E}$ such that for any algebraic adversary $\mathcal{A}$ the probability of $\mathcal{A}$ winning the following game is $\mathsf{negl}(\lambda)$.

1. Given $[g, g^\tau, g^{\tau^2}, \dots,g^{\tau^{n-1}}]$ $\mathcal{A}$ outputs commitments to $\mathsf{Poly}_\mathsf{Arr_1}(X)$, $\mathsf{Poly}_\mathsf{Arr_2}(X)$, $\mathsf{Poly_\pi}$, $\mathsf{Poly}_\mathsf{Arr_1'}(X)$, $\mathsf{Poly}_\mathsf{Arr_2'}(X)$, $Q(X)$

2. $\mathcal{E}$, given access to $\mathcal{A}$'s outputs from the previous step, outputs $\mathsf{Poly}_\mathsf{Arr_1}(X)$, $\mathsf{Poly}_\mathsf{Arr_2}(X)$, $\mathsf{Poly_\pi}$, $\mathsf{Poly}_\mathsf{Arr_1'}(X)$, $\mathsf{Poly}_\mathsf{Arr_2'}(X)$, $Q(X)$

3. $\mathcal{A}$ plays the part of the prover in showing that $Y_{\mathsf{Zero}}$ is zero at a random challenge $\zeta$

4. $\mathcal{A}$ wins if: 

   i) $\mathcal{V}$ accepts at the end of the protocol

   ii) $\mathsf{Arr}_2 \neq \mathsf{Permute}(\mathsf{Arr}_1 ,\pi)$

Our proof is as follows:

For the second win condition to be fulfilled, it must be that $(\mathsf{Arr_\pi}[i], \mathsf{Arr_2}[i]) \neq (i, \mathsf{Arr_1}[i])$ for some $i \in [0, n-1]$. But then, $\mathsf{Arr_1'}$ and $\mathsf{Arr_2'}$ contain differing element. This means that $\mathsf{Poly}_\mathsf{Arr_1'}(X) = \prod^{n-1}_{i = 1}(X - Y\cdot i - \mathsf{Arr_1}[i])$  and $\mathsf{Poly}_\mathsf{Arr_2'}(X) = \prod^{n-1}_{i = 1}(X - Y\cdot \mathsf{Arr_\pi}[i] - \mathsf{Arr_2}[i])$ and different polynomials, and thus by the Schwartz-Zippel lemma, there is negligible probability that they are equal at $X=r$ and $Y=2$ for the random challenge $r, s$. Any strategy to increase this probability to greater than negligible means $\mathcal{A}$ must pass in $\mathsf{Arr_j'}$ for $j = 1$ or $j = 2$ that is not defined according to the its corresponding constraint. But then $\mathsf{Poly}_\mathsf{Vanishj}(X)$ is not vanishing on $\mathcal{H}_\kappa$, so $Q(X)$ is not a polynomial (it is a rational function). This means that $\mathcal{A}$ cannot calcuated the correct commitment value $g^{Q(\tau)}$ without solving the t-SDH. Thus, $\mathcal{A}$ chooses an arbitrary value for $Q(\tau)$ and sends $K_Q = g^{Q(\tau)}$. Before this, it also sends commitments to $\mathsf{Poly}_\mathsf{Arr_1}(X)$, $\mathsf{Poly}_\mathsf{Arr_2}(X)$, $\mathsf{Poly_\pi}$, $\mathsf{Poly}_\mathsf{Arr_1'}(X)$, and $\mathsf{Poly}_\mathsf{Arr_2'}(X)$. All commitments $\mathcal{A}$ has outputted is are linear combinations of the elements in $[g, g^\tau, g^{\tau^2}, \dots,g^{\tau^{n-1}}]$. $\mathcal{E}$ is given these coefficients (since $\mathcal{A}$ is an algebraic adversary) so $\mathcal{E}$ can output the original polynomials.

$\mathcal{A}$ then obtains the random challenge $\zeta$ (using strong Fiat-Shamir). By the binding property of KZG commitments, $\mathsf{Poly}_\mathsf{Arr_1}(\zeta)$, $\mathsf{Poly}_\mathsf{Arr_2}(\zeta)$, $\mathsf{Poly_\pi(\zeta)}$, $\mathsf{Poly}_\mathsf{Arr_1'}(\zeta)$, and $\mathsf{Poly}_\mathsf{Arr_2'}(\zeta)$  can only feasibliy be opened to one value. For $\mathcal{A}$ to have the verifier accept, it must send a proof that $Q(\zeta)$ opens to $Q(\zeta) = \frac{Y_\mathsf{Vanish1} + \rho Y_\mathsf{Vanish2}}{(\zeta^n - 1)}$. This means being able to send $g^{q(\tau)}$ where $q(\tau) = \frac{Q(\tau) - Q(\zeta)}{\tau - \zeta}$ and $Q(\zeta) = \frac{Y_\mathsf{Vanish1} + \rho Y_\mathsf{Vanish2}}{(\zeta^n - 1)}$. Since $Q(\tau)$ and $Q(\zeta)$ are known, this implies knowing $g^{\frac{1}{\tau - \zeta}}$. Thus $\mathcal{A}$ would have found $\langle\zeta,g^{\frac{1}{\tau - \zeta}}\rangle$, which is the t-SDH problem. We have shown that creating an accepting proof reduces to the t-SDH, so $\mathcal{A}$'s probability of success is negligible.

### Zero-Knowledge

We prove that the above protocol is zero-knowledge when $\mathsf{PolyCommit}_\mathsf{Ped}$ (from the KZG paper) is used for the polynomial commitments. We assume the product check is zero-knowledge (it is proven in [mult3](./mult3)), and conduct a proof for the rest of the protocol. We do so by constructing a probabilistic polynomial time simulator $\mathcal{S}$ that knows the trapdoor $\tau$, which, for every (possibly malicious) verifier $\mathcal{V}$, can output a view of the execution of the protocol that is indistinguishable from the view produced by the real execution of the program.

The simulator $\mathcal{S}$ choose arbitrary values for ${\mathsf{Poly}_\mathsf{Arr_1}(\tau)}$, ${\mathsf{Poly}_\mathsf{Arr_2}(\tau)}$, and $\mathsf{Poly_\pi}(\tau)$, then computes $g^{\mathsf{Poly}_\mathsf{Arr_1}(\tau)}$,  $g^{\mathsf{Poly}_\mathsf{Arr_2}(\tau)}$, $g^{\mathsf{Poly_\pi}(\tau)}$ to output as the commitments $ K_\mathsf{Arr_1}$, $K_\mathsf{Arr_1}$, $K_\pi$. $\mathcal{S}$ then generates $r$ and $s$ as values for the ranom challenge (by strong Fiat-Shamir). It then chooses arbitrary values for ${\mathsf{Poly}_\mathsf{Arr_1'}(\tau)}$ and ${\mathsf{Poly}_\mathsf{Arr_2'}(\tau)}$, then computes $g^{\mathsf{Poly}_\mathsf{Arr_1'}(\tau)}$ and $g^{\mathsf{Poly}_\mathsf{Arr_2'}(\tau)}$ to output as the commitments $ K_\mathsf{Arr_1'}$ and $K_\mathsf{Arr_1'}$. It creates a view of the product check as described in the zero-knowledge proof for [mult3](./mutl3).

$\mathcal{S}$ generates the challenge evaluation point $\rho$ (by strong Fiat-Shamir) and computes $Q(\tau)$ using $\rho$ and the values it chose for ${\mathsf{Poly}_\mathsf{Arr_1}(\tau)}$, ${\mathsf{Poly}_\mathsf{Arr_2}(\tau)}$, $\mathsf{Poly_\pi}(\tau)$, ${\mathsf{Poly}_\mathsf{Arr_1'}(\tau)}$, and ${\mathsf{Poly}_\mathsf{Arr_2'}(\tau)}$. $\mathcal{S}$ outputs the commitment $K_Q = g^{Q(\tau)}$.

Now, $\mathcal{S}$ generates the second random challenge, $\zeta$ (which we assume is not in $\mathcal{H}_\kappa$; if it is in $\mathcal{H}_\kappa$, $\mathcal{S}$ simply restarts and runs from the beginning). This is once again by strong Fiat-Shamir. $\mathcal{S}$ then create fake opening proofs for ${\mathsf{Poly}_\mathsf{Arr_1}(\zeta)}$, ${\mathsf{Poly}_\mathsf{Arr_2}(\zeta)}$, $\mathsf{Poly_\pi}(\zeta)$, ${\mathsf{Poly}_\mathsf{Arr_1'}(\zeta)}$, and ${\mathsf{Poly}_\mathsf{Arr_2'}(\zeta)}$, to arbitrary values. This is done using the knowledge of $\tau$, calculating the respective witness $q(\tau) = \frac{{f(\tau) - f(\zeta)}}{\tau - \zeta}$ for each of the polynomials.

Finally, $\mathcal{S}$ creates a fake opening proof for $Q(\zeta) = \frac{Y_\mathsf{Vanish1} + \rho Y_\mathsf{Vanish2}}{(\zeta^n - 1)}$. This is done using knowledge of $\tau$ to calculate an accepting witness $q(\tau)$, as above. This means that $Y_\mathsf{Zero}$ will be zero, and the transcript will be accepted by the verifier. It is indistinguishable from a transcript generates from a real execution, since $\mathsf{PolyCommit}_\mathsf{Ped}$ has the property of Indistinguishability of Commitments due to the randomization by $h^{\hat{\phi}(x)}$. 