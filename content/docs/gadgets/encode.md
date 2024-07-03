# Encode

## Relation

$ \mathcal{R}_{\mathtt{mult3}} := \left\{ \begin{array}{l} (K_\mathsf{Arr_1},K_\mathsf{Arr2}, \end{array} \middle | \begin{array}{l} \mathsf{Arr_1} = [a_{(1,0)}, a_{(1,1)}, a_{(1,2)}, \dots, a_{(1,n-1)}],\\ \mathsf{Arr_2} = [a_{(2,0)}, a_{(2,1)}, a_{(2,2)}, \dots, a_{(2,n-1)}], \\  \mathsf{Poly}_\mathsf{Arr_1}=\mathsf{FFT.Interp}(\omega,\mathsf{Arr_1}), \\ \mathsf{Poly}_\mathsf{Arr_2}=\mathsf{FFT.Interp}(\omega,\mathsf{Arr_2}), \\ K_\mathsf{Arr_1}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_1}),\\ K_\mathsf{Arr_2}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_2}), \end{array} \right\} $

## Intuition

The prover ($\mathcal{P}$) holds two arrays $\mathsf{Arr_1}$ and $\mathsf{Arr_2}$ of $n$ integers from $\mathbb{Z}_q$: $[a_0, a_1, a_2, \dots, a_{n-1}]$. It wishes to map the pairs of elements, $\{ \mathsf{Arr_1[i], \mathsf{Arr_2[i]}}\}$ into a single element, $\mathsf{Arr_3}[i]$ without collisions. It will produce a succinct (independent of $n$) proof that $\mathsf{Arr_3}$ contains the encoding of $\mathsf{Arr_1}$ and $\mathsf{Arr_2}$ according to this mapping. 

For the mapping scheme, we use random linear combinations, since it is both collision resistant and it can be proven in zero knowledge. To do so, the prover defines $f_i(X) = \mathsf{Arr_1}[i] + \mathsf{Arr_2}[i] \cdot X$ for $i \in [0, n-1]$. Then, the the prover calculates $\mathsf{Arr_3}[i] = f_i(r)$ for a random field element $r$. Assuming $f_i$ is commited to before $r$ is know, this scheme is collision resistant by the Schwartz-Zippel lemma; if two polynomials are equal at $r$ then with overwhelming probability they are the same polynomial. However, to ensure negligible probabilty of collisions, $n$ may need to be smaller relative to the field size than initially thought. This is due to the birthday paradox. We are not evaluating the probability that a *single one* of the $n$ polynomials collides with one of the other $n-1$ polynomials, but rather the probability than *any one* polynomial out of $n$ collides with any other of the $n-1$ polynomials. The value of the latter is significantly larger, since it compares every possible pair of the $n$ polynomials, instead of just comparing one of the polynomials with the $n-1$ other polynomials. The probability of collisions can, however, still be made sufficiently small by bounding how large $n$ can be relative to the field size.

In order to prove the relation, the prover will encode the three arrays into three polynomials: $\mathsf{Poly}_\mathsf{Arr_1}$, $\mathsf{Poly}_\mathsf{Arr_2}$, and $\mathsf{Poly}_\mathsf{Arr_3}$ (using [evaluation points](../../background/poly-iop) on the domain $\mathcal{H}_\kappa$). It will commit to each polynomial: $K_\mathsf{Arr_1}$, $K_\mathsf{Arr_2}$, and $K_\mathsf{Arr_3}$. The verifier ($\mathcal{V}$) cannot check any of the $\mathsf{Arr_i}$ or $\mathsf{Poly}_\mathsf{Arr_i}$ values directly (they may contain secret information, and even if they do not, they are too long to check) so the verifier only sees $K_\mathsf{Arr_1}$,$K_\mathsf{Arr_2}$, and $K_\mathsf{Arr_3}$. It will use these commitments to verified the constraint $\mathsf{Arr_3}[i] = \mathsf{Arr_1}[i] + \mathsf{Arr_2}[i]\cdot r$.

## Protocol Details

### Array Level

