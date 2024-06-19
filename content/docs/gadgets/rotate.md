# Rotate

Assume $\mathsf{Arr}$ is an array of data of size $n$. It is encoded as the y-coordinates into univariant polynomials where the x-coordinates (called the domain $\mathcal{H}_\kappa$) are chosen as the multiplicative group of order $\kappa$ with generator $\omega\in\mathbb{G}_\kappa$ (see [Background](../background/poly-iop.md) for more). In short, $\omega^0$ is the first element and $\omega^{\kappa-1}$ is the last element of $\mathcal{H}_\kappa$. We call our polynomial $\mathsf{Poly}_\mathsf{Arr}(X)$. The goal is to construct an output polynomial $\mathsf{Poly_\mathsf{Arr_\alpha}}$ where all the elements in $\mathsf{Arr}$ have been rotated by $\alpha$ positions; i.e. $\mathsf{Arr}[i] \larr \mathsf{Arr}[i + \alpha]$. Here, the index is defined mod $n$.

Written in terms the polynomials, the goal is that $\mathsf{Poly_\mathsf{Arr_\alpha}}(\omega^i) = \mathsf{Poly_\mathsf{Arr}}(\omega^i\cdot\omega^\alpha)$ for all $i \in [0, n-1]$. To this end, we define $\mathsf{Poly_\mathsf{Arr_\alpha}}(X) = \mathsf{Poly_\mathsf{Arr}}(X)\cdot\omega^\alpha$ and demonstrate below why this satisfies the requirements of the output polynomials.

Consider $\mathsf{Poly_\mathsf{Arr}}(X) = c_{n-1}X^{n-1} + \dots + c_2X^2 + c_1X + c_0$. Then: 

$\mathsf{Poly_\mathsf{Arr_\alpha}}(X)$​ 

$= \mathsf{Poly_Arr}(X) \cdot \omega^\alpha$​

$= (c_{n-1}X^{n-1} + \dots + c_2X^2 + c_1X + c_0)(\omega^\alpha) $​

$= c_{n-1}X^{n-1}\cdot\omega^\alpha + \dots + c_2X^2\cdot\omega^\alpha + c_1X\cdot\omega^\alpha + c_0\cdot\omega^\alpha$

$= \mathsf{Poly_\mathsf{Arr}}(X\cdot\omega^\alpha)$

In particular, $\mathsf{Poly_{Arr_\alpha}}(\omega^i) = \mathsf{Poly_\mathsf{Arr}}(\omega^i\cdot\omega^\alpha)$ for $i \in [0, n-1]$. We also note that $\omega^n = \omega^0$, since $\omega$ is of order $\kappa$. Thus the exponent of $\omega$ is defined mod $n$, like the indexing of $\mathsf{Arr}$. This means that $\omega^i\cdot\omega^\alpha$ wraps around to have the rotation defined as it should be for $i + \alpha \gt n-1$.
