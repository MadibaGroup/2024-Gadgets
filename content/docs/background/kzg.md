# Polynomial Commitment Schemes (PCS)

Say that Carol sends a polynomial to Alice and the same polynomial to Bob. Later Alice and Bob are sitting together and want to make sure their polynomials, which they got from Carol, are actually the same. How do you compare polynomials? There are many sufficient comparisons, but here are two ways to get started:

1. Ensure the degree $d$ is the same and each coefficient is the same,
2. Ensure the degree $d$ is the same and check at $d+1$ points.

Say Alice and Bob go with the second method for their polynomials from Carol, which are both degree 1000. They start checking points and after checking 500 points, the polynomials are the same so far. How confident are they that they have the same polynomial? They are not 100% confident until they check all $d+1$ points but should they be more than 0% confident?

Imagine Carol is trying to trick them with different polynomials. If Carol knows Alice and Bob will check $P(0),P(1),P(2),\ldots,P(d+1)$, she can make sure the polynomials are the same at the first set of points, hoping they give up early (so Alice and Bob should be 0% confident after checking 500 points). 

However if Alice and Bob choose random points to compare at, then Carol has to get "lucky" and hope the polynomials happen to be the same at each of those points. The nice thing is we can quantify what "lucky" means for Carol. Two polynomials of degree $d$ can only match at $d$ points (even if the polynomials are very similar to each, say differing only by one coefficient). Every time Alice and Bob check a random point, the probability that Carol gets lucky and does not get caught presenting different polynomials is $\frac{d}{q}$ where $q$ is the number of points on the polynomial. In a cryptographic setting, $q$ will be the integers $[0,\ldots,q-1]$ for large prime $q$. If $q$ is 256-bits and the polynomial is degree 1000, then the probability Carol can cheat is $\frac{1000}{2^{256}}$ which is negligibly small. So after checking one point, Alice and Bob are not confident with probability $1$. But they are with probability $1-\frac{1000}{2^{256}}$ which is close enough to 1 for practical purposes:

$0.99999999999999999999999999999999999999999999999999999999999999999999999999136383\ldots$

So we can add a third probabilistic method for Alice and Bob to use in checking their polynomials:

3. Ensure the degree $d$ is the same and check at $1$ random point.

This is close enough to Method 2 for practical purposes and is materially less work for Alice and Bob. In mathematics, this result is based on the DeMillo–Lipton–Schwartz–Zippel lemma, more commonly known as the Schwartz–Zippel lemma. We will use this idea in building a commitment scheme for polynomials. 

## KZG Polynomial Commitment

Recall the Pedersen commitment scheme, which for message $m$ and randomness $r$ produces commitment value $K_m=\mathsf{Commit}(m,r)=g^mh^r$. The commitment scheme is *hiding* in that examining $K_m$ does not reveal the message $m$ or any information about what $m$ might be. It is also *binding* which means the person who creates and circulates $K_m$ can later show how $m$ was used to construct $K_m$ but cannot show that for a different value $m'\neq m$. This comes under the important caveat that $g$ and $h$ are random group elements and the committer does not know there discrete logarithm. 





## Definitions

**Definition 1 (Polynomial Commitment Scheme).** A polynomial commitment scheme (PCS) is an interactive proof system that enables $\mathcal{P}$ to convince $\mathcal{V}$ that he knows a polynomial, without revealing the polynomial directly. $\mathcal{P}$ and $\mathcal{V}$ run the protocol in three moves: **gen**, **com**, and **open**. [Plonk]

