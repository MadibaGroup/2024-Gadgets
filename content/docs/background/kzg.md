---
title: PCS
type: docs
---

# Polynomial Commitment Schemes (PCS)

## Comparing Polynomials

Say that Carol sends a polynomial to Alice and the same polynomial to Bob. Later Alice and Bob are sitting together and want to make sure their polynomials, which they got from Carol, are actually the same. How do you compare polynomials? There are many sufficient comparisons, but here are two ways to get started:

1. Ensure the degree $d$ is the same and each coefficient is the same,
2. Ensure the degree $d$ is the same and check at $d+1$ unique points.

Say Alice and Bob go with the second method for their polynomials from Carol, which are both degree 1000. They start checking points and after checking 500 points, the polynomials are the same so far. How confident are they that they have the same polynomial? They are not 100% confident until they check all $d+1$ points but should they be more than 0% confident?

Imagine Carol is trying to trick them with different polynomials. If Carol knows Alice and Bob will check $P(0),P(1),P(2),\ldots,P(d+1)$, she can make sure the polynomials are the same at the first set of points, hoping they give up early. So Alice and Bob should be 0% confident after checking 500 points if they know Carol is malicious. 

However if Alice and Bob choose random points to compare at, then Carol has to get "lucky" and hope the polynomials happen to be the same at each of the random points. The nice thing is we can quantify what "lucky" means for Carol. Two polynomials of degree $d$ can only match at $d$ points (even if the polynomials are very similar to each, say differing only by one coefficient). Every time Alice and Bob check a random point, the probability that Carol gets lucky and does not get caught presenting different polynomials is $\frac{d}{q}$ where $q$ is the number of points on the polynomial (all values in $\mathbb{Z}_q$). In a cryptographic setting, $q$ will be a large prime. If $q$ is 256-bits and the polynomial is degree 1000, then the probability Carol can get away with cheating after the first check is $\frac{1000}{2^{256}}$, which is already negligibly small. So after checking just one random point, Alice and Bob are pretty confident the polynomials are the same if the point matches. Specifically, probability $1-\frac{1000}{2^{256}}$ which is close enough to 1 for practical purposes:

$0.99999999999999999999999999999999999999999999999999999999999999999999999999136383\ldots$

So we can add a third probabilistic method for Alice and Bob to use in checking their polynomials:

3. Ensure the degree $d$ is the same and check at $1$ random point.

Compared to Method 2, it is materially less work for Alice and Bob. In mathematics, this result is commonly known as the Schwartz–Zippel[^1] lemma. We will use this idea in building a commitment scheme for polynomials. 

## A Wishlist for Polynomial Commitments

Recall the Pedersen[^2] commitment scheme, which for message $m$ and randomness $r$ produces commitment value $K_m=\mathsf{Commit}(m,r)=g^mh^r$. The commitment scheme is *hiding* which means that examining $K_m$ does not reveal the message $m$ or any information about what $m$ might be. It is also *binding* which means the person who creates and circulates $K_m$ can later show how $m$ was used to construct $K_m$ but cannot show that for a different value $m'\neq m$. This comes under the important caveat that $g$ and $h$ are random group elements and the committer does not know the discrete logarithm between them.

A second property that Pedersen commitments exhibit is an *additive homomorphism*, where $K_{m_1}\cdot K_{m_2}=\mathsf{Commit}(m_1+m_2,r_1+r_2)$. This is sometimes useful for protocols requiring addition. 

A final property to be considered with commitments is how many $m$ values can be committed to and how many $K_m$ values can be produced. Specifically, are they exactly equal, which would make the commitment function *one-to-one*. Alternatively, it might be *one-to-many* or *many-to-one*. Consider a few variants of Pedersen commitments:

* Feldman commitment: $\mathsf{Commit}(m)=g^m$ is one-to-one (but not hiding as you can check if your guess at $m$ is right or not). $m\in\mathbb{Z}_q$ restricted to exponent group. 
* Hashed Pedersen commitment: $\mathsf{Commit}(m,r)=g^{\mathcal{H}(m)}h^r$ is many-to-one for collision-resistant hash function $\mathcal{H}$. $m\in\{0,1\}^*$ can be any bit-string of any length.
* Pedersen multi-commitment: $\mathsf{Commit}(m_1,m_2,m_3,r)=g_1^{m_1}g_2^{m_2}g_3^{m_3}h^r$ is many-to-one. $m_i\in\mathbb{Z}_q$ restricted to exponent group. 
* Elgamal: $\mathsf{Commit}(m,r)=\{g^r,g^{m}h^r\}$ is one-to-many. $m\in\mathbb{Z}_q$ restricted to exponent group. 

Next, let us consider committing to polynomials. We can use any of the above schemes to commit to a complete description of a polynomial, where a complete description might be its coefficients, all of its roots, or a sufficient set of points on the polynomial. However we can also consider making a special purpose commitment function for polynomials that might allow us some useful things for dealing with polynomials. A wishlist for a polynomial scheme might look as follows:

* Binding: necessary for commitments.
* (Optionally) hiding: necessary for using the commitment for zk-SNARKs but not necessary for just SNARKs.
* Useful homomorphisms: 
  * Polynomial addition
  * Polynomial multiplication
* Selective opening: demonstrating a single chosen point on a polynomial without revealing the rest of the polynomial.
* Many-to-one (succinct): having $K_{P(X)}$ be constant-size regardless of how "big" (the degree) polynomial $P(X)$ is.

The Kate-Zaverucha-Goldberg (KZG) polynomial commitment scheme gives us most of the above properties. Polynomial multiplication has a little red tape around it but the rest of the properties follow directly from KZG. KZG is only for univariant polynomials (which is all we will use in Plonkbook) however many research papers have shown how to adapt it to multivariant polynomials. 

## KZG Commitments

### Starting Point

KZG commitments will commit to a univariate polynomial assuming it is in coefficient form. That is:
$$
P(\square)=c_0+c_1\cdot\square+c_2\cdot\square^2+c_3\cdot\square^3+c_4\cdot\square^4=\sum_{i=0}^d c_i\cdot\square^i
$$

The rough idea is that a commitment will be $K_{P(\square)}=\mathsf{Commit}(P(\square), r)=g^{P(\square)}h^r$. For simplicity, we will describe the simpler version that is not hiding: $K_{P(\square)}=\mathsf{Commit}(P(\square))=g^{P(\square)}$.

What does it mean to commit to $P(\square)$ without specifying what $\square$ is? Recall that $\square$ can take on any value (in $\mathbb{Z}_q$). Probably the most intuitive idea is to use the multi-commitment (above) to commit to the coefficients: $\mathsf{Commit}(c_0,c_1,c_2,c_3)=g_0^{c_0}g_1^{c_1}g_2^{c_2}g_3^{c_3}$. This hits a few items on our wish list: it is binding, could be hiding (with $h^r$), is additively homomorphic (sum of polynomials is the sum of their coefficients), and is succinct.

What about selective opening? The committer (prover) knows that $y_\alpha=P(\alpha)$ for some value $\alpha$ and wants to prove it. To work toward computing $P(\alpha)=c_0+c_1\cdot\alpha+c_2\cdot\alpha^2+c_3\cdot\alpha^3$, the prover can create $\langle \alpha, \alpha^2, \alpha^3\rangle$ but can the prover get them into the "right spot" in the commitment: $g_0^{c_0}g_1^{c_1\cdot\alpha}g_2^{c_2\alpha^2}g_3^{c_3\alpha^3}$? It cannot do it with exponentiation as operations like $(g_0^{c_0}g_1^{c_1}g_2^{c_2}g_3^{c_3})^\alpha$ will distribute $\alpha$ to each term: $g_0^{c_0\cdot\alpha}g_1^{c_1\cdot\alpha}g_2^{c_2\cdot\alpha}g_3^{c_3\cdot\alpha}$. Even if it got them into the right spot, the prover would have to prove they used the right values. Finally, the prover cannot "add" the terms up: $g_0^{c_0}g_1^{c_1\cdot\alpha}g_2^{c_2\alpha^2}g_3^{c_3\alpha^3}\rightarrow g_0^{c_0+c_1\cdot\alpha+c_2\alpha^2+c_3\alpha^3}$ without knowing the discrete logarithm between $g_0, g_1, g_2, g_3$ (if it did, it would break the binding property) plus it would have to get rid of the discrete logarithm terms. Anyways, there are three or four roadblocks to adding selective opening to such a commitment.

