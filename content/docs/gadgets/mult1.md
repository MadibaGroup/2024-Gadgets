## Security Proof

### Completeness

Any honest prover can do the computations explained above and create an accepting proof.

### Soundness

We prove knowledge soundness in the Algebraic Group Model (AGM). To do so, we must prove that there exists an efficient extractor $E$ such that for any algebraic adversary $A$ the probability of $A$ winning the following game is $\mathsf{negl}(\lambda)$.

1. Given $[g, g^\tau, g^{\tau^2}, \dots,g^{\tau^{n-1}}]$ $A$ outputs commitments to $\mathsf{Poly}_\mathsf{Acc}(X)$, $\mathsf{Poly}_\mathsf{Arr}(X)$, $Q$.

2. $E$, given access to $A$'s outputs from the previous step, outputs $\mathsf{Poly}_\mathsf{Acc}(X)$, $\mathsf{Poly}_\mathsf{Arr}(X)$, $Q$.

3. $A$ plays the part of the prover in showing that $Y_{\mathsf{Zero}}$ is zero at a random challenge $\zeta$

4. $A$ wins if: 

   i) $V$ accepts at the end of the protocol

   ii) $\mathsf{Arr}_3\neq \mathsf{Arr}_1 \cdot \mathsf{Arr}_2$

Our proof is as follows. Each commitment $A$ outputs in step 1 is a linear combination of the elements in $[g, g^\tau, g^{\tau^2}, \dots,g^{\tau^{n-1}}]$. $E$ is given these coefficients (since $A$ is an algebraic adversary) so $E$ can output the original polynomials.

Next, $A$ obtains a random challenge $\zeta$ using strong Fiat-Shamir. They must now conduct an opening proof for ${\mathsf{Poly}_\mathsf{Arr}(\zeta)}, {\mathsf{Poly}_\mathsf{Acc}(\zeta)}$ and ${\mathsf{Poly}_\mathsf{Arr}(\zeta \cdot \omega)}$. (This can also be done as a batch opening, but for simplicity we will look at individual openings and the logic will extend to the batch opening optimization). 

For the second win condition to be fulfilled, one of the three constraints must be false. But then the $\mathsf{Poly}_\mathsf{Vanish}$ corresponding to that constraint is not vanishing on $\mathcal{H}_\kappa$, so $Q(X)$ is not a polynomial (it is a rational function). This means that $A$ cannot calcuated the correct commitment value $g^{Q(\tau)}$ without solving the t-SDH. Thus, $A$ chooses an arbitrary value to send as $K_Q$. By the binding property of KZG commitments, $\mathsf{Poly}_\mathsf{Acc}(\zeta)$,  $\mathsf{Poly}_\mathsf{Arr}(\zeta)$, and $\mathsf{Poly}_\mathsf{Acc}(\zeta \cdot \omega)$ can only feasibliy be opened to one value each; therefore, for $A$ to have the verifier accept, they must send a proof that $Q(\zeta)$ opens to $Q(\zeta) = \frac{Y_\mathsf{Vanish1} + \rho Y_\mathsf{Vanish2} + \rho^2Y_\mathsf{Vanish3}}{\zeta^n - 1}$. This means being able to send $g^{q(\tau)}$ where $q(\tau) = \frac{Q(\tau) - Q(\zeta)}{\tau - \zeta}$ and $Q(\zeta) = \frac{Y_\mathsf{Vanish1} + \rho Y_\mathsf{Vanish2} + \rho^2Y_\mathsf{Vanish3}}{\zeta^n - 1}$. Since $Q(\tau)$ and $Q(\zeta)$ are known, this implies knowing $g^{\frac{1}{\tau - \zeta}}$. Thus the prover would have found $\langle\zeta,g^{\frac{1}{\tau - \zeta}}\rangle$, which is the t-SDH problem. We have shown that creating an accepting proof reduces to the t-SDH, so $A$'s probability of success is negligible.

- Should I show how using with the ability described above $P$ would enable one to construct an algortihm that break t-BSDH?

### Zero-Knowledge

We prove that the above protocol is zero-knowledge when $\mathsf{PolyCommit}_\mathsf{Ped}$ (from the KZG paper) is used for the polynomial commitments. We do so by constructing a probabilistic polynomial time simulator $S$ that knows the trapdoor $\tau$, which, for every (possibly malicious) verifier $V$​, can output a view of the execution of the protocol that is indistinguishable from the view produced by the real execution of the program.

