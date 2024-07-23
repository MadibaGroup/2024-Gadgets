# Circuit

## Recap of types

| Type | Description | Recap | This |
| ---- | ----------- | ----- | ---- |
| circuit | $z=\mathsf{Circ}(x,y)$ | $z$ is the output of disclosed arithmetic circuit $\mathsf{Circ}$ with disclosed (and/or undisclosed) inputs $x$ and $y$. | ✅ |

## Relation

$\mathcal{R}_{\mathtt{circ}} := \left\{ \begin{array}{l} (K_\mathsf{T},K_\mathsf{In}) \end{array} \middle | \begin{array}{l} \mathsf{In}=[i_0,i_1,i_2,i_3], \\ \mathsf{T}=[t_0,t_1,t_2,t_3], \\ \mathsf{T}[3]\in\{0,1\}, \\ \mathsf{T}[3]\cdot(\mathsf{In}[0]\cdot\mathsf{T}[0]+\mathsf{In}[1]\cdot\mathsf{T}[1])+(1-\mathsf{T}[3])\cdot(\mathsf{In}[0]\cdot\mathsf{In}[1]\cdot\mathsf{T}[2])+\mathsf{In}[2]=\mathsf{In}[3], \\ \mathsf{Poly}_\mathsf{T}=\mathsf{FFT.Interp}(\omega,\mathsf{T}), \\ \mathsf{Poly}_\mathsf{In}=\mathsf{FFT.Interp}(\omega,\mathsf{In}), \\ K_\mathsf{T}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{T}), \\ K_\mathsf{In}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{In}) \\ \end{array} \right\}$

## Intuition

The prover ($\mathcal{P}$) and the verifier ($\mathcal{V}$) are both given a circuit $\mathsf{T}$, where $\mathsf{T}[0]$ and $\mathsf{T}[1]$ are the coefficients of the two inputs respectively, $\mathsf{T}[2]$ is the coefficient of the product of the two inputs, and $\mathsf{T}[3]$ is the selector of the gate ($\mathsf{T}[3]$ is one for addition gate and zero for multiplication gate). The prover wants to prove he knows an input vector $\mathsf{In}$ satisfying $\mathsf{T}$. Specifically, we define $\mathsf{In}[0]$ and $\mathsf{In}[1]$ are the inputs of the circuit, $\mathsf{In}[2]$ is a constant, and $\mathsf{In}[3]$ is the output. Thus, the prover will produce a succinct proof that $\mathsf{In}$ satisfies the following condition: the equation $\mathsf{T}[3]\cdot(\mathsf{In}[0]\cdot\mathsf{T}[0]+\mathsf{In}[1]\cdot\mathsf{T}[1])+(1-\mathsf{T}[3])\cdot(\mathsf{In}[0]\cdot\mathsf{In}[1]\cdot\mathsf{T}[2])+\mathsf{In}[2]=\mathsf{In}[3]$ holds.

This means that if $\mathsf{T}[3] = 0$, this is the circuit which must be satisfied:

{{< mermaid >}}

flowchart LR

   in0["In[0]"] & in1["In[1]"] **-->** id1((x))

   t2["T[2]"] & id1 **-->** id2((x))

   in2["In[2]"] & id2 **-->** id3((+))

   id3 **-->** in3["In[3]"]

{{< /mermaid >}}

And if $\mathsf{T}[3] = 1$, this is the circuit which must be satisfied:

{{< mermaid >}}

flowchart LR

   in0["In[0]"] & t0["T[0]"] **-->** id1((x))

   in1["In[1]"] & t1["T[1]"] **-->** id2((x))

   id1 & id2 **-->** id3((+))

   in2["In[2]"] & id3 **-->** id4((+))

   id4 **-->** in3["In[3]"]

{{< /mermaid >}}