KZG takes a different approach. To commit to a polynomial, the idea is to commit to its evaluation at a set of points rather than committing to the coefficients. How many points do we need to commit to? Recall the story about Alice and Bob comparing the polynomials they received from Carol. We concluded that statistically it is sufficient for Alice and Bob to check at $1$ random point. 

The idea of KZG is to commit to the evaluation of the polynomial at one random point $\tau$ and have this commitment represent a commitment to the entire polynomial. The important caveat is that no one knows what $\tau$ is (it is a secret). How do we pull this off? How can the prover figure out the evaluation of their polynomial at $\tau$ without knowing what $\tau$ is? And finally, if they can figure out what $P(\tau)$ is, can they not just reverse engineer what $\tau$ is (for example, by committing to $y=P(\tau)=\tau$)?

KZG uses a trusted setup to generate a commitment to $\tau$. It then uses homomorphic operations to let a prover compute a commitment to $P(\tau)$ using the commitment to $\tau$ without learning what $\tau$ is or even what $P(\tau)$ is. Committing to $P(\tau)$ is considered as good as committing to the entire polynomial! We will see these details next.

### Setup

Someone trusted chooses a random value (from $\mathbb{Z}_q$) to use as  $\tau$. They will also generate $d$ powers of $\tau$ which will be needed to commit to polynomials up to degree $d$. They will commit to each power of $\tau$ as follows. The output is called a structured random string (SRS) and will be available for the prover to use and the verifier to use:

$$
\langle g^{(\tau^0)}, g^{(\tau^1)}, g^{(\tau^2)}, g^{(\tau^3)}, \ldots, g^{(\tau^d)} \rangle \equiv SRS
$$

What if the person generating the powers of $\tau$ is not trustworthy? What goes wrong? There are two ways the trusted party can cheat: (1) tell everyone the secret value $\tau$ (or use it for itself), and (2) generate the SRS with the wrong structure (the values are not successive powers of $\tau$). We can address both of these issues. To address (1), we can let many parties generate an SRS and combine them into a single SRS. As long as one party deletes $\tau$ without looking at it, the entire SRS is secure. Many blockchain projects have done this already with thousands of contributors. We can also address (2) through special purpose zero knowledge proofs that demonstrate each contributor's SRS has the correct format without revealing $\tau$. We will not cover these details but you can learn more from [this article (a16z crypto research)](https://a16zcrypto.com/posts/article/on-chain-trusted-setup-ceremony/).

### Commitment

Using the coefficients of the polynomial, the prover can use the SRS to create a commitment to the polynomial evaluated at $\tau$ as follows:
$$
\begin{align}
K_{P(\tau)}&=\mathsf{Commit}(P(\tau))\\
&= (g)^{c_0} (g^{\tau})^{c_1} (g^{\tau^2})^{c_2} (g^{\tau^3})^{c_3} \ldots \\
&= g^{c_0 + c_1\tau + c_2\tau^2 + c_3\tau^3 + \ldots}\\
&= g^{P(\tau)}
\end{align}
$$
At the end of the commitment operation, the prover does not know either $\tau$ nor $P(\tau)$ and would have to solve a discrete logarithm to learn them. The degree of the polynomial $P(\square)$ has no impact on the size of $K_{P(\tau)}$ so it is succinct. However the SRS has to be long enough to have $\tau^d$ for committing to degree $d$ polynomials. 

