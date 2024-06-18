# Lookup (Type 1)

## Recap of types

| Type                 | Description                         | Recap                                                        | This |
| -------------------- | ----------------------------------- | :----------------------------------------------------------- | ---- |
| [lookup1](./lookup1) | $\mathsf{Arr}[i]\in \{0,1\}$        | Each element of array $\mathsf{Arr}$ is in $\{0,1\}$ (or another small set). | ✅    |
| [lookup2](./lookup2) | $\mathsf{Arr}[i]\in \mathsf{Table}$ | Each element of array $\mathsf{Arr}$ is in a disclosed table of values $\mathsf{Table}$. |      |

## Relation

$ \mathcal{R}_{\mathtt{lookup1}} := \left\{ \begin{array}{l} (K_\mathsf{Arr}) \end{array} \middle | \begin{array}{l} \mathsf{Arr} = [a_0, a_1, a_2, \dots, a_{n-1}], \\ \mathsf{Poly}_\mathsf{Arr}=\mathsf{FFT.Interp}(\omega,\mathsf{Arr}), \\ K_\mathsf{Arr}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr}), \end{array} \right\} $

## Intuition

The prover ($\mathcal{P}$) holds an array $\mathsf{Arr}$ of $n$ integers from $\mathbb{Z}_q$: $[a_0, a_1, a_2, \dots, a_{n-1}]$. It will produce a succinct (independent of $n$) proof that each element in $\mathsf{Arr}$ is in $\{0,1\}$ (or another small set). The prover will encode $\mathsf{Arr}$ into the polynomial $\mathsf{Poly}_\mathsf{Arr}$ (using [evaluation points]() on the domain $\mathcal{H}_\kappa$). It will commit to the polynomial: $K_\mathsf{Arr}$. The verifier ($\mathcal{V}$) cannot check any of the $\mathsf{Arr}$ values directly (they may contain secret information, and even if they do not, they are too long to check) so the verifier only sees $K_\mathsf{Arr}$.

In order to check that each element of $\mathsf{Arr}$ is either 0 or 1, consider the expression $\mathsf{Arr}[i] \cdot (\mathsf{Arr}[i] - 1)$. If $\mathsf{Arr}[i]$ is 0, then the first part of the product is zero, and if $\mathsf{Arr}[i]$ is 1, then the second part of the product is zero. If $\mathsf{Arr}[i]$ is not 0 or 1, then the expression will be non-zero. Thus, to check our condition, it suffices to check that $\mathsf{Arr}[i] \cdot (\mathsf{Arr}[i] - 1) = 0$ for $i$ from 0 to $n-1$.

## Protocol Details

### Array Level

* $\mathcal{P}$ holds an array $\mathsf{Arr} = [a_0, a_1, a_2, \dots, a_{n-1}]$ of $n$ integers ($a_i \in \mathbb{Z}_q$)

### Polynomial Level

We assume that $\mathsf{Arr}$ is encoded as the y-coordinates into a univariant polynomial where the x-coordinates (called the domain $\mathcal{H}_\kappa$) are chosen as the multiplicative group of order $\kappa$ with generator $\omega\in\mathbb{G}_\kappa$ (see [Background](../background/poly-iop.md) for more). In short, $\omega^0$ is the first element and $\omega^{\kappa-1}$ is the last element of $\mathcal{H}_\kappa$. If $\kappa$ is larger than the length of the array, the array can be padded with elements of value 0 or 1.

Recall the constraint we want to prove: 

1. $\mathsf{Arr}[i] \cdot (\mathsf{Arr}[i] - 1) = 0$ for $i$ from 0 to $n-1$

We write our constraint in polynomial form and label it:

1. For all $X$ from $\omega^0$ to $\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Vanish}(X)=\mathsf{Poly}_\mathsf{Arr}(X) \cdot (\mathsf{Poly}_\mathsf{Arr}(X) - 1) =0$ 