Consider, as an example, the circuit $5x+6y$. Thus, $\mathsf{T}=[5,6,0,1]$. Since $\mathsf{T}$ is publicly known to both parties, $\mathsf{Poly}_\mathsf{T}$ is also known and the prover does not need to prove the correctness of $\mathsf{T}$. Now the prover claims $\mathsf{In}=[6,5,0,60]$ satisfies the circuit. Indeed, $5\cdot 6+ 6\cdot 5 + 0 = 60$. Instead of sending each element of $\mathsf{In}$ one by one to show this, the prover interpolates a polynomial $\mathsf{Poly}_\mathsf{In}$ from $\mathsf{In}$ and computes a vanishing polynomial with $\mathsf{Poly}_\mathsf{T}$ and $\mathsf{Poly}_\mathsf{In}$. If the prover can prove the polynomial is vanishing, the verifier will be convinced that the prover knows a valid $\mathsf{In}$.

## Protocol Details

### Array Level

* $\mathsf{P}$ and $\mathsf{V}$ hold the array of the circuit $\mathsf{T}=[t_0,t_1,t_2,t_3]$
* $\mathsf{P}$ holds an input array $\mathsf{In}=[i_0,i_1,i_2,i_3]$
* $\mathsf{T}$ and $\mathsf{In}$ satisfy the following conditions
    * $\mathsf{T}[3]\in\{0,1\}$
    * $\mathsf{T}[3]\cdot(\mathsf{In}[0]\cdot\mathsf{T}[0]+\mathsf{In}[1]\cdot\mathsf{T}[1])+(1-\mathsf{T}[3])\cdot(\mathsf{In}[0]\cdot\mathsf{In}[1]\cdot\mathsf{T}[2])+\mathsf{In}[2]=\mathsf{In}[3]$

### Polynomial Level

We assume arrays $\mathsf{T}$ and $\mathsf{In}$ are encoded as the y-coordinates into a univariant polynomial where the x-coordinates (called the domain $\mathcal{H}_\kappa$) are chosen as the multiplicative group of order $\kappa$ with generator $\omega\in\mathbb{G}_\kappa$ (see [Background](../background/poly-iop.md) for more). In short, $\omega^0$ is the first element and $\omega^{\kappa-1}$ is the last element of $\mathcal{H}_\kappa$. If $\kappa$ is larger than the length of the array, the array can be padded with elements of value 1 (which will not change the product). In this case, $\kappa$ is $4$.

Recall the constraint we want to prove: 

1. $\mathsf{T}[3]\cdot(\mathsf{In}[0]\cdot\mathsf{T}[0]+\mathsf{In}[1]\cdot\mathsf{T}[1])+(1-\mathsf{T}[3])\cdot(\mathsf{In}[0]\cdot\mathsf{In}[1]\cdot\mathsf{T}[2])+\mathsf{In}[2]=\mathsf{In}[3]$

In polynomial form, the constraint is:

1. For $X=\omega^0$: $\displaylines{\mathsf{Poly}_\mathsf{T}(X\omega^3)\cdot(\mathsf{Poly}_\mathsf{In}(X)\cdot\mathsf{Poly}_\mathsf{T}(X)+\mathsf{Poly}_\mathsf{In}(X\omega)\cdot\mathsf{Poly}_\mathsf{T}(X\omega))\\+(1-\mathsf{Poly}_\mathsf{T}(X\omega^3))\cdot(\mathsf{Poly}_\mathsf{In}(X)\cdot\mathsf{Poly}_\mathsf{In}(X\omega)\cdot\mathsf{Poly}_\mathsf{T}(X\omega^2))+\mathsf{Poly}_\mathsf{In}(X\omega^2)=\mathsf{Poly}_\mathsf{In}(X\omega^3)}$

We take care of the "for $X$" condition by zeroing out the rest of the polynomial that is not zero. See the gadget <span style="border-style:dotted;border-width: 2px;"> [zero1](./zero1)</span> for more on why this works.

1. $\displaylines{\mathsf{Poly}_\mathsf{Vanish}(X)=[\mathsf{Poly}_\mathsf{T}(X\omega^3)\cdot(\mathsf{Poly}_\mathsf{In}(X)\cdot\mathsf{Poly}_\mathsf{T}(X)+\mathsf{Poly}_\mathsf{In}(X\omega)\cdot\mathsf{Poly}_\mathsf{T}(X\omega))\\+(1-\mathsf{Poly}_\mathsf{T}(X\omega^3))\cdot(\mathsf{Poly}_\mathsf{In}(X)\cdot\mathsf{Poly}_\mathsf{In}(X\omega)\cdot\mathsf{Poly}_\mathsf{T}(X\omega^2))+\\\mathsf{Poly}_\mathsf{In}(X\omega^2)-\mathsf{Poly}_\mathsf{In}(X\omega^3)]\cdot\frac{X^\kappa-1}{X-\omega^0}}$

