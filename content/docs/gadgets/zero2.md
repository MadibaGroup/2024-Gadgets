# Zero (Type 2)

## Recap of types


| Type              | Description                                          | Recap                                                        | This |
| ----------------- | ---------------------------------------------------- | ------------------------------------------------------------ | ---- |
| [zero1](../zero1) | $\mathsf{Arr}[i]\leftarrow0$                         | Zero-out elements of an array $\mathsf{Arr}$ such as: all, first, last, all-but-first, all-but-last. |      |
| [zero2](../zero2) | $\mathsf{Arr}[i]\leftarrow0$ iff $\mathsf{Sel}[i]=0$ | Zero-out elements of an array $\mathsf{Arr}$ according to a binary array $\mathsf{Sel}$. | ✅    |

 

# Zeroing Parts of an Array (2)

Assume both $\mathsf{Arr}$ (an array of data) and $\mathsf{Sel}$ (a binary array) are of size $n$. They are encoded as the y-coordinates into univariant polynomials where the x-coordinates (called the domain $\mathcal{H}_\kappa$) are chosen as the multiplicative group of order $\kappa$ with generator $\omega\in\mathbb{G}_\kappa$ (see [Background](../../background/poly-iop) for more). In short, $\omega^0$ is the first element and $\omega^{\kappa-1}$ is the last element of $\mathcal{H}_\kappa$. We call our polynomials $\mathsf{Poly}_\mathsf{Arr}(X)$ and $\mathsf{Poly}_\mathsf{Sel}(X)$. The goal is to construct an output polynomial where all the elements in $\mathsf{Arr}$ that share an index with a zero in $\mathsf{Sel}$ are zeroed (and non-zero elements in $\mathsf{Arr}$ that share an index with a one in $\mathsf{Sel}$ are kept non-zero).

Note that $\mathsf{Poly}_\mathsf{Sel}(X)$ has a root at every point in $\mathcal{H}_\kappa$ that corresponds to a zero entry in $\mathsf{Sel}$, since it has a $y$ value of zero at each of these values of $x$. Denote this set of roots $R = [r_0, r_1, \dots, r_{m-1}]$, where $m$ is the number of zero entries in $\mathsf{Sel}$. Denote also $S = [s_0, s_1, \dots, s_{l-1}]$ the (possibly empty) set of remaining roots of $\mathsf{Poly}_\mathsf{Sel}(X)$, for some $l \in \mathbb{N}$. Then written in factored form we have $\mathsf{Poly}_\mathsf{Sel}(X) = \prod^{i \lt m}_{i = 0} (X - r_i) \cdot \prod^{i \lt l}_{i=0}(X-s_i) \cdot R(X)$ for some $R(X) = \frac{\mathsf{Poly}_\mathsf{Sel}(X)}{\prod^{i \lt m}_{i = 0} (X - r_i) \cdot \prod^{i \lt l}_{i=0}(X-s_i)}$. Note that $R(X)$ is indeed a polynomials, and that it has no roots over the field.

Now, consider $\mathsf{Poly}_\mathsf{Arr}(X) \cdot \mathsf{Poly}_\mathsf{Sel}(X) = \mathsf{Poly}_\mathsf{Arr}(X) \cdot \prod^{i \lt m}_{i = 0} (X - r_i) \cdot \prod^{i \lt l}_{i=0}(X-s_i) \cdot R(X)$. First, note it has roots at each $r \in R$, thus this new polynomial has zeroed out all the elements of $\mathsf{Arr}$ corresponding to a zero entries in $\mathsf{Sel}$. Further, the only other roots it has are from $S$, or are roots that were already part of $\mathsf{Poly}_\mathsf{Arr}(X)$. Since no value $s \in S$ is also in $\mathcal{H}_\kappa$, all non-zero entries in $\mathsf{Arr}$ are left non-zero.Thus, $\mathsf{Poly}_\mathsf{Arr}(X) \cdot \mathsf{Poly}_\mathsf{Sel}(X)$ is our desired output polynomial.