### Commitment Operations

#### Open

To completely open the polynomial, the prover will just send the coefficients to the verifier $\langle c_0,c_1,c_2,c_3,\ldots \rangle$ and the verifier will recompute $(g)^{c_0} (g^{\tau})^{c_1} (g^{\tau^2})^{c_2} (g^{\tau^3})^{c_3} \ldots$ and check it matches $K_{P(\tau)}$. However this is not very interesting by itself as the prover could have done something simpler like hashing the coefficients. This is why KZG offers additional features tailored to working with polynomials.

#### Addition

KZG commitments are homomorphic with respect to the polynomial addition: $K_{P_1(\tau)}\cdot K_{P_2(\tau)}=\mathsf{Commit}(P_1(\tau)+P_2(\tau))$. As is usually the case, scalar multiplication (multiplication by a disclosed integer) can also be performed.

#### Multiplication

KZG commitments are not exactly homomorphic with respect to polynomial multiplication but we can get something close. The subtle difference is as follows. For addition, anyone who sees $K_{P_1(\tau)}$ and $K_{P_2(\tau)}$ can compute $K_{P_1(\tau)+P_2(\tau)}$ directly without involving anyone else. This is not possible with multiplication. However the prover can assert the value for $K_{P_1(\tau)\cdot P_2(\tau)}$ and can convince the verifier it is correct, given $K_{P_1(\tau)}$ and $K_{P_2(\tau)}$. The rough idea is to use the bilinear pairing to show:
$$
\begin{align}
e(K_{P_1(\tau)},K_{P_2(\tau)})&\stackrel{?}{=}e(K_{P_1(\tau)+P_2(\tau)},g)\\
 e(g^{P_1(\tau)},g^{P_2(\tau)})&=e(g^{P_1(\tau)\cdot P_2(\tau)},g)
 \\&=e(g,g)^{P_1(\tau)\cdot P_2(\tau)}
 \end{align}
$$
There is some red tape with this as the pairing might not be symmetric ($g$ and $h$ are in different groups for $e(g,h)$) and other subtle details. We will show a different approach to multiplications with our $\texttt{mult}$ gadget. 

#### Selective Open: Root

The most useful property of KZG (and any polynomial commitment scheme) is being able to open the committed polynomial to certain points. We start with a simple case, the prover wants to show the polynomial is zero at a certain value $P(r)=0$. Recall that $r$ is called a root of the polynomial. Also recall that a set of roots $r_0, r_1, r_2 \ldots$ can be turned into coefficient form using multiplication:
$$
P(\square)=(\square-r_0)(\square-r_1)(\square-r_2)(\square-r_3)\ldots
$$
If the prover is correct that $r$ is a root of the polynomial, then it is the case that the term $(\square-r)$ will evenly divide the polynomial without any remainder. It is also the case that if $r$ is not a root of the polynomial, $P(\square)/(\square-r)$ will not be an even division and will result in some remainder. The prover will use this fact to prove $r$ is a root.

The prover will compute the quotient polynomial $Q(\square)=P(\square)/(\square-r)$ and provide a commitment to it $K_{Q(\tau)}$ with KZG and give the commitment to the verifier. Knowing the value $r$, if the verifier can check if $P(\square)=Q(\square)(\square -r)$, it will be convinced that $(r-\square)$ evenly divides $P(\square)$ (since $Q$ is a single polynomial) and thus $r$ is a root. 