The simulator $S$ choose arbitrary values for ${\mathsf{Poly}_\mathsf{Arr}(\tau)}$ and ${\mathsf{Poly}_\mathsf{Acc}(\tau)}$, then computes $g^{\mathsf{Poly}_\mathsf{Arr}(\tau)}$ and $g^{\mathsf{Poly}_\mathsf{Acc}(\tau)}$ to output as the commitments $K_\mathsf{Arr}$ and $ K_\mathsf{Acc}$. $S$ then generates the challenge evaluation pount $\rho$ using string Fiat-Shamir, and computes $Q(\tau)$ using $\rho$ and the values they chose for ${\mathsf{Poly}_\mathsf{Arr}(\tau)}$ and ${\mathsf{Poly}_\mathsf{Acc}(\tau)}$. $S$ outputs the commitment $K_Q = g^{Q(\tau)}$.

Now, $S$ generates the second random challenge point $\zeta$, once again using strong Fiat-Shamir. $S$ then create fake opening proofs for ${\mathsf{Poly}_\mathsf{Arr}(\zeta)}, {\mathsf{Poly}_\mathsf{Acc}(\zeta)}$ and ${\mathsf{Poly}_\mathsf{Arr}(\zeta \cdot \omega)}$ to arbitrary values. This is done using the knowledge of $\tau$, calculating the witness $q(\tau) = \frac{{f(\tau) - f(\zeta)}}{\tau - \zeta}$ for ${\mathsf{Poly}_\mathsf{Arr}(\zeta)}$ and ${\mathsf{Poly}_\mathsf{Acc}(\zeta)}$, and the same but with $\zeta \cdot \omega$ for  ${\mathsf{Poly}_\mathsf{Arr}(\zeta \cdot \omega)}$.

Finally, $S$ creates a fake opening proof for $Q(\zeta) = \frac{Y_\mathsf{Vanish1} + \rho Y_\mathsf{Vanish2} + \rho^2Y_\mathsf{Vanish3}}{\zeta^n - 1}$. This is done using knowledge of $\tau$ to calculate an accepting witness $q(\tau)$, as above. This means that $Y_\mathsf{Zero}$ will be zero, and the transcript will be accepted by the verifier. It is indistinguishable from a transcript generates from a real execution, since $\mathsf{PolyCommit}_\mathsf{Ped}$ has the property of Indistinguishability of Commitments due to the randomization by $h^{\hat{\phi}(x)}$. 

- do we want to talk about perfect vs statistically vs computationally indistinguishable?
- should I be more formal in the proof? (e.g. writing out the indistinguishable in terms of probability distributions, etc)



Here is my previous (unsuccessful) attempt at a simulator. I've left it in case the above attempt is incorrect and we need to work with a more complicated simulator using rewinding, in which case perhaps the ideas below are still useful.

- We define a simulator $S$ which selects a random value $\zeta'$ and compute commitments to  $\mathsf{Poly}_\mathsf{Acc}(X)$, $\mathsf{Poly}_\mathsf{Arr}(X)$, $Q_1$, $Q_2$, and $Q3$ as well as $\mathsf{Poly}_\mathsf{Acc}(\zeta'), \mathsf{Poly}_\mathsf{Arr}(\zeta'), Q_1(\zeta'), Q_2(\zeta'),$ $Q_3(\zeta')$, and $\mathsf{Poly}_\mathsf{Acc}(\zeta' \cdot \omega)$ such that the three equalities $a), b),$ and $c)$ above hold and each polynomial correctly opens to it's evaluation at $\zeta$. It then hashes the commitments to the five polynomials to get the random challenge $\zeta$; if $\zeta' \neq \zeta$, then it rewinds and selects a new random value $\zeta'$.
- Expected number of iterations to get a random value $\zeta'$ such that $\zeta' = \zeta$ is $| \mathbb{F} |$, so we rewind and run it up to $| \mathbb{F} |$ times (or $n \cdot | \mathbb{F} |$) times for some value $n$?). It outputs fail if it has not succeeded in generating $\zeta'$ such that $\zeta'  = \zeta$ by then. Thus it terminates after at most $| \mathbb{F} |$ iterations (or $n \cdot |\mathbb{F}|$ iterations?) and it fails with negligible probability.
- The distribution for the view from this simulation is (perfectly? computationally?) indistinguishable from that of a real execution (when we use Pedersen commitments to make it zk), since Pedersen commmitments are perfectly hiding.