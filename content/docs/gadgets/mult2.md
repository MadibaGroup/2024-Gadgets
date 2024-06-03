# Multiplication (Type 2)

## Recap of types



| Type   | Description                                                  | Recap                                                        | This |
| ------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ---- |
| Type 1 | $\mathsf{Arr}_3=\mathsf{Arr}_1 \cdot \mathsf{Arr}_2$         | $\mathsf{Arr}_3$ is the element-wise multiplication of $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$. |      |
| Type 2 | $\mathsf{Prod}_\mathsf{Arr}=\prod_{i = 0}^{n-1} \mathsf{Arr}[i]$ | $\mathsf{Prod}_\mathsf{Arr}$ is the disclosed product of all the elements in $\mathsf{Arr}$. | ✅    |
| Type 3 | $\prod_{i = 0}^{n-1} \mathsf{Arr}_1[i]=\prod_{i = 0}^{n-1} \mathsf{Arr}_2[i]$ | $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$ have the same undisclosed product. |      |



## Relation



$ \mathcal{R}_{\mathtt{mult2}} := \left\{ \begin{array}{l} (K_\mathsf{Arr},\mathsf{Prod}_\mathsf{Arr}) \end{array} \middle | \begin{array}{l} \mathsf{Arr} = [a_0, a_1, a_2, \dots, a_{n-1}], \\ \mathsf{Prod}_\mathsf{Arr} = \prod_{i = 0}^{n-1} a_i, \\ \mathsf{Poly}_\mathsf{Arr}=\mathsf{FFT.Interp}(\omega,\mathsf{Arr}), \\ K_\mathsf{Arr}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr}) \end{array} \right\} $



## Intuition

The prover ($\mathcal{P}$) holds an array $\mathsf{Arr} = [a_0, a_1, a_2, \dots, a_{n-1}]$ and a public value $\mathsf{Prod}_\mathsf{Arr}$. It will produce a succinct (independent of $n$) proof that $\mathsf{Prod}_\mathsf{Arr}$ is the product of all the elements in the array. The prover will encode the array into a polynomial $\mathsf{Poly}_\mathsf{Arr}$ (using evaluation points) and commit to the polynomial $K_\mathsf{Arr}$. The verifier ($\mathcal{V}$) cannot check $\textsf{Arr}$ or $\mathsf{Poly}_\mathsf{Arr}$ directly (they may contain secret information, and even if they do not, it is too long to check) so the verifier only sees $K_\mathsf{Arr}$ and the asserted value $\mathsf{Prod_\mathsf{Arr}}$.

In order to prove $K_\mathsf{Arr}$ and  $\mathsf{Prod}_\mathsf{Arr}$ are consistent, the prover will build a helper array $\mathsf{Acc}_\mathsf{Arr}$ called an accumulator (or accumulating array or incremental array). This should not be confused with accumulators from cryptography, which are a concept related to succinct proofs but are distinct. As with $\mathsf{Arr}$, the prover will also encode $\mathsf{Acc}$ as a polynomial and provide a commitment of it to the verifier. The idea is that the prover will prove a relation between $\mathsf{Arr}$ and $\mathsf{Acc}$; and a relation between $\mathsf{Acc}$ and $\mathsf{Prod_\mathsf{Arr}}$. Put together, it will imply the correct relation between $\mathsf{Arr}$ and $\mathsf{Prod_\mathsf{Arr}}$​.

Consider a small numeric example in $\mathbb{Z}_{97}$ where  $\mathsf{Arr}= [84,67,11,92,36,67]$ and $\mathsf{Prod}_\mathsf{Arr}=72$. The first idea is to get $\mathsf{Prod}_\mathsf{Arr}$ into an array. Say, we just append it: $\mathsf{Arr}''= [84,67,11,92,36,67,72] $. How does the prover show $\mathsf{Arr}''$ is correct? The last value of the array depends on every single element that precedes it, which will be a complex constraint to prove. 

An alternative idea is to create a new array that starts the same as $\mathsf{Arr}$ and ends up at $\mathsf{Prod}_\mathsf{Arr}$ by folding in the integers from $\mathsf{Arr}$ one-by-one with multiplication. Then each value in the new array will depend on only two values, as below. 

The first value in $\mathsf{Acc}$ will be a copy of the first value from $\mathsf{Arr}$:

* $\mathsf{Arr}= [84,67,11,92,36,67]$

* $\mathsf{Acc}= [84, \bot,\bot,\bot,\bot,\bot] $​

The next value will be the multiplication (mod 97) of: 67 (the value at the same index in $\mathsf{Arr}$) and 84 (the previous value in $\mathsf{Acc}$​):

* $\mathsf{Arr}= [84,67,11,92,36,67]$

* $ \mathsf{Acc} = [84, (67\cdot84),\bot,\bot,\bot,\bot] = [84, 2,\bot,\bot,\bot,\bot]$ 

The next value will be the multiplication of: 11 (the value at the same index in $\mathsf{Arr}$) and 2 (the previous value in $\mathsf{Acc}$):

