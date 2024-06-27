# Concatenation

## Recap of types

| Type | Description | Recap | This |
| ---- | ----------- | ----- | ---- |
| concat | $\mathsf{Arr}_3=\mathsf{Arr}_1\cup\mathsf{Arr}_2$ | $\mathsf{Arr}_3$ is the concatenation of $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$ | ✅ |

## Relation

$$
\mathcal{R}_{\mathtt{mult1}} := \left\{ \begin{array}{l} (K_{\mathsf{Arr}_1},K_{\mathsf{Arr}_2},K_{\mathsf{Arr}_3}) \end{array} \middle | \begin{array}{l} \mathsf{Arr}_3=\mathsf{Arr}_1\cup\mathsf{Arr}_2, \\ \mathsf{Arr}_3[i]=\mathsf{Arr}_1[i],i\in[0,n_1), \\ \mathsf{Arr}_3[i+n_1]=\mathsf{Arr}_2[i],i\in[0,n_2), \\ \mathsf{Poly}_\mathsf{Arr_j}=\mathsf{FFT.Interp}(\omega,\mathsf{Arr_j}), 1\leq j \leq 3, \\ K_\mathsf{Arr_j}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr_j}), 1\leq j \leq 3, \end{array} \right\}
$$

## Intuition

The prover ($\mathcal{P}$) holds three arrays $\mathsf{Arr}_1$, $\mathsf{Arr}_2$ and $\mathsf{Arr_3}$. He wants to convince the verifier ($\mathcal{V}$) $\mathsf{Arr}_3$ is the concatenation of $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$. Specifically, assume there are $n_1$, $n_2$, and $n_3$ elements in $\mathsf{Arr}_1$, $\mathsf{Arr}_2$, and $\mathsf{Arr}_3$ respectively, then the prover will prove: (i) $\mathsf{Arr}_3[i]=\mathsf{Arr}_1[i],i\in[0,n_1)$ (ii) $\mathsf{Arr}_3[i+n_1]=\mathsf{Arr}_2[i],i\in[0,n_2)$. It is intuitive to think about using the grand product check in the Plookup. However, the product check of Plookup holds if and only if (i) $\mathsf{Arr}_1$ is the subset of $\mathsf{Arr}_2$ (ii) $\mathsf{Arr}_3$ is the union set of $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$ (iii) $\mathsf{Arr}_3$ is sorted by $\mathsf{Arr}$. Thus, we cannot use the product check directly, but we can leverage the same fact in the product check of Plookup: the product of $\mathsf{Arr}_3$ equals the product of $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$ if and only if $\mathsf{Arr}_3$ is the union set of $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$ (permutation check). But we need to prove the concatenation relationship rather than the permutation. To solve this problem, we can break $\mathsf{Arr}_3$ to two sub arrays $\mathsf{Arr}_{3_l}$ and $\mathsf{Arr}_{3_h}$ as we do in Plookup, and prove the product of $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$ equals the product of $\mathsf{Arr}_{3_l}$ and $\mathsf{Arr}_{3_h}$.

## Protocol Details

### Array Level

### Polynomial Level

$\prod_{i<{n}}[\alpha(1+\beta)+\mathsf{Poly}_{\mathsf{Arr}_1}(\omega^i)+\beta\mathsf{Poly}_{\mathsf{Arr}_1}(\omega^{i+1})]\cdot\prod_{i<{d-1}}[\alpha(1+\beta)+\mathsf{Poly}_{\mathsf{Arr}_2}(\omega^i)+\beta\mathsf{Poly}_{\mathsf{Arr}_2}(\omega^{i+1})]=\prod_{i<{n+d-1}}[\alpha(1+\beta)+\mathsf{Poly}_{\mathsf{Arr}_3}(\omega^i)+\beta\mathsf{Poly}_{\mathsf{Arr}_3}(\omega^{i+1})]$

### Commitment Level
