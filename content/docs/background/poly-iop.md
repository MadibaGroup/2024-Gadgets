# Notes on Polynomial Interactive Oracle Proofs (Draft)


## Encoding Arrays of Data into Polynomials

Data starts as a array of integers where integers in each slot are between 0 and $q-1$, where $q$ is a large (generally 256 bit) prime number. Recall that we call this set of integers $\mathbb{Z}_q$.

| $\mathsf{data}_0$ | $\mathsf{data}_1$ | $\mathsf{data}_2$ | $\mathsf{data}_3$ | $\mathsf{data}_4$ |
| ----------------- | ----------------- | ----------------- | ----------------- | ----------------- |

It is common to denote a polynomial like $P(X)$ where $X$ is the variable of the polynomial. We are going denote the variable with an empty box $\square$ which can be interpreted as a place where you can put any integer in $\mathbb{Z}_q$ you want evaluated (or equivalent, where you place an x-coordinate to learn what the y-coordinate is). 

A polynomial in this notation looks like:

* $P(\square)=c_0+c_1\cdot\square+c_2\cdot\square^2+c_3\cdot\square^3+c_4\cdot\square^4=\sum_{i=0}^d c_i\cdot\square^i$  

The values $c_i$ are called coefficients. Different arrays of data will (depending on how data is encoded, next) result in different coefficients and thus different polynomials. The degree of the polynomial is the largest exponent. So the polynomial above has degree 4 and thus will have 5 coefficients and 5 terms of the form $c_i\cdot\square^i$ (including $i=0$). Sometimes the coefficient will be zero: the term is thus not written down but in a list of coefficients, it will be included as a 0. 

The main question to tackle is how to "encode" a array of integers into a polynomial. This is generally done one of three ways:

1. Coefficients
2. Evaluation Points
3. Roots

Each has its advantages and disadvantages, which we discuss next. 

Fast forwarding a bit, once the polynomial is created, it is not shared directly with anyone. Instead a commitment to it is shared. The commitment does two things: (1) it makes it succinct: e.g., constant size regardless of how long the array is; and (2) it can hide the data in the array as necessary. We will discuss one specific polynomial commitment scheme called KZG. KZG needs the polynomial in the format of a list of its coefficients. If we have the polynomial in a different form, we will have to convert to coefficients. Thus this needs to be considered when weighing the pros/cons of the three encoding methods.



### Encoding 1: Coefficients

Create polynomial as: $P_1(\square)=\mathsf{data}_0+\square+\mathsf{data}_2\cdot\square^2+\mathsf{data}_3\cdot\square^3+\mathsf{data}_4\cdot\square^4=\sum_{i=0}^d \mathsf{data}_i\cdot\square^i$  

Properties:

* Fast 👍: fastest path to commitment as the output is already in coefficient form.
* Addition 👍: two arrays can be added together (slot-by-slot) by simply adding the polynomials together
* Multiplication 👎: no support for multiplication of arrays (Remark: multiplying the polynomials does not multiply the coefficients. It results in a cross multiplication of every term in the first polynomial with every term in the second polynomial. Further the degree of the resulting polynomial will double that of the starting polynomials).
* Opening 🤷🏻: proving the value of the $i$th element in the array is $\mathsf{data}_i$ doing polynomial math on $P_1(\square)$ is not possible. However $\Sigma$-protocols done directly on KZG may enable this kind of proof. In any case, this is not particularly well explored. 
* Other useful properties 👍: the sum of all values in a array can be computed by evaluating the polynomial at $P_1(\boxed{1})$! $\sum_{i=0}^d \mathsf{data}_i\cdot\boxed{1}^i=\sum_{i=0}^d \mathsf{data}_i$ . You can also show two arrays have the same sum (called a "sum check") by subtracting them and showing $P(\boxed{1})=0$.
* Other useful properties 👍: when all coefficients are 0, the polynomial will be the zero polynomial ($P_1(\square)=0$). Coefficients can be entire polynomials, not just integers. A common optimization in Poly-IOP systems is taking a set of equations of polynomials that should equal 0, placing each into the coefficient of a super-polynomial, and showing the super-polynomial is the zero polynomial (which can be proven overwhelmingly by showing it is 0 at a randomly selected point).

  