The verifier has $r$ and $K_{P(\tau)}$ and $K_{Q(\tau)}$. The verifier can compute a commitment to $(\square-r)$ at $\tau$ by just treating it as the polynomial   $V(\square)=(\square-r)=-r+1\cdot\square$ and using KZG to produce $K_{V(\tau)}$. The verifier can then check:
$$
\begin{align}
e(K_{P(\tau)},g)&\stackrel{?}{=}e(K_{Q(\tau)},K_{V(\tau)})\\
 e(g^{P(\tau)},g)&=e(g^{Q(\tau)},g^{V(\tau)})
 \\e(g,g)^{P(\tau)}&=e(g,g)^{Q(\tau)\cdot V(\tau)}
 \end{align}
$$
Notice that the degree of the polynomial $P(\square)$ (and thus $Q(\square)$) has no impact on the size of the proof given to the verifier or the work that the verifier has to do. Even if the degree of $P(\square)$ is billions, the proof is the same size and time for the verifier. In a sense, this is our first special purpose SNARK and one we will leverage into making SNARKs for all the gadgets in Plonkbook.

#### Selective Open: Multiple Roots

If the prover wants to open at multiple roots, it runs the exact same protocol but has the verifier set $V(\square)=(\square-r_0)\cdot(\square-r_1)\cdot(\square-r_2)\ldots$ for all the roots it wants to prove. Importantly, this does not add any complexity for the verifier once $K_{V(\tau)}$ has been created.

#### Selective Open: Arbitrary Point

Most of the time, the prover wants to prove the polynomial passes through an arbitrary point $\{x,y\}=\{x,P(x)\}$ where $y$ is some integer that is not zero (and thus $x$ is not a root). Again, this is very easy to prove once we have a protocol for proving roots. The intuition is as follows: if and only if $P(\square)$ has value $y$ at point $x$, then subtracting $y$ from $P(\square)$ will create a new polynomial $\tilde{P}(\square)=P(\square)-y$ that is zero at point $x$, making $x$ a root. (Visually you can imagine a polynomial with height $y$ at a point $x$, and subtracting $y$ shifts the whole polynomial down $y$ units, moving that point down to the x-axis.) The verifier can construct $K_{\tilde{P}(\tau)}$ from $K_{P(\tau)}$ using the additive homomorphic property:  $K_{\tilde{P}(\tau)}=K_{P(tau)\cdot} g^{-y}$. Then the prover shows $K_{\tilde{P}(\tau)}$ has a root at point $x$ using the protocol above for proving roots.

#### Selective Open: Batch of Points

If the cost of opening a set of points is roughly the same as the cost of opening a single point, it is called a batch opening. Several variants might be desired:

* Batch open multiple points on the same polynomial: direct from KZG, shown below.
* Batch open the same point on multiple polynomials: direct from KZG, not shown. 
* Batch open multiple points on multiple polynomials: not directly possible with KZG but possible with variants.

To batch open multiple points on the same polynomial $P(\square)$, the prover asserts the full set of points to the verifier. The verifier will interpolate a polynomial through these points $R(\square)$ and compute its commitment $K_{R(\tau)}$. Recall that when proving a single point $\{x,y\}$, it would compute $\tilde{P}(\square)=P(\square)-y$. This does not work for multiple points because the amount to "slide down" the polynomial differs. However $R(\square)$ has the right amount at every value of $x$ being opened. So the verifier computes $K_{\tilde{P}(\tau)}=K_{P(\tau)} \cdot (K_{R(\tau)})^{-1}$ where $\tilde{P}(\square)=P(\square)-R(\square)$ using the additively homomorphic property, and now all the points are roots, so the prover uses the "selective open: multiple roots" protocol above on $\tilde{P}(\square)$. 

## Footnotes

[^1]: The result was actually pointed out earlier by DeMillo and Lipton, and thus is occasionally referred to as the DeMillo–Lipton–Schwartz–Zippel lemma
[^2]: The commitment was popularized by Pedersen but Pedersen attributes it to Bos and Chaum in his paper. It appeared even earlier as a bit commitment scheme in a paper by Chaum, Damgard, and van de Graaf.