The equation is vanishing for every value of $X\in\mathcal{H}_\kappa$ (but not necessarily true outside of these values). To show this, we divide the polynomial by $X^\kappa-1$, which is a minimal vanishing polynomial for $\mathcal{H}_\kappa$ that does not require interpolation to create. If the quotient is polynomial (and not a rational function), then $\mathsf{Poly}_\mathsf{Vanish}(X)$ must be vanishing on $\mathcal{H}_\kappa$ too. Specifically, the prover computes:

$$
Q(X) = \frac{\mathsf{Poly}_\mathsf{Vanish}(X)}{X^\kappa-1}
$$

By rearranging, we can get $\mathsf{Poly}_\mathsf{Zero}(X)$ as a true zero polynomial (zero at every value both in $\mathcal{H}_\kappa$ and outside of it):

$\mathsf{Poly}_\mathsf{Zero}(X)=\mathsf{Poly}_\mathsf{Vanish}(X)-Q(X)\cdot(X^{\kappa}-1)=0$

Ultimately the range gadget will satisfy the following constraints at the Commitment Level:
1. Show $Q(X)$ exists (as a polynomial that evenly divides the divisor)
2. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is correctly constructed from $\mathsf{Poly}_\mathsf{T}(X)$ and $\mathsf{Poly}_\mathsf{In}(X)$
3. Show $\mathsf{Poly}_\mathsf{Zero}(X)$ is the zero polynomial

### Commitment Level

The verifier will never see the vector $\mathsf{In}$ or the polynomial $\mathsf{Poly}_\mathsf{In}$. It is undisclosed because it either (i) contains private data or (ii) is too large to examine and maintain a succinct proof system. Instead, the prover will use commitments.

The prover will write the following commitments to the transcript:
* $K_\mathsf{T}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{T}(X))$
* $K_\mathsf{In}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{In}(X))$

The prover will generate a random challenge evaluation point (using strong Fiat-Shamir) on the polynomial that is outside of $\mathcal{H}_\kappa$. Call this point $\rho$. It will be used by the prover to create polynomial $Q(X)$ (see above) and the prover will write to the transcript:

* $\rho$
* $K_Q=\mathsf{KZG.Commit}(Q(X))$

The prover will generate a second random challenge evaluation point (using strong Fiat-Shamir) on the polynomial that is outside of $\mathcal{H}_\kappa$. Call this point $\zeta$. The prover will write the point and opening proofs to the transcript:

* $\zeta$
* $\mathsf{Poly}_\mathsf{T}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{T},\zeta)$
* $\mathsf{Poly}_\mathsf{T}(\zeta\omega)=\mathsf{KZG.Open}(K_\mathsf{T},\zeta\omega)$
* $\mathsf{Poly}_\mathsf{T}(\zeta\omega^2)=\mathsf{KZG.Open}(K_\mathsf{T},\zeta\omega^2)$
* $\mathsf{Poly}_\mathsf{T}(\zeta\omega^3)=\mathsf{KZG.Open}(K_\mathsf{T},\zeta\omega^3)$
* $\mathsf{Poly}_\mathsf{In}(\zeta)=\mathsf{KZG.Open}(K_\mathsf{In},\zeta)$
* $\mathsf{Poly}_\mathsf{In}(\zeta\omega)=\mathsf{KZG.Open}(K_\mathsf{In},\zeta\omega)$
* $\mathsf{Poly}_\mathsf{In}(\zeta\omega^2)=\mathsf{KZG.Open}(K_\mathsf{In},\zeta\omega^2)$
* $\mathsf{Poly}_\mathsf{In}(\zeta\omega^3)=\mathsf{KZG.Open}(K_\mathsf{In},\zeta\omega^3)$
* $Q(\zeta)=\mathsf{KZG.Open}(K_Q,\zeta)$