### Encoding 2: Evaluation Points

Create a list of points $\{x,y\}$ for the data: $\langle\{0,\mathsf{data}_0\},\{1,\mathsf{data}_1\},\{2,\mathsf{data}_2\},\{3,\mathsf{data}_3\},\{4,\mathsf{data}_4\}\rangle$ and interpolate a polynomial $P_2(\square)$ through these points. 

Properties:

* Slow (or moderate) 👎: converting a set of points into a set of coefficients is called interpolation and is $O(n^2)$ time generally. A certain optimization allows $O(n\log n)$ time by chosing $x$ coordinates with a mathematical relationship (more on this later).
* Addition 👍: two arrays can be added together (slot-by-slot) by simply adding the polynomials together
* Multiplication 👍: two arrays can be multiplied together (slot-by-slot) by simply multiplying the polynomials together
* Opening 👍: proving the value of the $i$th element in the array is $\mathsf{data}_i$ is possible with polynomial math by showing $P_2(\boxed{i})=\mathsf{data}_i$ and KZG has a precise algorithm for this. 



### Encoding 3: Roots

Create polynomial as: $P_3(\square)=(\square-\mathsf{data}_0)(\square-\mathsf{data}_1)(\square-\mathsf{data}_2)(\square-\mathsf{data}_3)(\square-\mathsf{data}_4)$

Alternatively, create a list of roots $\{x,0\}$ for the data: $\langle\{\mathsf{data}_0,0\},\{\mathsf{data}_1,0\},\{\mathsf{data}_2,0\},\{\mathsf{data}_3,0\},\{\mathsf{data}_4,0\}\rangle$ and interpolate a polynomial $P_3(\square)$ through these points. 

Properties:

* Slow 👎 (or moderate): multiplying out naively requires $O(n^2)$ time. Treating as a set of points and interpolating also requires $O(n^2)$ time (because the x-coordinates are the data, they cannot be chosen freely to optimize interpolation). Applying divide and conquer can provide $O(n \log^2 n)$.^[1]
* Addition 👎: two arrays cannot be added from adding (or otherwise manipulating) the polynomials.
* Multiplication 👎: two arrays cannot be multiplied from multiplying (or otherwise manipulating) the polynomials (but you can do a "union" operation below).
* Opening 👍: proving that a value is in the array somewhere is easy and KZG as a precise algorithm for this (opening a root is the same as opening a point, where the y-coordinate is 0). However you cannot show a value is specifically the $i$th value in the array because the polynomial loses the order of the data in the array (see next property). 
* Other useful properties 👍: the order of the data in the array does not matter. The same polynomial will be produced even if the order is changed. This is useful when the array represents a "bag" of unordered data. You can easily prove two "bags" of data are the same because the polynomials will be the same. One use-case of this is proving the output of a shuffle/permutation is the same data as the input (just in a different order).
* Other useful properties 👍: multiplying two polynomials results in a concatenation of the data in the arrays (or conjunction/union of the data in both bags). This might be useful in some protocols.

^[1]: Hat tip Pratyush Mishra.

### Decision Tree for Encoding

Basically we decide if we specifically need unordered "bags" of data. If so, encoding as roots is the only option. If not, we consider if we need to ever get the data back from the polynomial. Generally we do and encoding as evaluation points is the most common encoding technique. When do we encode the data and never want it back? Usually when (1) the coefficients are all supposed to be zero so we are just showing that property, or (2) we want back the sum of the data and not the data itself. In these cases, you can still work with evaluation point encoding but it will be faster to just do coefficient encoding.



{{< mermaid >}}
  flowchart LR;
      A[Array to Polynomial] --> B{Is the data unordered?};
      B -- Unordered --> C[Roots];
      B -- Ordered or don't care --> D{Open data from polynomial later?};
      D -- Yes --> E[Evaluation Points];
      D -- No --> F[Coefficients];
{{< /mermaid >}}

The short answer is to start with evaluation point encoding until you realize you need something different.



## Optimizing Interpolation (Roots of Unity + FFT)

