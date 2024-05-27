# Multiplication (within polynomial)

## Array-level

Given array $\mathsf{Arr}_1$ of length $\ell$ and integer $\mathsf{Prod}_1$, demonstrate that $\mathsf{Prod}=\prod_{i=0}^{\ell-1}\mathsf{Arr}_1[i]$.

* Build accumulator array $\mathsf{Acc}_1$ from $\mathsf{Arr}_1$ as follows:
  * $\mathsf{Acc}_1[\ell-1]=\mathsf{Arr}_1[\ell-1]$
  * $\mathsf{Acc}_1[i]=\mathsf{Arr}_1[i]\cdot \mathsf{Acc}_1[i+1]$​
* If built correctly, $\mathsf{Acc}_1[0]=\mathsf{Prod}_1$



Example:

* Checked input:
  * $\mathsf{Prod}_1=66$
* Unchecked input:
  * $\mathsf{Arr}_1=\langle 84,67,11,92,36,67 \rangle$
* Execution trace:
  * $\mathsf{Acc}_1= \langle \bot,\bot,\bot,\bot,\bot,67 \rangle$​
  * $\mathsf{Acc}_1= \langle \bot,\bot,\bot,\bot,(36\cdot 67),67 \rangle$
  * $\mathsf{Acc}_1= \langle \bot,\bot,\bot,\bot,84,67 \rangle$​
  * $\mathsf{Acc}_1= \langle \bot,\bot,\bot,(92\cdot 84),84,67 \rangle$









## Polynomial-level



## Commitment-levl





A proof of assets will need to do three things ultimately:

1) Prove knowledge of the keys owned by the prover
2) At some point, map from secp to bls
3) Select the subset of balances that correspond to the owned keys

Can we do this succinctly? What is the best order to do these? What tools can we use for 2? Different answers to these questions result in different protocols. The protocol we present is not our first guess at the best protocol, it is the result of trying and studying around 5 different candidates.

It seems not possible to do (1) succinctly, however each key can can be a one-time, pre-computed proof that can be reused in each proof of solvency.

There are a few approaches to mapping from secp to bls, as well as deciding when to map. Informally, we discuss mapping a value from secp to bls, but precisely this means create a BLS public key that has the same private key (or commitment with the same committed message) as the secp public key, and prove this (equal discrete logarithms in different groups) using a COPZ proof.

The simplest design would be to do all of PoA in secp (such as with Provisions + Bulletproofs) and take the final value, a commitment to the assets, and map it to BLS. However this enjoys no potential efficiency benefit from being able to operate in bls.

A second idea would be to map each private key one by one into BLS  at the start of the protocol (as a one-time, precomputation step). The complexity is forming an anonymity set with public keys to which you do not know the private key. This can be patched with a more complex proof that maps the key when it is known and maps a 0 when it is unknown. 

The third idea is based on the observation that we do not need the secret key values themselves in BLS, we just need to know which keys the prover knows. Importantly, we do not have to map any specific value from secp to bls, we just need to capture the outcome of a proof in secp with a value in bls. This is the basis of our protocol. The prover will create a commitment to the value 0 in BLS when they cannot (or will no) complete a proof of knowledge in secp, and will create a commitment to the value 1 in BLS when they can. We can tie a proof that the commitment is 0 or 1 (in bls) to the proof of knowledge (in secp) using standard $\Sigma$-protocol conjunction and disjunction logic.

In our protocol, (3) in nearly succinct for the verifier. It does require the verifier to either: encode the balances into a polynomial themselves, or check that was done correctly. Doing it themselves requires interpolation, and FFT is $O(n \log{n})$ but these are not crypto operations. Alternatively, the prover can open the polynomial at each value which is $O(n)$ for the verifier to check, but each requires an expensive pairing. We proceed with having the verifier interpolate. This needs to be re-done for each proof, assuming at least one balance changes (if not, it can be reused).

## Protocol