* $\mathsf{Arr}= [84,67,11,92,36,67]$
* $ \mathsf{Acc} = [84, 2,(11\cdot2),\bot,\bot,\bot] = [84,2,22,\bot,\bot,\bot]$​​ 
* $ \mathsf{Acc} = [84, 2,22,(92\cdot22),\bot,\bot] = [84,2,22,84,\bot,\bot]$ 
* $ \mathsf{Acc} = [84, 2,22,84,(36\cdot84),\bot] = [84,2,22,84,17,\bot]$​ 
* $ \mathsf{Acc} = [84,2,22,84,17,(67\cdot17)] = [84, 2, 22, 84, 17, 72]$​ 
* $\mathsf{Prod}_\mathsf{Arr}=72$

Notice the last value in $\mathsf{Acc}$ is $\mathsf{Prod_\mathsf{Arr}}$. The prover wants to show three constraints:

1. The first value in $\mathsf{Acc}$ matches the first value in $\mathsf{Arr}$,
2. The rest of the values in $\mathsf{Acc}$ are of the form $\mathsf{Acc}[i]=\mathsf{Arr}[i]\cdot\mathsf{Acc}[i-1]$,
3. The last value in $\mathsf{Acc}$ matches $\mathsf{Prod}_\mathsf{Arr}$​​.

If all three constraints are true, then $\mathsf{Prod}_\mathsf{Arr}$ is the product of the elements of $\mathsf{Arr}$. 

Last, while it is not necessary to do, it is often convenient to hold the the value $\mathsf{Prod}_\mathsf{Arr}$ at the start of the array $\mathsf{Acc}$ instead of the end. For this reason, the mathematical explaination below will construct $\mathsf{Acc}$ "backwards" (or right-to-left) from the above example, where the last value of $\mathsf{Acc}$ matches the last value of $\mathsf{Arr}$, the values are folded in from right to left, and the first (leftmost) value of $\mathsf{Acc}$ is $\mathsf{Prod}_\mathsf{Arr}$:

* $\mathsf{Arr}= [84,67,11,92,36,67]$​
* $ \mathsf{Acc} = [\bot, \bot, \bot, \bot, \bot, 67]$ 
* $ \mathsf{Acc} = [\bot, \bot, \bot, \bot, 84, 67]$ 
* $\ldots$
* $ \mathsf{Acc} = [72, 84, 36, 65, 84, 67]$​ 
* $\mathsf{Prod}_\mathsf{Arr}=72$



## Array Level

The prover has a list of value $A = [a_0, a_1, a_2, \dots, a_{n-1}]$  and wishes to commit to and prove that product of all entries in $A$ equals some value $z$. In other words, that $\prod_{i = 0}^{n-1} a_i= z$. First, the prover will compute an accumulator, $B$ according to the following rules: $i)  B_i = A_i \cdot B_{i+1} \space 0 \leq i \lt n-1$ and $ii)B_{n-1} = A_{n-1}$. Then $B_0 = z$. To prove that $\prod_{i = 0}^{n-1} a_i= z$, it suffices for the prover to demonstrate that $B_0 = z$ and that $i)$ and $ii)$ hold, and that $B_0 = z$​.



## Polynomial Level

We assume arrays are encoded as y-coordinates into a univariant polynomial where the x-coordinates are chosen as the multiplicative group of order $\kappa$ with generator $\omega\in\mathbb{G}_\kappa$ (see [Background](../background/poly-iop.md) for more). In short, $\omega^0$ is the first element and $\omega^{\kappa-1}$ is the last element. 

In polynomial form, the three constraints are:

1. For $X=w^\kappa$: $\mathsf{Poly}_\mathsf{Acc}(X)=\mathsf{Poly}_\mathsf{Arr}(X)$,
2. For all $X$ except $X=\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc}(X)=\mathsf{Poly}_\mathsf{Arr}(X)\cdot\mathsf{Poly}_\mathsf{Acc}(\omega\cdot X)$ 
3. For $X=w^0$: $\mathsf{Poly}_\mathsf{Acc}(X)=\mathsf{Prod}_\mathsf{Arr}$

We adjust each of these constraints to show an equality with 0:

1. For $X=w^\kappa$: $\mathsf{Poly}_\mathsf{Acc}(X)-\mathsf{Poly}_\mathsf{Arr}(X)=0$,
2. For all $X$ except $X=\omega^{\kappa-1}$: $\mathsf{Poly}_\mathsf{Acc}(X)-\mathsf{Poly}_\mathsf{Arr}(X)\cdot\mathsf{Poly}_\mathsf{Acc}(\omega\cdot X)=0$ 
3. For $X=w^0$: $\mathsf{Poly}_\mathsf{Acc}(X)-\mathsf{Prod}_\mathsf{Arr}=0$​

Next we take care of the "for $X$​" conditions by zeroing out the rest of the polynomial that is not zero. See the gadget: [Zero-ing](./zero.md) for more on why this works.