To check the proof, the verifier uses the transcript to construct the value $Y_\mathsf{Zero}$ as follows:

$$
\displaylines{Y_\mathsf{Vanish}=[\mathsf{Poly}_\mathsf{T}(\zeta\omega^3)\cdot(\mathsf{Poly}_\mathsf{In}(\zeta)\cdot\mathsf{Poly}_\mathsf{T}(\zeta)+\mathsf{Poly}_\mathsf{In}(\zeta\omega)\cdot\mathsf{Poly}_\mathsf{T}(\zeta\omega))\\+(1-\mathsf{Poly}_\mathsf{T}(\zeta\omega^3))\cdot(\mathsf{Poly}_\mathsf{In}(\zeta)\cdot\mathsf{Poly}_\mathsf{In}(\zeta\omega)\cdot\mathsf{Poly}_\mathsf{T}(\zeta\omega^2))+\\\mathsf{Poly}_\mathsf{In}(\zeta\omega^2)-\mathsf{Poly}_\mathsf{In}(\zeta\omega^3)]\cdot\frac{\zeta^\kappa-1}{\zeta-\omega^0}}
$$

Finally, if the constraint system is true, the following constraint will be true (and will be false otherwise with overwhelming probability, due to the Schwartz-Zippel lemma on $\rho$ and $\zeta$) :

* $Y_\mathsf{Zero}\overset{?}{=}0$

## Security Proof

### Completeness

If $Y_\mathsf{Zero}$ is zero, then $\mathcal{V}$ will accept. Therefore, to show completeness, we show that any prover who holds $\mathsf{In}$ that satisfies the circuit $\mathsf{T}$ can follow the steps outlined in the above protocol and the resulting $Y_\mathsf{Zero}$ will be equal to zero.  To see this, observed that $Y_\mathsf{Zero}$

$= \mathsf{Poly}_\mathsf{Vanish}(\zeta)-Q(\zeta)\cdot(\zeta^{\kappa}-1)$

$= [\mathsf{Poly}_\mathsf{T}(\zeta\omega^3)\cdot(\mathsf{Poly}_\mathsf{In}(\zeta)\cdot\mathsf{Poly}_\mathsf{T}(\zeta)+\mathsf{Poly}_\mathsf{In}(\zeta\omega)\cdot\mathsf{Poly}_\mathsf{T}(\zeta\omega))+(1-\mathsf{Poly}_\mathsf{T}(\zeta\omega^3))\cdot(\mathsf{Poly}_\mathsf{In}(\zeta)\cdot\mathsf{Poly}_\mathsf{In}(\zeta\omega)\cdot\mathsf{Poly}_\mathsf{T}(\zeta\omega^2))+\\\mathsf{Poly}_\mathsf{In}(\zeta\omega^2)-\mathsf{Poly}_\mathsf{In}(\zeta\omega^3)]\cdot\frac{\zeta^\kappa-1}{\zeta-\omega^0} -Q(\zeta)\cdot(\zeta^\kappa-1)$

$= [\mathsf{Poly}_\mathsf{T}(\zeta\omega^3)\cdot(\mathsf{Poly}_\mathsf{In}(\zeta)\cdot\mathsf{Poly}_\mathsf{T}(\zeta)+\mathsf{Poly}_\mathsf{In}(\zeta\omega)\cdot\mathsf{Poly}_\mathsf{T}(\zeta\omega))+(1-\mathsf{Poly}_\mathsf{T}(\zeta\omega^3))\cdot(\mathsf{Poly}_\mathsf{In}(\zeta)\cdot\mathsf{Poly}_\mathsf{In}(\zeta\omega)\cdot\mathsf{Poly}_\mathsf{T}(\zeta\omega^2))+\\\mathsf{Poly}_\mathsf{In}(\zeta\omega^2)-\mathsf{Poly}_\mathsf{In}(\zeta\omega^3)]\cdot\frac{\zeta^\kappa-1}{\zeta-\omega^0} - \frac{\mathsf{Poly}_\mathsf{Vanish}(\zeta)}{\zeta^\kappa-1} \cdot(\zeta^\kappa-1)$