* $\mathcal{P}$ holds an array $\mathsf{Arr_1} = [a_{(1,0)}, a_{(1,1)}, a_{(1,2)}, \dots, a_{(1,n-1)}]$
* $\mathcal{P}$ holds an array $\mathsf{Arr_2} = [a_{(2,0)}, a_{(2,1)}, a_{(2,2)}, \dots, a_{(2, n-1)}]$
* $\mathcal{P}$ generates the random challenge $r$, then computes $\mathsf{Arr_3}$ as follows:
  * $\mathsf{Arr_3}[i] = \mathsf{Arr_1}[i] + \mathsf{Arr_2}[i]\cdot r$

### Polynomial Level

We assume that $\mathsf{Arr1}$, $\mathsf{Arr_2}$, and $\mathsf{Arr_3}$ are encoded as the y-coordinates into a univariant polynomial where the x-coordinates (called the domain $\mathcal{H}_\kappa$) are chosen as the multiplicative group of order $\kappa$ with generator $\omega\in\mathbb{G}_\kappa$ (see [Background](../../background/poly-iop) for more). In short, $\omega^0$ is the first element and $\omega^{\kappa-1}$ is the last element of $\mathcal{H}_\kappa$. If $\kappa$ is larger than the length of the arrays, the arrays can be padded with zeros.

Recall the constraint we want to prove:

1. $\mathsf{Arr_3}[i] = f_i(X) =\mathsf{Arr_1}[i] + \mathsf{Arr_2}[i]\cdot r$

We write the constraint in polynomial form:

1. For all $X$ from $\omega^0$ to $\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Arr_3}(X) = \mathsf{Poly}_\mathsf{Arr_1}(X) + \mathsf{Poly_{Arr_2}}(X)\cdot r$ 

We adjust the constraint to show an equality with 0 and label it:

1. $\mathsf{Poly}_\mathsf{Vanish}(X)= \mathsf{Poly}_\mathsf{Arr_3}(X) - (\mathsf{Poly}_\mathsf{Arr_1}(X) + \mathsf{Poly_{Arr_2}}(X)\cdot r)= 0$

This equation is true for every value of $X \in \mathcal{H}_\kappa$ (but not necessarily true outside of these values). To show this, we divide the polynomial by  $X^\kappa - 1$, which is a minimal vanishing polynomial for $\mathcal{H}_\kappa$ that does not require interpolation to create. If the quotient is polynomial (and not a rational function), then $\mathsf{Poly}_\mathsf{Vanish}(X)$ must be vanishing on $\mathcal{H}_\kappa$ too. Specifically, the prover computes:

1. $Q(X) = \frac{\mathsf{Poly}_\mathsf{Vanish}(X)}{X^\kappa - 1}$

By rearranging, we can get $\mathsf{Poly}_\mathsf{Zero}(X)$ as a true zero polynomial (zero at every value both in $\mathcal{H}_\kappa$ and outside of it):

$\mathsf{Poly}_\mathsf{Zero}(X)=\mathsf{Poly}_\mathsf{Vanish}(X)  - Q(X)\cdot (X^n - 1)=0$

Ultimately the rotate argument will satisfy the following constraints at the Commitment Level:

1. Show $Q(X)$ exists (as a polynomial that evenly divides the divisor)
2. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is correctly constructed from $\mathsf{Poly}_\mathsf{Arr_1}(X)$, $\mathsf{Poly_{Arr_2}}$, and $\mathsf{Poly}_\mathsf{Arr_3}(X)$
3. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is the zero polynomial

### Commitment Level

The verifier will never see the arrays or polynomials themselves. They are undisclosed because they either (i) contain private data or (ii) they are too large to examine and maintain a succinct proof system. Instead the prover will use commitments.

The prover will write the following commitments to the transcript:

* $K_\mathsf{Arr_1}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_1}(X))$
* $K_\mathsf{Arr_2}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_2}(X))$​

The prover will generate a random challenge evaluation point (using strong Fiat-Shamir) on the polynomials that is outside of $\mathcal{H}_\kappa$. Call this point $r$. We note that since $\mathsf{Arr_1}$ and $\mathsf{Arr_2}$ were commited to before $r$ was know, $fi$ is commited to before $r$ is known. This is important for the collision resistance of our encoding scheme. Now, using $r$, the prover can compute and commit to the following polynomials:

- $K_\mathsf{Arr_3}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_3}(X))$

* $K_Q=\mathsf{KZG.Commit}(Q(X))$

