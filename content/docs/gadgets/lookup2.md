# Lookup (Type 2)

## Recap of types

| Type | Description | Recap | This |
| ---- | ----------- | ----- | ---- |
| lookup1  | $\mathsf{Arr}[i]\in \{0,1\}$ | Each element of array $\mathsf{Arr}$ is in $\{0,1\}$ (or another small set). |
| lookup2  | $\mathsf{Arr}[i]\in \mathsf{Table}$ | Each element of array $\mathsf{Arr}$ is in a disclosed table of values $\mathsf{Table}$. | ✅ |

## Relation

$ \mathcal{R}_{\mathtt{lookup2}} := \left\{ \begin{array}{l} (\mathsf{Arr},\mathsf{T}) \end{array} \middle | \begin{array}{l} \mathsf{Arr}[i]\in\mathsf{T}, 0\leq i \leq n-1, \\ \mathsf{Poly}_\mathsf{Arr}=\mathsf{FFT.Interp}(\omega,\mathsf{Arr}), \\ \mathsf{Poly}_\mathsf{T}=\mathsf{FFT.Interp}(\omega,\mathsf{T}), \\ K_\mathsf{Arr}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr}), \\ K_\mathsf{T}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{T}), \end{array} \right\} $

## Intuition

The prover ($\mathcal{P}$) holds an array $\mathsf{Arr}$ of $n$ integers from $\mathbb{Z}_q$: $[a_0, a_1, a_2, \dots, a_{n-1}]$. It will produce a succinct (independent of $n$) proof that each value in $\mathsf{Arr}$ is the element of a public table $\mathsf{T}$. The prover will encode $\mathsf{Arr}$ and $\mathsf{T}$ into polynomials: $\mathsf{Poly}_\mathsf{Arr}$ and $\mathsf{Poly}_\mathsf{T}$. Assume $\mathsf{Arr}$ is equal to $\mathsf{T}$, then the prover can complete the proof by simply performing the product check or the permutation check between $\mathsf{Arr}$ and $\mathsf{T}$. However, if $\mathsf{Arr}$ is not equal to $\mathsf{T}$ (this is the case we want to solve), the prover needs to reveal the points where $\mathsf{Arr}$ equals to $\mathsf{T}$, which makes zero knowledge impossible. Thus, the prover needs to construct auxiliary polynomial(s) to prove the constraints between $\mathsf{Arr}$ and $\mathsf{T}$ are desired. 

There are different approaches to achieving the lookup argument, e.g., [halo2](https://zcash.github.io/halo2/design/proving-system/lookup.html) and [Plookup](https://o1-labs.github.io/proof-systems/plonk/plookup.html). We will discuss these two approaches as follows.