$= [\mathsf{Poly}_\mathsf{T}(\zeta\omega^3)\cdot(\mathsf{Poly}_\mathsf{In}(\zeta)\cdot\mathsf{Poly}_\mathsf{T}(\zeta)+\mathsf{Poly}_\mathsf{In}(\zeta\omega)\cdot\mathsf{Poly}_\mathsf{T}(\zeta\omega))+(1-\mathsf{Poly}_\mathsf{T}(\zeta\omega^3))\cdot(\mathsf{Poly}_\mathsf{In}(\zeta)\cdot\mathsf{Poly}_\mathsf{In}(\zeta\omega)\cdot\mathsf{Poly}_\mathsf{T}(\zeta\omega^2))+\\\mathsf{Poly}_\mathsf{In}(\zeta\omega^2)-\mathsf{Poly}_\mathsf{In}(\zeta\omega^3)]\cdot\frac{\zeta^\kappa-1}{\zeta-\omega^0} \newline - [[\mathsf{Poly}_\mathsf{T}(\zeta\omega^3)\cdot(\mathsf{Poly}_\mathsf{In}(\zeta)\cdot\mathsf{Poly}_\mathsf{T}(\zeta)+\mathsf{Poly}_\mathsf{In}(\zeta\omega)\cdot\mathsf{Poly}_\mathsf{T}(\zeta\omega))+(1-\mathsf{Poly}_\mathsf{T}(\zeta\omega^3))\cdot(\mathsf{Poly}_\mathsf{In}(\zeta)\cdot\mathsf{Poly}_\mathsf{In}(\zeta\omega)\cdot\mathsf{Poly}_\mathsf{T}(\zeta\omega^2))+\\\mathsf{Poly}_\mathsf{In}(\zeta\omega^2)-\mathsf{Poly}_\mathsf{In}(\zeta\omega^3)]\cdot\frac{\zeta^\kappa-1}{\zeta-\omega^0} \cdot(\zeta^\kappa-1)]$

$=0$

Where the third equality relies on the fact that $\mathsf{Poly_{Vanish}}(X)$ is divisible by $X^\kappa -1$. This is true if $\mathsf{Poly_{Vanish}}(\zeta)$ is vanishing on $\mathcal{H}_\kappa$, i.e. if:

 $\mathsf{Poly}_\mathsf{T}(\zeta\omega^3)\cdot(\mathsf{Poly}_\mathsf{In}(\zeta)\cdot\mathsf{Poly}_\mathsf{T}(\zeta)+\mathsf{Poly}_\mathsf{In}(\zeta\omega)\cdot\mathsf{Poly}_\mathsf{T}(\zeta\omega))+(1-\mathsf{Poly}_\mathsf{T}(\zeta\omega^3))\cdot(\mathsf{Poly}_\mathsf{In}(\zeta)\cdot\mathsf{Poly}_\mathsf{In}(\zeta\omega)\cdot\mathsf{Poly}_\mathsf{T}(\zeta\omega^2))\\+\mathsf{Poly}_\mathsf{In}(\zeta\omega^2)-\mathsf{Poly}_\mathsf{In}(\zeta\omega^3)]\cdot\frac{\zeta^\kappa-1}{\zeta-\omega^0} = 0$

Which hold if, for $X=\omega^0$:

$ \mathsf{Poly}_\mathsf{T}(X\omega^3)\cdot(\mathsf{Poly}_\mathsf{In}(X)\cdot\mathsf{Poly}_\mathsf{T}(X)+\mathsf{Poly}_\mathsf{In}(X\omega)\cdot\mathsf{Poly}_\mathsf{T}(X\omega))+(1-\mathsf{Poly}_\mathsf{T}(X\omega^3))\cdot(\mathsf{Poly}_\mathsf{In}(X)\cdot\mathsf{Poly}_\mathsf{In}(X\omega)\cdot\mathsf{Poly}_\mathsf{T}(X\omega^2))\\+\mathsf{Poly}_\mathsf{In}(X\omega^2)=\mathsf{Poly}_\mathsf{In}(X\omega^3)$

