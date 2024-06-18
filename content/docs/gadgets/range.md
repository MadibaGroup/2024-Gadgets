# Range

## Recap of types

| Type | Description | Recap | This |
| ---- | ----------- | ----- | ---- |
| range | $\mathsf{Arr}[i]\in[0,r]$ | Each element of array $\mathsf{Arr}$ is in the range $[0,r]$ | ✅ |

## Relation

$\mathcal{R}_{\mathtt{add1}} := \left\{ \begin{array}{l} (K_\mathsf{Arr}) \end{array} \middle | \begin{array}{l} 0\le{\mathsf{Arr}[i]}\le{r}, i\in[0,n), \\ \mathsf{Poly}_\mathsf{Arr}=\mathsf{FFT.Interp}(\omega,\mathsf{Arr}), \\ K_\mathsf{Arr}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr}), \end{array} \right\}$