Moving forward, we will assume we are using Encoding 2: Evaluation Points. In short, this means placing the elements of our array into the $y$-coordinates ($\mathsf{data}_i=P(\boxed{x_i})$) of points on the polynomial. Before commiting to $P(\square)$, we need to use interpolation to find the coefficients of the polynomial that is fitted to these points. General interpolation algorithms are $O(n^2)$ work for $n$ evaluation points but this can be reduced to $O(n\log n)$ with an optimization.

The optimization we will explore enables interpolation via the fast Fourier transform (FFT). It concerns how to choose the $x$-coordinates, which will serve as the index for accessing the data: evaluating $P(X)$ at $x_i$ will reveal $\mathsf{data}_i$. First note, $x$-coordinates are from the exponent group ($Z_q$) and the choices exceed what is feasible to use ($2^{255}$ values in bls). Any subset can be used and interpolated. The optimization is to chose them with a mathematical structure. Specifically, instead an additive sequence (e.g., $0,1,2,3,\ldots$), we use a multiplicative sequence $1,\omega,\omega\cdot\omega,\omega\cdot\omega\cdot\omega,\ldots$ or equivalently: $\omega^0,\omega^1,\omega^2,\ldots,\omega^{\kappa-1}$. Further, the sequence is closed under multiplication which means that next index after $\omega^{\kappa-1}$ wraps back to the first index: $\omega^{k-1} \cdot \omega = \omega^\kappa = \omega^0=1$ (this property is also useful in proving relationships between data in the array and its neighbouring values). 

![Roots of Unity](https://hackmd.io/_uploads/SyQdoMDxC.png)


For terminology, we say $\omega$ is a generator with multiplicative order $\kappa$ in $\mathbb{Z}_q$ (or $\omega \in \mathbb{G}_\kappa$). This implies $\omega^\kappa=1$. Rearranging, $\omega=\sqrt[\kappa]{1}$. Thus we can equivalently describe $\omega$ as a $\kappa$-th root of 1. Finally, as 1 is the unity element in $Z_q$, $\omega$ is commonly called a $\kappa$-th root of unity. 

For practical purposes, $\kappa$ represents the length of the longest array of data we can use in our protocol. Where does $\kappa$ come from? Different elements of $Z_q$ will have different multiplicative orders but every order must be a divisor of $q-1$. Thus $\kappa$ is the largest divisor of the exact value of $q$ used in an elliptic curve standard. BLS12-384 has $\kappa=2^{32}$ (for terminology, this called a $2$-adicity of $32$). In summary, we can only encode data arrays of length up to $2^{32}=4,294,967,296$.



## Zeroing Parts of an Array

Assuming an input array of size $n$: $\langle \mathsf{data}_0,\mathsf{data}_1,\ldots,\mathsf{data}_{n-1}\rangle$ and input array encoded into the polynomial. This uses "Encoding 2" from above (evaluation points) and uses "Roots of Unity + FFT" from above where $\omega\in\mathbb{G}_\kappa$ is a generator for the x-coordinates of the points.

$\bot$ is an arbitrary non-zero integer.

| Operation          | Input Array                 | Input Polynomial | Output Array                            | Output Polynomial                           |
| ------------------ | --------------------------- | ---------------- | --------------------------------------- | ------------------------------------------- |
| Zero all           | $\langle 3,1,3,3,7 \rangle$ | $P(X)$           | $\langle 0,0,0,0,0 \rangle$             | $P(X)\cdot(X^n-1)$                          |
| Zero first         | $\langle 3,1,3,3,7 \rangle$ | $P(X)$           | $\langle 0,\bot,\bot,\bot,\bot \rangle$ | $P(X)\cdot(X-\omega^0)$                     |
| Zero last          | $\langle 3,1,3,3,7 \rangle$ | $P(X)$           | $\langle \bot,\bot,\bot,\bot,0 \rangle$ | $P(X)\cdot(X-\omega^{n-1})$                 |
| Zero all but first | $\langle 3,1,3,3,7 \rangle$ | $P(X)$           | $\langle \bot,0,0,0,0 \rangle$          | $P(X)\cdot\frac{(X^n-1)}{(X-\omega^0)}$     |
| Zero all but last  | $\langle 3,1,3,3,7 \rangle$ | $P(X)$           | $\langle 0,0,0,0,\bot \rangle$          | $P(X)\cdot\frac{(X^n-1)}{(X-\omega^{n-1})}$ |

Why are these useful? Consider a [range proof](https://hackmd.io/@dabo/B1U4kx8XI) from Boneh, Fisch, Gabizon, and Williamson. At a certain point in the protocol, we reach the following:

![Screenshot 2024-04-12 at 9.11.45 PM](https://hackmd.io/_uploads/H1Xg2IvgR.png)

First note that $w$ and $\omega$ look the same but are different: $w$ are three new polynomials we are creating, while $\omega$ is the generator of $\mathbb{G}_\kappa$ we are using to pick x-coordinates for the polynomials. 

* In the first constraint, we want to prove the value of f(1) and g(1) are the same. So we subtract (g-f) which leaves a zero in the first element of the array but the rest of the array will contain other stuff. By applying "zero all but first", we can zero out the rest of the array. We now have an array that is all zero (in polynomial form, this is called a vanshing polynomial and we can prove a polynomial vanishes easily and in a batch).
* In the second constraint, we prove the last element in g is a binary bit (0 or 1) and then we apply the "zero all but last" to make an array that is all zero (and vanishing polynomial).
* In the third constraint, the first two terms of the multiplication are proving $g(X)$ has a certain form that does not need to be understood here. What is important is that there is a relationship between each integer in the array ($g(X)$) and the integer right beside it ($g(X\omega)$) in the array. When the relationship holds, the result is a zero in $w_3$. Unfortunately, there is a corner case: for the last element in the array, the "next" integer wraps back to the first integer and the relationship does not hold across this boundary. So we use "zero last" to manually zero out the last integer in the array, leaving us with a zero array (and vanishing polynomial). 





# KZG Commitments

KZG is a commitment scheme, designed specifically for polynomials. While any commitment scheme can be used to commit to a polynomial (e.g., hash the polynomial) and later reveal it, KZG has homomorphic properties that are useful for polynomials. They are as follows:

1) Addition. $\mathsf{Comm}$
2) Multiplication Check. $P_1(z)\cdot P_2(z) - P_3(z)=0$ 
3) Evaluation Check.