1. $\mathsf{Poly}_\mathsf{Vanish1}=(\mathsf{Poly}_\mathsf{Acc}(X)-\mathsf{Poly}_\mathsf{Arr}(X))\cdot\frac{(X^\kappa-1)}{(X-\omega^{\kappa-1})}=0$,
2. $\mathsf{Poly}_\mathsf{Vanish2}=(\mathsf{Poly}_\mathsf{Acc}(X)-\mathsf{Poly}_\mathsf{Arr}(X)\cdot\mathsf{Poly}_\mathsf{Acc}(\omega\cdot X))\cdot(X-\omega^{\kappa-1})=0$ 
3. $\mathsf{Poly}_\mathsf{Vanish3}=(\mathsf{Poly}_\mathsf{Acc}(X)-\mathsf{Prod}_\mathsf{Arr})\cdot\frac{(X^\kappa-1)}{(X-\omega^0)}=0$

These equations are true for every value of $X \in \mathbb{G}_\kappa$ (but not necessarily true outside of these values) show we show each is a vanishing polynomial. 



---



The prover interpolates $A$ and $B$ to get $P_A(x)$ and $P_B(x)$. To do so, an $n$th root of unity (which we will call $\omega$) is used, and the values of $A$ and $B$ are paired with the powers of this root of unity as follows: $(\omega^0, a_0), (\omega^1, a_1), \dots, (\omega^{n-1}, a_{n-1})$ and similarly for $B$. Then the interpolation (using the fast fourier transform (FFT)) creates a polynomial that passes through the points represented by these tuples. We also define $H$ as the set $[\omega^0, \omega^1, \dots, \omega^{n-1}]$. We are assuming that $n$ is a power of 2, so that we can use FFT. In practice, if it is not, then a power of 2 greater than $n$, say $2^m$, is used and the last $2^m - n$ entries of $A$ are padded with 1s.

The prover then compute $W_1(x) = [P_B(x) - P_B(x\omega) \cdot P_a(x))] \cdot (x - \omega^{n-1})$ and $W_2(x) = [P_A(x) - P_B(x)] \cdot \frac{x^n - 1} {x - w^{n-1}}$. If $W_1$ is vanishing on $H$, then $i)$ holds for $0 \leq i \lt n$. We note that the $(x - \omega^{n-1})$ is included in the equation to add a root at $x = n -1$, since $i)$ does not hold for $i = n -1 $ (unless $z = 1$, in which case the ($x - \omega^{n-1}$) is unnecessary). Similarly, If $W_2$ is vanishing on $H$, then $ii)$ holds. We note that multiplying [$P_A(x) - P_B(x)]$ by $\frac{x^n - 1} {x - w^{n-1}}$ zeroes $W_2$ on all of $H$ except $\omega^{n-1}$, thus $W_2$ vanishing on $H$ is testing only the equality of $A_{n-1}$ and $B_{n-1}$​.

The prover then computes $Q_1(x) = \frac{W_1(x)}{x^n - 1}$ and $Q_2(x) = \frac{W_2(x)}{x^n - 1}$. Since $x^n - 1$ is the vanishing polynomial for $H$, if $Q_1$ and $Q_2$ are polynomials (and not rational functions), then $W_1$ and $W_2$ must be vanishing on $H$.

The prover then commits to $P_A$, $P_B$, $Q_1$, and $Q_2$. They then hash the four commitments to get a random challenge, $\tau$. They open $P_A(\tau), P_B(\tau), Q_1(\tau), Q_2(\tau),$ and $P_B(\tau \cdot \omega)$ to demonstrate that $Q_1$ and $Q_2$ are polynomials, as defined above $-$ thus $i)$ and $ii)$ hold. They also open $P_B(1)$, showing that $B_0 = z$. Thus they have demontrated that $\prod_{i = 0}^{n-1} a_i= z$​​.



### Optimized

Here, rework each constraint so it is a vanishing polynomial.



## Commitment Level

The verifier first checks that $\tau$​ is correct by hashing the four commitments which have been sent to them by the prover.

Next, the verifier checks that the evaluations $P_A(\tau), P_B(\tau), Q_1(\tau), Q_2(\tau),$ and $P_B(\tau \cdot \omega)$​ sent to them by the prover are correct. This can be done (assuming we are working with KZG) as a single batch opening. 

The verifier will then check the following two equalities, so make sure that $Q_1$ and $Q_2$​ were defined as they should be:

​	$a) \space [P_B(\tau) - P_B(\tau \cdot \omega)\cdot P_A(\tau)] \cdot (\tau - \omega^{n-1}) = (\tau^n - 1) \cdot Q_1(\tau)$​ 

​	$b) \space \frac{P_A(\tau) - P_B(\tau)}{\tau - \omega^{n-1}} = Q_2(\tau)$

Finally, the verifier will check the opening of $P_B(1)$. This value is $z$, and they have verified that it is equal to the product of the entries in $A$​.



## Implementations

* [Rust]()
* [Mathematica]() (Toy Example)



## Security Proof



### Completeness

### Soundness

### Zero-Knowledge
