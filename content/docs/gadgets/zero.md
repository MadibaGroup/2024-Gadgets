## Zeroing Parts of an Array

Assuming an input array of size $n$: $\langle \mathsf{data}_0,\mathsf{data}_1,\ldots,\mathsf{data}_{n-1}\rangle$ and input array encoded into the polynomial. This uses "Encoding 2" from above (evaluation points) and uses "Roots of Unity + FFT" from above where $\omega\in\mathbb{G}_\kappa$ is a generator for the x-coordinates of the points.

$\bot$ is an arbitrary non-zero integer.

| Operation          | Input Array                 | Input Polynomial | Output Array                            | Output Polynomial                                     |
| ------------------ | --------------------------- | ---------------- | --------------------------------------- | ----------------------------------------------------- |
| Zero all           | $\langle 3,1,3,3,7 \rangle$ | $P(X)$           | $\langle 0,0,0,0,0 \rangle$             | $P(X)\cdot(X^\kappa-1)$                               |
| Zero first         | $\langle 3,1,3,3,7 \rangle$ | $P(X)$           | $\langle 0,\bot,\bot,\bot,\bot \rangle$ | $P(X)\cdot(X-\omega^0)$                               |
| Zero last          | $\langle 3,1,3,3,7 \rangle$ | $P(X)$           | $\langle \bot,\bot,\bot,\bot,0 \rangle$ | $P(X)\cdot(X-\omega^{\kappa-1})$                      |
| Zero all but first | $\langle 3,1,3,3,7 \rangle$ | $P(X)$           | $\langle \bot,0,0,0,0 \rangle$          | $P(X)\cdot\frac{(X^\kappa-1)}{(X-\omega^0)}$          |
| Zero all but last  | $\langle 3,1,3,3,7 \rangle$ | $P(X)$           | $\langle 0,0,0,0,\bot \rangle$          | $P(X)\cdot\frac{(X^\kappa-1)}{(X-\omega^{\kappa-1})}$ |

Why are these useful? Consider a [range proof](https://hackmd.io/@dabo/B1U4kx8XI) from Boneh, Fisch, Gabizon, and Williamson. At a certain point in the protocol, we reach the following:

![Screenshot 2024-04-12 at 9.11.45 PM](https://hackmd.io/_uploads/H1Xg2IvgR.png)

First note that $w$ and $\omega$ look the same but are different: $w$ are three new polynomials we are creating, while $\omega$ is the generator of $\mathbb{G}_\kappa$ we are using to pick x-coordinates for the polynomials. 

* In the first constraint, we want to prove the value of f(1) and g(1) are the same. So we subtract (g-f) which leaves a zero in the first element of the array but the rest of the array will contain other stuff. By applying "zero all but first", we can zero out the rest of the array. We now have an array that is all zero (in polynomial form, this is called a vanshing polynomial and we can prove a polynomial vanishes easily and in a batch).
* In the second constraint, we prove the last element in g is a binary bit (0 or 1) and then we apply the "zero all but last" to make an array that is all zero (and vanishing polynomial).
* In the third constraint, the first two terms of the multiplication are proving $g(X)$ has a certain form that does not need to be understood here. What is important is that there is a relationship between each integer in the array ($g(X)$) and the integer right beside it ($g(X\omega)$) in the array. When the relationship holds, the result is a zero in $w_3$. Unfortunately, there is a corner case: for the last element in the array, the "next" integer wraps back to the first integer and the relationship does not hold across this boundary. So we use "zero last" to manually zero out the last integer in the array, leaving us with a zero array (and vanishing polynomial). 