* g^P(X) -> g^(c_0+c_1X+c_2X^2)
* Evaluate at tau to commit
* Trusted setup (Powers of Tau) -> $g,g^{\tau}, g^{\tau^2}$



* Array: [4, 6, 7, 2, 346, 24...]
* Array -> Polynomial -> Polynomial Commitment (Poly-IOP; polynomial interactive oracle proof)
* Array -> Polynomial -> How?
  * Evaluation points -> [{1,4},{2,6},{3,7}...] -> Lagrange / FFT interpolation
  * Roots -> [{4,0},{6,0},{7,0},..] -> Interpolation is sufficient, not necessary -> Expansion -> (X-4)(X-6)(X-7) 
  * Coefficients -> 4+6X+7X^2 -> Nothing 



* KZG: $g^{c_0+c_1*\tau+c_2*\tau^2+\ldots}$
* Comm(P1)*Comm(P2) = Comm (P1+P2)
* Comm(P1)^P2=Comm(P1*P2)
* Comm(P1)^a=Comm(a*P1)
* Open Root (z=P1(0))
  * Open a root at 7
  * Q=(X-4)(X-6)(X-7) / (X-7) -> evenly divide iff 7 is a root
  * Q(X-7) = P -> check with pairing
* Open point (z=P1(x))
  * Subtract off z and now x is a root, see above
* Batch opening many points:
  * Open many points on same polynomial P
  * Open the same point on many polynomials 
  * Open many points on many polynomials -> not sure?



$P_1(X)=c_0+c_1*X+\ldots$



$P_3(\square)=(\square-\mathsf{data}_0)(\square-\mathsf{data}_1)$

$P_3(\square)=((\square-\mathsf{data}_2)(\square-\mathsf{data}_3)(\square-\mathsf{data}_4)$

$P_3(\square)=(\square-\mathsf{data}_0)(\square-\mathsf{data}_1)(\square-\mathsf{data}_2)(\square-\mathsf{data}_3)(\square-\mathsf{data}_4)$
