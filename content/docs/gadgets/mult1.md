# Multiplication (Type 1)

## Recap of types

| Type                | Description                                                  | Recap                                                        | This |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| [mult1](./mult1.md) | $\mathsf{Arr}_3=\mathsf{Arr}_1 \cdot \mathsf{Arr}_2$         | $\mathsf{Arr}_3$ is the element-wise multiplication of $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$. | ✅    |
| [mult2](./mult2)    | $\mathsf{Prod}_\mathsf{Arr}=\prod_{i = 0}^{n-1} \mathsf{Arr}[i]$ | $\mathsf{Prod}_\mathsf{Arr}$ is the disclosed product of all the elements in $\mathsf{Arr}$. |      |
| [mult3](./mult3)    | $\prod_{i = 0}^{n-1} \mathsf{Arr}_1[i]=\prod_{i = 0}^{n-1} \mathsf{Arr}_2[i]$ | $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$ have the same undisclosed product. |      |

## Relation

$ \mathcal{R}_{\mathtt{mult2}} := \left\{ \begin{array}{l} (K_\mathsf{Arr_1},K_\mathsf{Arr_2},K_\mathsf{Arr_3}) \end{array} \middle | \begin{array}{l} \mathsf{Arr_3}[i]=\mathsf{Arr_1}[i]\cdot\mathsf{Arr_2}[i], 0\leq i \leq n-1, \\ \mathsf{Poly}_\mathsf{Arr_j}=\mathsf{FFT.Interp}(\omega,\mathsf{Arr_j}), 1\leq j \leq 3, \\ K_\mathsf{Arr_j}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_j}), 1\leq j \leq 3, \end{array} \right\} $

## Intuition

The prover ($\mathcal{P}$) holds two arrays $\mathsf{Arr_1}$ and $\mathsf{Arr_2}$ of $n$ integers from $\mathbb{Z}_q$: $[a_0, a_1, a_2, \dots, a_{n-1}]$. It will produce a succinct (independent of $n$) proof that $\mathsf{Arr_3}$ is the element-wise product of all the elements in the array: $\mathsf{Arr_3}[i]=\mathsf{Arr_1}[i]\cdot\mathsf{Arr_2}[i]$. The prover will encode the three arrays into three polynomials: $\mathsf{Poly}_\mathsf{Arr_1}$, $\mathsf{Poly}_\mathsf{Arr_2}$, and $\mathsf{Poly}_\mathsf{Arr_3}$ (using [evaluation points](../background/poly-iop) on the domain $\mathcal{H}_\kappa$). It will commit to each polynomial: $K_\mathsf{Arr_1}$, $K_\mathsf{Arr_2}$, and $K_\mathsf{Arr_3}$. The verifier ($\mathcal{V}$) cannot check any of the $\mathsf{Arr_i}$ or $\mathsf{Poly}_\mathsf{Arr_i}$ values directly (they may contain secret information, and even if they do not, they are too long to check) so the verifier only sees $K_\mathsf{Arr_1}$,$K_\mathsf{Arr_2}$, and $K_\mathsf{Arr_3}$.

In order to prove$K_\mathsf{Arr_1}$,$K_\mathsf{Arr_2}$, and $K_\mathsf{Arr_3}$ are consistent, the prover will compute the difference between $(\mathsf{Poly}_\mathsf{Arr_1}\cdot\mathsf{Poly}_\mathsf{Arr_2})$ and $(\mathsf{Poly}_\mathsf{Arr_3})$ using [add1](./add1). Next, it will show it is 0 for each evaluation point in the domain $\mathcal{H}_\kappa$. Showing a polynomial is zero on the domain (a "vanishing polynomial") is a common sub-protocol used by many gadgets.

## Protocol Details

### Array Level

* $\mathcal{P}$ holds an array $\mathsf{Arr_1} = [a_{(1,0)}, a_{(1,1)}, a_{(1,2)}, \dots, a_{(1,n-1)}]$ of $n$ integers ($a_{(1,i)} \in \mathbb{Z}_q$​)
* $\mathcal{P}$ holds an array $\mathsf{Arr_2} = [a_{(2,0)}, a_{(2,1)}, a_{(2,2)}, \dots, a_{(2,n-1)}]$ of $n$ integers ($a_{(2,i)} \in \mathbb{Z}_q$)
* $\mathcal{P}$ computes or holds an array $\mathsf{Arr_3} = [a_{(3,0)}, a_{(3,1)}, a_{(3,2)}, \dots, a_{(3,n-1)}]$ of $n$ integers ($a_{(3,i)} \in \mathbb{Z}_q$) such that:
  * $\mathsf{Arr_3}[i]=\mathsf{Arr_1}[i]\cdot\mathsf{Arr_2}[i]$ for $i$ from 0 to $n-1$

### Polynomial Level

We assume the three arrays $\mathsf{Arr_1}$, $\mathsf{Arr_2}$ and $\mathsf{Arr_3}$ are encoded as the y-coordinates into a univariant polynomial where the x-coordinates (called the domain $\mathcal{H}_\kappa$) are chosen as the multiplicative group of order $\kappa$ with generator $\omega\in\mathbb{G}_\kappa$ (see [Background](../background/poly-iop.md) for more). In short, $\omega^0$ is the first element and $\omega^{\kappa-1}$ is the last element of $\mathcal{H}_\kappa$. If $\kappa$​ is larger than the length of the array, the array can be padded.

Recall the constraint we want to prove: 

1. $\mathsf{Arr_3}[i]=\mathsf{Arr_1}[i]\cdot\mathsf{Arr_2}[i]$ for $i$ from 0 to $n-1$

In polynomial form, the constraint is:

1. For all $X$ from $\omega^0$ to $\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Arr_3}(X)=\mathsf{Poly}_\mathsf{Arr_1}(X)\cdot\mathsf{Poly}_\mathsf{Arr_2}(X)$ 

We adjust the constraints to show an equality with 0:

1. For all $X$ from $\omega^0$ to $\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Vanish}(X)=\mathsf{Poly}_\mathsf{Arr_3}(X)-\mathsf{Poly}_\mathsf{Arr_1}(X)\cdot\mathsf{Poly}_\mathsf{Arr_2}(X)=0$ 

This equation is true for every value of $X \in \mathcal{H}_\kappa$ (but not necessarily true outside of these values). To show this, we divide each polynomial by  $X^\kappa - 1$, which is a minimal vanishing polynomial for $\mathcal{H}_\kappa$ that does not require interpolation to create. If the quotients are polynomials (and not rational functions), then $\mathsf{Poly}_\mathsf{Vanish}(X)$ must be vanishing on $\mathcal{H}_\kappa$ too. Specifically, the prover computes:

1. $Q(X) = \frac{\mathsf{Poly}_\mathsf{Vanish}(X)}{X^\kappa - 1}$

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