The prover will generate a random challenge evaluation point (using strong Fiat-Shamir) on the polynomials that is outside of $\mathcal{H}_\kappa$. Call this point $\zeta$. The prover will write the point and opening proofs to the transcript:

* $\zeta$
* $\mathsf{Poly}_\mathsf{Arr_1}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr_1},\zeta)$
* $\mathsf{Poly}_\mathsf{Arr_2}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr_2},\zeta)$
* $\mathsf{Poly}_\mathsf{Arr_3}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{Arr_3},\zeta)$
* $Q(\zeta)=\mathsf{KZG.Open}(K_Q,\zeta)$

To check the proof, the verifier uses the transcript to construct the value $Y_\mathsf{Zero}$ as follows:

* $Y_\mathsf{Vanish}= \mathsf{Poly}_\mathsf{Arr_3}(\zeta) - (\mathsf{Poly}_\mathsf{Arr_1}(\zeta) + \mathsf{Poly_{Arr_2}}(\zeta)\cdot r)$ 
* $Y_\mathsf{Zero}=Y_\mathsf{Vanish}  - Q(\zeta)\cdot (\zeta^n - 1)$

Finally, if the constraint system is true, the following constraint will be true (and will be false otherwise with overwhelming probability, due to the Schwartz-Zippel lemma on $\zeta$) :

* $Y_\mathsf{Zero}\overset{?}{=}0$

## Implementations



## Security Proof

### Completeness

Any honest prover can do the computations explained above and create an accepting proof.

### Soundness

We prove knowledge soundness in the Algebraic Group Model (AGM). To do so, we must prove that there exists an efficient extractor $\mathcal{E}$ such that for any algebraic adversary $\mathcal{A}$, the probability of $\mathcal{A}$ winning the following game is $\mathsf{negl}(\lambda)$.

1. Given $[g, g^\tau, g^{\tau^2}, \dots,g^{\tau^{n-1}}]$ $\mathcal{A}$ outputs commitments to $\mathsf{Poly}_\mathsf{Arr_1}(X)$, $\mathsf{Poly}_\mathsf{Arr_2}(X)$, $\mathsf{Poly}_\mathsf{Arr_3}(X)$ , $Q(X)$

2. $\mathcal{E}$, given access to $\mathcal{A}$'s outputs from the previous step, outputs $\mathsf{Poly}_\mathsf{Arr_1}(X)$, $\mathsf{Poly}_\mathsf{Arr_2}(X)$, $\mathsf{Poly}_\mathsf{Arr_3}(X)$, $Q(X)$

3. $\mathcal{A}$ plays the part of the prover in showing that $Y_{\mathsf{Zero}}$ is zero at a random challenge $\zeta$

4. $\mathcal{A}$ wins if: 

   i) $\mathcal{V}$ accepts at the end of the protocol

   ii) $\mathsf{Arr_3}[i] \neq \mathsf{Arr_1}[i] + \mathsf{Arr_1}[i]\cdot r$ for some $i \in [0, n-1]$

Our proof is as follows:

For the second win condition to be fulfilled, there must be some $i \in [0, n-1]$ such that $\mathsf{Arr_3}[i] \neq \mathsf{Arr_1}[i] + \mathsf{Arr_1}[i]\cdot r$. But then $\mathsf{Poly}_\mathsf{Vanish}(X)$ is not vanishing on $\mathcal{H}_\kappa$, so $Q(X)$ is not a polynomial (it is a rational function). This means that $\mathcal{A}$ cannot calcuated the correct commitment value $g^{Q(\tau)}$ without solving the t-SDH. Thus, $\mathcal{A}$ chooses an arbitrary value for $Q(\tau)$ and writes $K_Q = g^{Q(\tau)}$ to the transcript. Before this, it also writes commitments to $\mathsf{Poly}_\mathsf{Arr_1}(X)$, $\mathsf{Poly_{Arr_2}}(X)$, and $\mathsf{Poly}_\mathsf{Arr_3}(X)$. All commitments $\mathcal{A}$ has written are linear combinations of the elements in $[g, g^\tau, g^{\tau^2}, \dots,g^{\tau^{n-1}}]$. $\mathcal{E}$ is given these coefficients (since $\mathcal{A}$ is an algebraic adversary) so $\mathcal{E}$ can output the original polynomials.

