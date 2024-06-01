**Array Level**

The prover has a list of value $A = [a_0, a_1, a_2, \dots, a_{n-1}]$  and wishes to commit to and prove that product of all entries in $A$ equals some value $z$. In other words, that $\prod_{i = 0}^{n-1} a_i= z$. First, the prover will compute an accumulator, $B$ according to the following rules: $i)  B_i = A_i \cdot B_{i+1} \space 0 \leq i \lt n-1$ and $ii)B_{n-1} = A_{n-1}$. Then $B_0 = z$. To prove that $\prod_{i = 0}^{n-1} a_i= z$, it suffices for the prover to demonstrate that $B_0 = z$ and that $i)$ and $ii)$ hold, and that $B_0 = z$.

**Polynomial Level**

The prover interpolates $A$ and $B$ to get $P_A(x)$ and $P_B(x)$. To do so, an $n$th root of unity (which we will call $\omega$) is used, and the values of $A$ and $B$ are paired with the powers of this root of unity as follows: $(\omega^0, a_0), (\omega^1, a_1), \dots, (\omega^{n-1}, a_{n-1})$ and similarly for $B$. Then the interpolation (using the fast fourier transform (FFT)) creates a polynomial that passes through the points represented by these tuples. We also define $H$ as the set $[\omega^0, \omega^1, \dots, \omega^{n-1}]$. We are assuming that $n$ is a power of 2, so that we can use FFT. In practice, if it is not, then a power of 2 greater than $n$, say $2^m$, is used and the last $2^m - n$ entries of $A$ are padded with 1s.

The prover then compute $W_1(x) = [P_B(x) - P_B(x\omega) \cdot P_a(x))] \cdot (x - \omega^{n-1})$ and $W_2(x) = [P_A(x) - P_B(x)] \cdot \frac{x^n - 1} {x - w^{n-1}}$. If $W_1$ is vanishing on $H$, then $i)$ holds for $0 \leq i \lt n$. We note that the $(x - \omega^{n-1})$ is included in the equation to add a root at $x = n -1$, since $i)$ does not hold for $i = n -1 $ (unless $z = 1$, in which case the ($x - \omega^{n-1}$) is unnecessary). Similarly, If $W_2$ is vanishing on $H$, then $ii)$ holds. We note that multiplying [$P_A(x) - P_B(x)]$ by $\frac{x^n - 1} {x - w^{n-1}}$ zeroes $W_2$ on all of $H$ except $\omega^{n-1}$, thus $W_2$ vanishing on $H$ is testing only the equality of $A_{n-1}$ and $B_{n-1}$​.

The prover then computes $Q_1(x) = \frac{W_1(x)}{x^n - 1}$ and $Q_2(x) = \frac{W_2(x)}{x^n - 1}$. Since $x^n - 1$ is the vanishing polynomial for $H$, if $Q_1$ and $Q_2$ are polynomials (and not rational functions), then $W_1$ and $W_2$ must be vanishing on $H$.

The prover then commits to $P_A$, $P_B$, $Q_1$, and $Q_2$. They then hash the four commitments to get a random challenge, $\tau$. They open $P_A(\tau), P_B(\tau), Q_1(\tau), Q_2(\tau),$ and $P_B(\tau \cdot \omega)$ to demonstrate that $Q_1$ and $Q_2$ are polynomials, as defined above $-$ thus $i)$ and $ii)$ hold. They also open $P_B(1)$, showing that $B_0 = z$. Thus they have demontrated that $\prod_{i = 0}^{n-1} a_i= z$.

**Verifier Level**

The verifier first checks that $\tau$​ is correct by hashing the four commitments which have been sent to them by the prover.

Next, the verifier checks that the evaluations $P_A(\tau), P_B(\tau), Q_1(\tau), Q_2(\tau),$ and $P_B(\tau \cdot \omega)$​ sent to them by the prover are correct. This can be done (assuming we are working with KZG) as a single batch opening. 

The verifier will then check the following two equalities, so make sure that $Q_1$ and $Q_2$​ were defined as they should be:

​	$a) \space [P_B(\tau) - P_B(\tau \cdot \omega)\cdot P_A(\tau)] \cdot (\tau - \omega^{n-1}) = (\tau^n - 1) \cdot Q_1(\tau)$​ 

​	$b) \space \frac{P_A(\tau) - P_B(\tau)}{\tau - \omega^{n-1}} = Q_2(\tau)$

Finally, the verifier will check the opening of $P_B(1)$. This value is $z$, and they have verified that it is equal to the product of the entries in $A$.