This equation is true for every value of $X \in \mathcal{H}_\kappa$ (but not necessarily true outside of these values). To show this, we divide each polynomial by  $X^\kappa - 1$, which is a minimal vanishing polynomial for $\mathcal{H}_\kappa$ that does not require interpolation to create. If the quotient is polynomial (and not a rational function), then $\mathsf{Poly}_\mathsf{Vanish}(X)$ must be vanishing on $\mathcal{H}_\kappa$ too. Specifically, the prover computes:

1. $Q(X) = \frac{\mathsf{Poly}_\mathsf{Vanish}(X)}{X^\kappa - 1}$

By rearranging, we can get $\mathsf{Poly}_\mathsf{Zero}(X)$ as a true zero polynomial (zero at every value both in $\mathcal{H}_\kappa$ and outside of it):

1. $\mathsf{Poly}_\mathsf{Zero}(X)=\mathsf{Poly}_\mathsf{Vanish}(X) - Q(X)\cdot (X^n - 1)=0$

Ultimately the lookup1 argument will satisfy the following constraints at the Commitment Level:

1. Show $Q(X)$ exists (as a polynomial that evenly divides the divisor)
2. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is correctly constructed from $\mathsf{Poly}_\mathsf{Arr}(X)$
3. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is the zero polynomial

### Commitment Level

The verifier will never see the arrays or polynomials themselves. They are undisclosed because they either (i) contain private data or (ii) they are too large to examine and maintain a succinct proof system. Instead the prover will use commitments.

The prover will write the following commitments to the transcript:

* $K_\mathsf{Arr}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr}(X))$
* $K_Q=\mathsf{KZG.Commit}(Q(X))$

The prover will generate a random challenge evaluation point (using strong Fiat-Shamir) on the polynomial that is outside of $\mathcal{H}_\kappa$. Call this point $\zeta$. The prover will write the point and opening proofs to the transcript:

* $\zeta$
* $\mathsf{Poly}_\mathsf{Arr}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr},\zeta)$
* $Q(\zeta)=\mathsf{KZG.Open}(K_Q,\zeta)$

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

We prove knowledge soundness in the Algebraic Group Model (AGM). To do so, we must prove that there exists an efficient extractor $\mathcal{E}$ such that for any algebraic adversary $\mathcal{A}$ the probability of $\mathcal{A}$ winning the following game is $\mathsf{negl}(\lambda)$.

1. Given $[g, g^\tau, g^{\tau^2}, \dots,g^{\tau^{n-1}}]$ $\mathcal{A}$ outputs commitments to $\mathsf{Poly}_\mathsf{Arr}(X)$, $Q$

2. $\mathcal{E}$, given access to $\mathcal{A}$'s outputs from the previous step, outputs $\mathsf{Poly}_\mathsf{Arr}(X)$, $Q$.

3. $\mathcal{A}$ plays the part of the prover in showing that $Y_{\mathsf{Zero}}$ is zero at a random challenge $\zeta$

4. $\mathcal{A}$ wins if: 

   i) $\mathcal{V}$ accepts at the end of the protocol

   ii) $\mathsf{Arr}[i]\neq 0$ and  $\mathsf{Arr}[i]\neq 1$ for some $i \in [0, n-1]$

Our proof is as follows:

For the second win condition to be fulfilled, there must be at least one entry is $\mathsf{Arr}$ that is not 0 or 1. But then $\mathsf{Poly}_\mathsf{Vanish}(X)$ is not vanishing on $\mathcal{H}_\kappa$, so $Q(X)$ is not a polynomial (it is a rational function). This means that $\mathcal{A}$ cannot calcuated the correct commitment value $g^{Q(\tau)}$ without solving the t-SDH. Thus, $\mathcal{A}$ chooses an arbitrary value for $Q(\tau)$ and sends $K_Q = g^{Q(\tau)}$. It also sends a commitment to  $\mathsf{Poly}_\mathsf{Arr}(X)$. Both commitments $\mathcal{A}$ has outputted is are linear combinations of the elements in $[g, g^\tau, g^{\tau^2}, \dots,g^{\tau^{n-1}}]$. $\mathcal{E}$ is given these coefficients (since $\mathcal{A}$ is an algebraic adversary) so $\mathcal{E}$ can output the original polynomials.