$\mathcal{A}$ then obtains the random challenge $\zeta$ (using strong Fiat-Shamir). By the binding property of KZG commitments, $\mathsf{Poly}_\mathsf{Arr_1}(\zeta)$, $\mathsf{Poly_{Arr_2}}(\zeta)$, and $\mathsf{Poly}_\mathsf{Arr_3}(\zeta)$ can each only feasibliy be opened to one value. For $\mathcal{A}$ to have the verifier accept, it must produce a proof that $Q(\zeta)$ opens to $Q(\zeta) = \frac{Y_\mathsf{Vanish}} {(\zeta^n - 1)}$. This means being able to produce $g^{q(\tau)}$ where $q(\tau) = \frac{Q(\tau) - Q(\zeta)}{\tau - \zeta}$ and $Q(\zeta) = \frac{Y_\mathsf{Vanish}}{(\zeta^n - 1)}$. Since $Q(\tau)$ and $Q(\zeta)$ are known, this implies knowing $g^{\frac{1}{\tau - \zeta}}$. Thus $\mathcal{A}$ would have found $\langle\zeta,g^{\frac{1}{\tau - \zeta}}\rangle$, which is the t-SDH problem. We have shown that creating an accepting proof reduces to the t-SDH, so $\mathcal{A}$'s probability of success is negligible.

### Zero-Knowledge

We prove that the above protocol is zero-knowledge when $\mathsf{PolyCommit}_\mathsf{Ped}$ (from the KZG paper) is used for the polynomial commitments. We do so by constructing a probabilistic polynomial time simulator $\mathcal{S}$ that knows the trapdoor $\tau$, which, for every (possibly malicious) verifier $\mathcal{V}$, can output a view of the execution of the protocol that is indistinguishable from the view produced by the real execution of the program.

The simulator $\mathcal{S}$ chooses arbitrary values for ${\mathsf{Poly}_\mathsf{Arr_1}(\tau)}$, $\mathsf{Poly_{Arr_2}}(\tau)$, and ${\mathsf{Poly}_\mathsf{Arr_3}(\tau)}$, then computes $g^{\mathsf{Poly}_\mathsf{Arr_1}(\tau)}$, $g^{\mathsf{Poly}_\mathsf{Arr_2}(\tau)}$, and $g^{\mathsf{Poly}_\mathsf{Arr_3}(\tau)}$ to write as the commitments $ K_\mathsf{Arr_1}$, $ K_\mathsf{Arr_2}$ and $K_\mathsf{Arr_3}$. $\mathcal{S}$ computes $Q(\tau)$ using the values it chose for ${\mathsf{Poly}_\mathsf{Arr_1}(\tau)}$, ${\mathsf{Poly}_\mathsf{Arr_2}(\tau)}$, and ${\mathsf{Poly}_\mathsf{Arr_3}(\tau)}$. $\mathcal{S}$ writes the commitment $K_Q = g^{Q(\tau)}$.

Now, $\mathcal{S}$ generates the random challenge point $\zeta$ (which we assume is not in $\mathcal{H}_\kappa$; if it is in $\mathcal{H}_\kappa$, $\mathcal{S}$ simply restarts and runs from the beginning). This is by strong Fiat-Shamir. $\mathcal{S}$ then create fake opening proofs for ${\mathsf{Poly}_\mathsf{Arr_1}(\zeta)}$, ${\mathsf{Poly}_\mathsf{Arr_2}(\zeta)}$, and ${\mathsf{Poly}_\mathsf{Arr_3}(\zeta)}$, to arbitrary values. This is done using the knowledge of $\tau$, calculating the respective witness $q(\tau) = \frac{{f(\tau) - f(\zeta)}}{\tau - \zeta}$ for each of the polynomials.

Finally, $\mathcal{S}$ creates a fake opening proof for $Q(\zeta) = \frac{Y_\mathsf{Vanish}}{(\zeta^n - 1)}$. This is done using knowledge of $\tau$ to calculate an accepting witness $q(\tau)$, as above. This means that $Y_\mathsf{Zero}$ will be zero, and the transcript will be accepted by the verifier. It is indistinguishable from a transcript generates from a real execution, since $\mathsf{PolyCommit}_\mathsf{Ped}$ has the property of Indistinguishability of Commitments due to the randomization by $h^{\hat{\phi}(x)}$. 