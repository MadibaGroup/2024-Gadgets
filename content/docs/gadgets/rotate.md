# Rotate

## Recap of types

$ \mathcal{R}_{\mathtt{rotate}} := \left\{ \begin{array}{l} (K_\mathsf{Arr}, K_\mathsf{Arr'}) \end{array} \middle | \begin{array}{l} \mathsf{Arr} = [a_0, a_1, a_2, \dots, a_{n-1}],\\ \mathsf{Arr'}[i] = \mathsf{Arr}[i +\alpha], \\ \mathsf{Poly}_\mathsf{Arr}=\mathsf{FFT.Interp}(\omega,\mathsf{Arr}),\\ \mathsf{Poly}_\mathsf{Arr'}=\mathsf{FFT.Interp}(\omega,\mathsf{Arr'}), \\ K_\mathsf{Arr}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr}),\\ K_\mathsf{Arr'}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr'}),\\ \end{array} \right\} $

## Intuition

Assume $\mathsf{Arr}$ is an array of data of size $n$. It is encoded as the y-coordinates into univariant polynomials where the x-coordinates (called the domain $\mathcal{H}_\kappa$) are chosen as the multiplicative group of order $\kappa$ with generator $\omega\in\mathbb{G}_\kappa$ (see [Background](../../background/poly-iop) for more). In short, $\omega^0$ is the first element and $\omega^{\kappa-1}$ is the last element of $\mathcal{H}_\kappa$. We call our polynomial $\mathsf{Poly}_\mathsf{Arr}(X)$. The goal is to construct an output polynomial $\mathsf{Poly_\mathsf{Arr'}}$ where all the elements in $\mathsf{Arr}$ have been rotated by $\alpha$ positions; i.e. $\mathsf{Arr'}[i] \leftarrow \mathsf{Arr}[i + \alpha]$. Here, the index is defined mod $n$.

Written in terms the polynomials, the goal is that $\mathsf{Poly_\mathsf{Arr'}}(\omega^i) = \mathsf{Poly_\mathsf{Arr}}(\omega^i\cdot\omega^\alpha)$ for all $i \in [0, n-1]$. To this end, we define $\mathsf{Poly_\mathsf{Arr}}(X) = \mathsf{Poly_\mathsf{Arr}}(X)\cdot\omega^\alpha$ and demonstrate below why this satisfies the requirements of the output polynomials.

Consider $\mathsf{Poly_\mathsf{Arr}}(X) = c_{n-1}X^{n-1} + \dots + c_2X^2 + c_1X + c_0$. Then: 

$\mathsf{Poly_\mathsf{Arr'}}(X)$ 

$= \mathsf{Poly_Arr}(X) \cdot \omega^\alpha$

$= (c_{n-1}X^{n-1} + \dots + c_2X^2 + c_1X + c_0)(\omega^\alpha) $

$= c_{n-1}X^{n-1}\cdot\omega^\alpha + \dots + c_2X^2\cdot\omega^\alpha + c_1X\cdot\omega^\alpha + c_0\cdot\omega^\alpha$

$= \mathsf{Poly_\mathsf{Arr}}(X\cdot\omega^\alpha)$

In particular, $\mathsf{Poly_{Arr'}}(\omega^i) = \mathsf{Poly_\mathsf{Arr}}(\omega^i\cdot\omega^\alpha)$ for $i \in [0, n-1]$. We also note that $\omega^n = \omega^0$, since $\omega$ is of order $\kappa$. Thus the exponent of $\omega$ is defined mod $n$, like the indexing of $\mathsf{Arr}$. This means that $\omega^i\cdot\omega^\alpha$ wraps around to have the rotation defined as it should be for $i + \alpha \gt n-1$.

To prove the relation between $\mathsf{Arr}$ and $\mathsf{Arr'}$, the prover must commit to the two polynomials, $\mathsf{Poly_{Arr}}$ and $\mathsf{Poly_{Arr'}}$ as $K_{\mathsf{Arr}}$ and $K_\mathsf{Arr'}$. The verifier ($\mathcal{V}$) cannot check either array directly (they may contain secret information, and even if they do not, it is too long to check) so the verifier only sees $K_\mathsf{Arr_1}$ and $K_\mathsf{Arr_2}$. The relation will be proven by showing $K_\mathsf{Arr_1}$ and $K_\mathsf{Arr_2}$ and consistent; this mean verifying that $\mathsf{Arr'}[i] \leftarrow \mathsf{Arr}[i + \alpha]$.

## Protocol Details

### Array Level

\* $\mathcal{P}$ holds an array $\mathsf{Arr_1} = [a_0, a_1, a_2, \dots, a_{n-1}]$ of $n$ integers ($a_{i} \in \mathbb{Z}_q$)

\* $\mathcal{P}$ holds or computes $\mathsf{Arr}'$ such that:

  \* $\mathsf{Arr}'[i]= \mathsf{Arr}[i+\alpha]$ for $i \in [0, n-1]$

### Polynomial Level

We assume that $\mathsf{Arr}$ and $\mathsf{Arr'}$ are encoded as the y-coordinates into a univariant polynomial where the x-coordinates (called the domain $\mathcal{H}_\kappa$) are chosen as the multiplicative group of order $\kappa$ with generator $\omega\in\mathbb{G}_\kappa$ (see [Background](../../background/poly-iop) for more). In short, $\omega^0$ is the first element and $\omega^{\kappa-1}$ is the last element of $\mathcal{H}_\kappa$. If $\kappa$ is larger than the length of the arrays, then $\mathsf{Arr}$ can be padded with any value (say, all 1s) as long as this is done before the rotation to create $\mathsf{Arr'}$ is computed. 

Recall the constraint we want to prove: 

1. $\mathsf{Arr}'[i]= \mathsf{Arr}[i+\alpha]$ for $i \in [0, n-1]$

We write the constraint in polynomial form:

1. For all $X$ from $\omega^0$ to $\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Arr'}(X) = \mathsf{Poly}_\mathsf{Arr}(X)\cdot\omega^\alpha$ 

We adjust the constraint to show an equality with 0 and label it:

1. $\mathsf{Poly}_\mathsf{Vanish}(X)= \mathsf{Poly}_\mathsf{Arr'}(X) - \mathsf{Poly}_\mathsf{Arr}(X)\cdot\omega^\alpha = 0$

This equation is true for every value of $X \in \mathcal{H}_\kappa$ (but not necessarily true outside of these values). To show this, we divide the polynomial by  $X^\kappa - 1$, which is a minimal vanishing polynomial for $\mathcal{H}_\kappa$ that does not require interpolation to create. If the quotient is polynomial (and not a rational function), then $\mathsf{Poly}_\mathsf{Vanish}(X)$ must be vanishing on $\mathcal{H}_\kappa$ too. Specifically, the prover computes:

1. $Q(X) = \frac{\mathsf{Poly}_\mathsf{Vanish}(X)}{X^\kappa - 1}$

By rearranging, we can get $\mathsf{Poly}_\mathsf{Zero}(X)$ as a true zero polynomial (zero at every value both in $\mathcal{H}_\kappa$ and outside of it):

$\mathsf{Poly}_\mathsf{Zero}(X)=\mathsf{Poly}_\mathsf{Vanish}(X)  - Q(X)\cdot (X^\kappa - 1)=0$

Ultimately the rotate argument will satisfy the following constraints at the Commitment Level:

1. Show $Q(X)$ exists (as a polynomial that evenly divides the divisor)
2. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is correctly constructed from $\mathsf{Poly}_\mathsf{Arr}(X)$ and $\mathsf{Poly}_\mathsf{Arr'}(X)$
3. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is the zero polynomial

### Commitment Level

The verifier will never see the arrays or polynomials themselves. They are undisclosed because they either (i) contain private data or (ii) they are too large to examine and maintain a succinct proof system. Instead the prover will use commitments.

The prover will write the following commitments to the transcript:

- $K_\mathsf{Arr}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr}(X))$

- $K_\mathsf{Arr'}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr'}(X))$

-  $K_Q=\mathsf{KZG.Commit}(Q(X))$

The prover will generate a random challenge evaluation point (using strong Fiat-Shamir) on the polynomials that is outside of $\mathcal{H}_\kappa$. Call this point $\zeta$. The prover will write the point and opening proofs to the transcript:

-  $\zeta$

-  $\mathsf{Poly}_\mathsf{Arr}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr},\zeta)$

- $\mathsf{Poly}_\mathsf{Arr'}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr'},\zeta)$

- $Q(\zeta)=\mathsf{KZG.Open}(K_Q,\zeta)$

To check the proof, the verifier uses the transcript to construct the value $Y_\mathsf{Zero}$ as follows:

- $Y_\mathsf{Vanish}= \mathsf{Poly}_\mathsf{Arr'}(\zeta) - \mathsf{Poly}_\mathsf{Arr}(\zeta)\cdot\omega^\alpha$ 

- $Y_\mathsf{Zero}=Y_\mathsf{Vanish}  - Q(\zeta)\cdot (\zeta^\kappa - 1)$

Finally, if the constraint system is true, the following constraint will be true (and will be false otherwise with overwhelming probability, due to the Schwartz-Zippel lemma on $\zeta$) :

- $Y_\mathsf{Zero}\overset{?}{=}0$

## Security Proof

### Completeness

If $Y_\mathsf{Zero}$ is zero, then $\mathcal{V}$ will accept. Therefore, to show completeness, we show that any prover who holds $\mathsf{Arr'}$ and $\mathsf{Arr}$ such that $\mathsf{Arr'}[i] = \mathsf{Arr}[i +\alpha] \space \forall i \in [0, n-1]$ can follow the steps outlined in the above protocol and the resulting $Y_\mathsf{Zero}$ will be equal to zero.  To see this, observed that $Y_\mathsf{Zero}$

$ = Y_\mathsf{Vanish} - Q(\zeta)(\zeta^\kappa - 1)$ 

$ = \mathsf{Poly_{Arr'}}(\zeta) - \mathsf{Poly_{Arr}}(\zeta)\cdot \omega^\alpha - Q(\zeta)(\zeta^\kappa - 1)$

 $= \mathsf{Poly_{Arr'}}(\zeta) - \mathsf{Poly_{Arr}}(\zeta)\cdot \omega^\alpha - \frac{\mathsf{Poly_{Vanish}}(\zeta)}{\zeta^\kappa - 1}\cdot(\zeta^\kappa - 1)$ 

$= \mathsf{Poly_{Arr'}}(\zeta) - \mathsf{Poly_{Arr}}(\zeta)\cdot \omega^\alpha - \mathsf{Poly_{Arr'}}(\zeta) + \mathsf{Poly_{Arr}}(\zeta)\cdot \omega^\alpha$

$= 0$

Where the third equality relies on the fact that $\mathsf{Poly_{Vanish}}(X)$ is divisible by $X^\kappa - 1$. This is true if $\mathsf{Poly_{Vanish}}(X)$ is vanishing on $\mathcal{H_\kappa}$, i.e. if $\mathsf{Poly}_\mathsf{Arr'}(X) - \mathsf{Poly}_\mathsf{Arr}(X)\cdot\omega^\alpha = 0 \space \forall X \in \mathcal{H_\kappa}$.  This is true if $\mathsf{Poly}_\mathsf{Arr'}(X) - \mathsf{Poly}_\mathsf{Arr}(X\cdot\omega^\alpha) = 0 \space \forall X \in \mathcal{H_\kappa}$, which is in turn true if $\mathsf{Arr'}[i] - \mathsf{Arr}[i + \alpha] = 0 \space \forall i \in [0, \kappa -1]$, since $\mathsf{Poly_Arr}(\omega^i)=\mathsf{Arr}[i] \space \forall i \in [0, \kappa -1]$. But $\mathsf{Arr'}[i] - \mathsf{Arr}[i + \alpha] = 0 \space \forall i \in [0, n-1]$ is precisely the relation between $\mathsf{Arr'}$ and $\mathsf{Arr}$ that we assumed held for our prover (if $\kappa \gt n$ then the arrays get padded such that this relation still holds), thus the $Y_\mathsf{Zero}(X)$ it creates by following the protocol is the zero polynomial, and its transcript will be accepted.

### Soundness

We prove knowledge soundness in the Algebraic Group Model (AGM). To do so, we must prove that there exists an efficient extractor $\mathcal{E}$ such that for any algebraic adversary $\mathcal{A}$, the probability of $\mathcal{A}$ winning the following game is $\mathsf{negl}(\lambda)$.

1. Given $[g, g^\tau, g^{\tau^2}, \dots,g^{\tau^{n-1}}]$ $\mathcal{A}$ outputs commitments to $\mathsf{Poly}_\mathsf{Arr}(X)$, $\mathsf{Poly}_\mathsf{Arr'}(X)$, $Q(X)$
2. $\mathcal{E}$, given access to $\mathcal{A}$'s outputs from the previous step, outputs $\mathsf{Poly}_\mathsf{Arr}(X)$, $\mathsf{Poly}_\mathsf{Arr'}(X)$, $Q(X)$
3. $\mathcal{A}$ plays the part of the prover in showing that $Y_{\mathsf{Zero}}$ is zero at a random challenge $\zeta$
4. $\mathcal{A}$ wins if: 

   i) $\mathcal{V}$ accepts at the end of the protocol

   ii) $\mathsf{Arr'}[i] \neq \mathsf{Arr}[i + \alpha]$ for some $i \in [0, n-1]$

Our proof is as follows:

For the second win condition to be fulfilled, there must be some $i \in [0, n-1]$ such that $\mathsf{Arr'}[i] \neq \mathsf{Arr}[i + \alpha]$. But then $\mathsf{Poly}_\mathsf{Vanish}(X)$ is not vanishing on $\mathcal{H}_\kappa$, so $Q(X)$ is not a polynomial (it is a rational function). This means that $\mathcal{A}$ cannot calculated the correct commitment value $g^{Q(\tau)}$ without solving the t-SDH. Thus, $\mathcal{A}$ chooses an arbitrary value for $Q(\tau)$ and writes $K_Q = g^{Q(\tau)}$ to the transcript. Before this, it also writes commitments to $\mathsf{Poly}_\mathsf{Arr}(X)$ and $\mathsf{Poly}_\mathsf{Arr'}(X)$. All commitments $\mathcal{A}$ has written are linear combinations of the elements in $[g, g^\tau, g^{\tau^2}, \dots,g^{\tau^{n-1}}]$. $\mathcal{E}$ is given these coefficients (since $\mathcal{A}$ is an algebraic adversary) so $\mathcal{E}$ can output the original polynomials.

$\mathcal{A}$ then obtains the random challenge $\zeta$ (using strong Fiat-Shamir). By the binding property of KZG commitments, $\mathsf{Poly}_\mathsf{Arr}(\zeta)$ and $\mathsf{Poly}_\mathsf{Arr'}(\zeta)$ can each only feasibly be opened to one value. For $\mathcal{A}$ to have the verifier accept, it must produce a proof that $Q(\zeta)$ opens to $Q(\zeta) = \frac{Y_\mathsf{Vanish}} {(\zeta^\kappa - 1)}$. This means being able to produce $g^{q(\tau)}$ where $q(\tau) = \frac{Q(\tau) - Q(\zeta)}{\tau - \zeta}$ and $Q(\zeta) = \frac{Y_\mathsf{Vanish}}{(\zeta^\kappa - 1)}$. Since $Q(\tau)$ and $Q(\zeta)$ are known, this implies knowing $g^{\frac{1}{\tau - \zeta}}$. Thus $\mathcal{A}$ would have found $\langle\zeta,g^{\frac{1}{\tau - \zeta}}\rangle$, which is the t-SDH problem. We have shown that creating an accepting proof reduces to the t-SDH, so $\mathcal{A}$'s probability of success is negligible.

### Zero-Knowledge

We prove that the above protocol is zero-knowledge when $\mathsf{PolyCommit}_\mathsf{Ped}$ (from the KZG paper) is used for the polynomial commitments. We do so by constructing a probabilistic polynomial time simulator $\mathcal{S}$ that knows the trapdoor $\tau$, which, for every (possibly malicious) verifier $\mathcal{V}$, can output a view of the execution of the protocol that is indistinguishable from the view produced by the real execution of the program.

The simulator $\mathcal{S}$ chooses arbitrary values for ${\mathsf{Poly}_\mathsf{Arr}(\tau)}$ and ${\mathsf{Poly}_\mathsf{Arr'}(\tau)}$, then computes $g^{\mathsf{Poly}_\mathsf{Arr}(\tau)}$ and $g^{\mathsf{Poly}_\mathsf{Arr'}(\tau)}$ to write as the commitments $ K_\mathsf{Arr}$ and $K_\mathsf{Arr'}$. $\mathcal{S}$ computes $Q(\tau)$ using the values it chose for ${\mathsf{Poly}_\mathsf{Arr}(\tau)}$ and ${\mathsf{Poly}_\mathsf{Arr'}(\tau)}$. $\mathcal{S}$ writes the commitment $K_Q = g^{Q(\tau)}$.

Now, $\mathcal{S}$ generates the random challenge point $\zeta$ (which we assume is not in $\mathcal{H}_\kappa$; if it is in $\mathcal{H}_\kappa$, $\mathcal{S}$ simply restarts and runs from the beginning). This is by strong Fiat-Shamir. $\mathcal{S}$ then create fake opening proofs for ${\mathsf{Poly}_\mathsf{Arr}(\zeta)}$ and ${\mathsf{Poly}_\mathsf{Arr'}(\zeta)}$, to arbitrary values. This is done using the knowledge of $\tau$, calculating the respective witness $q(\tau) = \frac{{f(\tau) - f(\zeta)}}{\tau - \zeta}$ for each of the polynomials.

Finally, $\mathcal{S}$ creates a fake opening proof for $Q(\zeta) = \frac{Y_\mathsf{Vanish}}{(\zeta^\kappa - 1)}$. This is done using knowledge of $\tau$ to calculate an accepting witness $q(\tau)$, as above. This means that $Y_\mathsf{Zero}$ will be zero, and the transcript will be accepted by the verifier. It is indistinguishable from a transcript generates from a real execution, since $\mathsf{PolyCommit}_\mathsf{Ped}$ has the property of Indistinguishability of Commitments due to the randomization by $h^{\hat{\phi}(x)}$. 
