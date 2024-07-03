# Red Tape

Generally speaking, the gadgets are quite flexible and work well as defined. However there are some subtle issues to pay attention to when using them.

## Max array size

Arrays can be very large but encoding points into a multiplicative domain (using roots of unity) means a root of unity must exist that is the same or larger than the array size. A domain of size $\kappa$ exists only if $\kappa$ divides $q-1$, where $q$ is the modulus that applies to exponents in the elliptic curve. For BLS12-384, the largest $\kappa=2^{32}$. In summary, we can only encode data arrays of length up to $2^{32}=4,294,967,296$. We can use smaller domains. If $2^{32}$ divides $q-1$, so does $2^{31}$ or $2^{30}$ or any lesser power of 2. For terminology, this called a $2$-adicity of $32$.

In a sense, $2^{32}$ is an artificial limit since we do not need to encode data into multiplicative domains. We can use any domain, we just need spending more time interpolating with Lagrange. But since we only consider doing this when the domain gets really large, quadratic-time algorithms might be prohibitively expensive for numbers this big.

## Array expansion

Some gadgets operate on arrays of size $n$ but create temporary arrays that are larger than $n$ in order to prove the operation is done correctly. In particular, two gadgets are guilty of this:

*  In $\texttt{lookup2}$, there are a few methods. In the method based on plookup, the temporary arrays need length $n_1$ + $n_2$ where $n_1$ is the length of the array and $n_2$ is the length of the lookup table. 
*  In $\mathtt{range}$, each element is decomposed into binary. So if integers are shown to be the range $\{0,1\}^b$, an array of length $n$ needs a temporary array of size $n\cdot2^b$. This can quickly lead to an array length exceeding the maximum root of unity. See the $\texttt{range}$ gadget for discussion of other options.

## Padding arrays

If the domain of the array is larger than what is needed, we add some dummy elements to the end of the array to pad it. These can be zeroed out at any time with $\texttt{zero1}$ or $\texttt{zero2}$. However if we choose padding mindfully, we can avoid this extra work. For example, in $\texttt{add2}$, padding with $0$ does not impact the protocol, while padding with $1$ does not impact $\texttt{mult2}$. 

## Boundary conditions

Many gadgets involve comparisons between neighbouring values in the same array (or across two arrays). This comparison works until you hit the last element of the array and there is no "next" element in the array. In practice, there is always a "next" element but attention needs to be paid to what it is. Using a multiplicative domain, the next element will be a dummy element if the array is smaller than the domain, or it will "wrap" back to the first element of the array if it is full. Many gadgets will just let the constraint end up with some arbitrary value in the last element, and then manually zero it out with $\texttt{zero2}$.

## Trusted setup

KZG polynomial commitments require a trusted setup, which implicitly impacts the maximum length an array can take on. Interpolating an array of length $n$ into a univariate polynomial will output a polynomial of degree $n-1$. The trusted setup needs to have an SRS with $n-1$ elements. 

In terms of trust, the trusted setup can be run with any number of individuals participating, who each must prover that their participation did not corrupt the setup in any way. As long as 1 participant is honest and does not remember their random numbers, the setup achieves its security properties. 



## Additional reading

* [Article (Thaler)](https://a16zcrypto.com/posts/article/17-misconceptions-about-snarks/): includes points comparing univariate Poly-IOP to other zk-SNARK models
* [Paper (Nikolaenko)](https://eprint.iacr.org/2022/1592): trusted setup ceremony

 