$\mathcal{A}$ then obtains the random challenge $\zeta$ (using strong Fiat-Shamir). By the binding property of KZG commitments, $\mathsf{Poly}_\mathsf{Arr}(\zeta)$, can only feasibliy be opened to one value. For $\mathcal{A}$ to have the verifier accept, it must send a proof that $Q(\zeta)$ opens to $Q(\zeta) = \frac{Y_\mathsf{Vanish1}}{(\zeta^n - 1)}$. This means being able to send $g^{q(\tau)}$ where $q(\tau) = \frac{Q(\tau) - Q(\zeta)}{\tau - \zeta}$ and $Q(\zeta) = \frac{Y_\mathsf{Vanish1}}{(\zeta^n - 1)}$. Since $Q(\tau)$ and $Q(\zeta)$ are known, this implies knowing $g^{\frac{1}{\tau - \zeta}}$. Thus $\mathcal{A}$ would have found $\langle\zeta,g^{\frac{1}{\tau - \zeta}}\rangle$, which is the t-SDH problem. We have shown that creating an accepting proof reduces to the t-SDH, so $\mathcal{A}$'s probability of success is negligible.

### Zero-Knowledge

We prove that the above protocol is zero-knowledge when $\mathsf{PolyCommit}_\mathsf{Ped}$ (from the KZG paper) is used for the polynomial commitments. We do so by constructing a probabilistic polynomial time simulator $\mathcal{S}$ that knows the trapdoor $\tau$, which, for every (possibly malicious) verifier $\mathcal{V}$, can output a view of the execution of the protocol that is indistinguishable from the view produced by the real execution of the program.

The simulator $\mathcal{S}$ choose an arbitrary value for ${\mathsf{Poly}_\mathsf{Arr}(\tau)}$, then computes $g^{\mathsf{Poly}_\mathsf{Arr}(\tau)}$ to output as the commitment $ K_\mathsf{Arr}$. $\mathcal{S}$ then generates the challenge evaluation point $\rho$ (by strong Fiat-Shamir) and computes $Q(\tau)$ using $\rho$ and the value it chose for ${\mathsf{Poly}_\mathsf{Arr}(\tau)}$. $\mathcal{S}$ outputs the commitment $K_Q = g^{Q(\tau)}$.

Now, $\mathcal{S}$ generates the second random challenge point $\zeta$ (which we assume is not in $\mathcal{H}_\kappa$; if it is in $\mathcal{H}_\kappa$, $\mathcal{S}$ simply restarts and runs from the beginning). This is once again by strong Fiat-Shamir.  $\mathcal{S}$ then create fake opening proofs for ${\mathsf{Poly}_\mathsf{Arr}(\zeta)}$, to an arbitrary value. This is done using the knowledge of $\tau$, calculating the witness polynomial $q(\tau) = \frac{{f(\tau) - f(\zeta)}}{\tau - \zeta}$.

Finally, $\mathcal{S}$ creates a fake opening proof for $Q(\zeta) = \frac{Y_\mathsf{Vanish1}}{(\zeta^n - 1)}$. This is done using knowledge of $\tau$ to calculate an accepting witness $q(\tau)$, as above. This means that $Y_\mathsf{Zero}$ will be zero, and the transcript will be accepted by the verifier. It is indistinguishable from a transcript generates from a real execution, since $\mathsf{PolyCommit}_\mathsf{Ped}$ has the property of Indistinguishability of Commitments due to the randomization by $h^{\hat{\phi}(x)}$. 