Where we get the "for $X = \omega^0$" due to zeroing parts of the polynomials (see [zero1](../zero1.md)). Since $\mathsf{Poly_T}(\omega^i) = \mathsf{T}[i]$  and $\mathsf{Poly_{In}}(\omega^i) = \mathsf{In}[i]$, $\forall i \in [0, \kappa - 1]$, the above conditions are true if:

$\mathsf{T}[3]\cdot(\mathsf{In}[0]\cdot\mathsf{T}[0]+\mathsf{In}[1]\cdot\mathsf{T}[1])+(1-\mathsf{T}[3])\cdot(\mathsf{In}[0]\cdot\mathsf{In}[1]\cdot\mathsf{T}[2])+\mathsf{In}[2]=\mathsf{In}[3]$

But this means precisely that $\mathsf{In}$ satisfies the circuit $\mathsf{T}$, which was the condition we assumed about the prover. Thus, the $Y_\mathsf{Zero}$ it creates by following the protocol is zero, and its transcipt will be accepted.

### Soundness

We prove knowledge soundness in the Algebraic Group Model (AGM). To do so, we must prove that there exists an efficient extractor $\mathcal{E}$ such that for any algebraic adversary $\mathcal{A}$ the probability of $\mathcal{A}$ winning the following game is $\mathsf{negl}(\lambda)$.

1. Given $[g,g^\tau,g^{\tau^2},\dots,g^{\tau^{n-1}}]$, $\mathcal{A}$ outputs commitments to $\mathsf{Poly}_\mathsf{In}(X)$ and $Q$
2. $\mathcal{E}$, given access to $\mathcal{A}$'s outputs from the previous step, outputs $\mathsf{Poly}_\mathsf{In}(X)$ and $Q$
3. $\mathcal{A}$ plays the part of the prover in showing that $Y_\mathsf{Zero}$ is zero at a random challenge $\zeta$
4. $\mathcal{A}$ wins if
    * $\mathcal{V}$ accepts at the end of the protocol
    * the values in $\mathsf{In}$ do not satisfy the circuit

The proof is trivial: to make $\mathsf{Poly}_\mathsf{Vanish}$ exist, the values in $\mathsf{In}$ have to satisfy the circuit. By the Schwartz-Zippel lemma, the soundness error is $\kappa/|\mathbb{F}|$, which is negligible since $\kappa=4$ and $\mathbb{F}$ is enormous for any widely used elliptic curve.

### Zero-Knowledge

To prove the above protocol is zero-knowledge, we do so by constructing a probabilistic polynomial time simulator $\mathcal{S}$ which, for every (possibly malicious) verifier $\mathcal{V}$, can output a view of the execution of the protocol that is indistinguishable from the view produced by the real execution of the program.

The simulator $\mathcal{S}$ randomly generates $a$ and $b$ as the inputs of the circuit, and runs the circuit to compute the output $c$. Then $\mathcal{S}$ follows the same steps a prover would prove the lookup argument. $\mathcal{S}$ interpolates $\mathsf{Poly}_\mathsf{In^*}$ from $\mathsf{In^*}$. It computes $Q^*(X)$ and finally outputs the commitments to each of these polynomials (and writes the commitments to the transcript). Then, it generates the random challenge $\zeta^*$ (once again this is by strong Fiat-Shamir). It creates opening proofs for $\mathsf{Poly}_\mathsf{T^*}$ and $\mathsf{Poly}_\mathsf{In^*}$ at $\zeta^*,\zeta^*\omega,\zeta^*\omega^2,\zeta^*\omega^3$ respectively, and $Q^*(\zeta^*)$, and writes these to the transcript as well. Since $\mathcal{S}$ knows each of the above polynomials, it can honestly compute this step and the proof will be accepted by $\mathcal{V}$. The transcript it generates this way will be indistinguishable from a transcript generated from a real execution since $\mathsf{PolyCommit}_\mathsf{Ped}$ has the property of Indistinguishability of Commitments due to the randomization by $h^{\hat{\phi}(x)}$.