| PK                | bal              | owner? | slot       | selector                   | adjust S(X)                                                | prove root        |      |      |      |
| ----------------- | ---------------- | ------ | ---------- | -------------------------- | ---------------------------------------------------------- | ----------------- | ---- | ---- | ---- |
| secp              | Plaintext        |        | BLS        | BLS                        | BLS                                                        | BLS               |      |      |      |
| $y_0=\boxed{x_0}$ | $\mathsf{bal}_0$ | No     | $\omega^0$ | $c_0=\boxed{0}=g^0h^{r_0}$ | $\boxed{S_0(X)}=\boxed{S(X)-0}=\boxed{S(X)}\cdot c_0^{-1}$ | $S_0(\omega^0)=0$ |      |      |      |
| $y_1=\boxed{x_1}$ | $\mathsf{bal}_1$ | Yes    | $\omega^1$ | $c_1=\boxed{1}=g^1h^{r_1}$ | $\boxed{S_1(X)}=\boxed{S(X)-1}=\boxed{S(X)}\cdot c_1^{-1}$ | $S_1(\omega^1)=0$ |      |      |      |
| $y_2=\boxed{x_2}$ | $\mathsf{bal}_2$ | No     | $\omega^2$ | $c_2=\boxed{0}=g^0h^{r_2}$ | $\boxed{S_2(X)}=\boxed{S(X)-0}=\boxed{S(X)}\cdot c_2^{-1}$ | $S_2(\omega^2)=0$ |      |      |      |
| $y_3=\boxed{x_3}$ | $\mathsf{bal}_3$ | Yes    | $\omega^3$ | $c_3=\boxed{1}=g^1h^{r_3}$ | $\boxed{S_3(X)}=\boxed{S(X)-1}=\boxed{S(X)}\cdot c_3^{-1}$ | $S_3(\omega^3)=0$ |      |      |      |
|                   | $B(X)$           |        |            | $S(X)$                     |                                                            |                   |      |      |      |



Selector is the output of an OR proof, computed as:

* ($c_i = \boxed{0}) \mathtt{~~OR~~} [ \{PoK: x_i | y_i \} \mathtt{~~AND~~} (c_i=\boxed{1})]$​​​

Details:

* To prove $\{PoK: x_i | y_i \}$, you can use the  Schnorr $\Sigma$-Protocol
* To prove $c_i = \boxed{x}$ for some value of $x$ that is public, you compute $c_i/g^x$ and prove you know an $r_i$ such that $h^{r_i}=c_i/g^x$ using the Schnorr $\Sigma$​​-Protocol again
* So the three clauses in the overall proof are:
  * $\pi_A=\{PoK: x_i | y_i \}$​
  * $\pi_B=\{PoK: r_i | h^{r_i}=(c_i/g^0) \}$
  * $\pi_C=\{PoK: r_i | h^{r_i}=(c_i/g^1) \}$
* Putting it together, you prove: $\pi_B \mathtt{~~OR~~} (\pi_A \mathtt{~~AND~~} \pi_c)$​
* $\pi_B$ and $\pi_c$ are in BLS, while $\pi_A$ is in secp. This is ok because the AND/OR logic is about manipulating challenge values. The only red-tape is that the challenge length is taken as the smaller of the exponent groups (secp) for all proofs. The challenge value is typically a truncated hash anyways (when using Fiat-Shamir) so we truncate it to fit into secp (which will then fit identically into bls without a modular reduction, as bls has a larger exponent group).

$B(X)$ are the balances interpolated into the points of a polynomial:

* $B(X)=\mathsf{Interpolate}\langle  \mathsf{slot},\mathsf{bal}\rangle$

$S(X)$ is an assertion of the selector values interpolated into the points of a polynomial:

* $S(X)=\mathsf{Interpolate}\langle  \mathsf{slot},\mathsf{selector}\rangle$

We have to prove $S(X)$ is constructed correctly. To do this, we open $S(X)$ at each point and show it matches $c_i$. While KZG provides a protocol for opening a polynomial to a plaintext evaluation value, it is in fact possible to open a polynomial to a commitment to the evaluation value. 

One way to conceptualize this is to consider $c_i$ as a commitment to a degree 0 polynomial. By subtracting it from $S(X)$, the resulting polynomial will be 0 at the same value which can be shown and does not reveal any information about what $c_i$​ actually was. 

$R(X)$ will represent the selected balances as:

* $R(X)=B(X)\cdot S(X)$​​

To prove $R(X)$ is constructed properly, the verifier will check $R(\kappa)-B(\kappa)\cdot S(\kappa)=0$ for a random evaluation point $\kappa$

$R(X)$ encodes the values $\langle 0, \mathsf{bal}_1, 0, \mathsf{bal}_3 \rangle$. We then construct a polynomial $A(X)$ of the same data in accumulator form starting at the "end" of the polynomial and recursing to the start:

* $A(\omega^n)=R(\omega^n)$
* $A(\omega^{i-1})=R(\omega^{i-1})+A(\omega^{i})$ for $i=n-1,n-2,\ldots,0$

Thus $A(\omega^0)=\mathsf{assets}$. $A(X)$ can be carried over to the final value from the Proof of Liabilities protocol. 

