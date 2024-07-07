# Zero (Type 1)

## Recap of types

| Type              | Description                                          | Recap                                                        | This |
| ----------------- | ---------------------------------------------------- | ------------------------------------------------------------ | ---- |
| [zero1](../zero1) | $\mathsf{Arr}[i]\leftarrow0$                         | Zero-out elements of an array $\mathsf{Arr}$ such as: all, first, last, all-but-first, all-but-last. | ✅    |
| [zero2](../zero2) | $\mathsf{Arr}[i]\leftarrow0$ iff $\mathsf{Sel}[i]=0$ | Zero-out elements of an array $\mathsf{Arr}$ according to a binary array $\mathsf{Sel}$. |      |

### Subtypes of $\texttt{zero1}$

| Operation          | Output Array                            |
| ------------------ | --------------------------------------- |
| Zero all           | $\langle 0,0,0,0,0 \rangle$             |
| Zero first         | $\langle 0,\bot,\bot,\bot,\bot \rangle$ |
| Zero last          | $\langle \bot,\bot,\bot,\bot,0 \rangle$ |
| Zero all but first | $\langle \bot,0,0,0,0 \rangle$          |
| Zero all but last  | $\langle 0,0,0,0,\bot \rangle$          |

## Relation

Shown for "zero last" but can be adapted to the other sub-types of $\texttt{zero1}$:



  $ \mathcal{R}_{\mathtt{mult2}} := \left\{ \begin{array}{l} (K_\mathsf{Arr},K_\mathsf{Arr'}) \end{array} \middle | \begin{array}{l} \mathsf{Arr} = [a_0, a_1, \dots, a_{n-2}, a_{n-1}], \\ \mathsf{Arr'} = [a'_0, a'_1, \dots, a'_{n-2}, 0], \\ \mathsf{Poly}_\mathsf{Arr}=\mathsf{FFT.Interp}(\omega,\mathsf{Arr}), \\ \mathsf{Poly}_\mathsf{Arr'}=\mathsf{FFT.Interp}(\omega,\mathsf{Arr'}), \\ K_\mathsf{Arr}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr}) \\ K_\mathsf{Arr'}=\mathsf{KZG.Commit}(\mathsf{Poly}_\mathsf{Arr'}) \end{array} \right\} $

## Intuition

This gadget is a helper gadget that is not very useful as a stand-alone gadget, but is used in nearly all other gadgets to handle things like boundary conditions (see Red Tape). The semantics of the gadget might seem weird but eventually you will see it in use (e.g., $\texttt{add2}$). The prover ($\mathcal{P}$) holds an array $\mathsf{Arr} = [a_0, a_1, a_2, \dots, a_{n-1}]$ of $n$ integers (from $\mathbb{Z}_q$). It will produce a succinct (independent of $n$) proof that $\mathsf{Arr'}$ contains a 0 at a selected element (such as the last element). At the other elements, the operation is zero-preserving, meaning it will not replace a 0 in $\mathsf{Arr}$. If an element in $\mathsf{Arr}$ is non-zero and the 



it contains $\bot$ which is  an arbitrary integer 

 the same at every element 

 $\mathsf{Prod}_\mathsf{Arr}$ is the product of all the elements in the array. The prover will encode the array into a polynomial $\mathsf{Poly}_\mathsf{Arr}$ (using [evaluation points](../../background/poly-iop) on the domain $\mathcal{H}_\kappa$) and commit to the polynomial $K_\mathsf{Arr}$. The verifier ($\mathcal{V}$) cannot check $\textsf{Arr}$ or $\mathsf{Poly}_\mathsf{Arr}$ directly (they may contain secret information, and even if they do not, it is too long to check) so the verifier only sees $K_\mathsf{Arr}$ and the asserted value $\mathsf{Prod_\mathsf{Arr}}$.Zeroing Parts of an Array

Assuming an input array of size $n$: $\langle \mathsf{data}_0,\mathsf{data}_1,\ldots,\mathsf{data}_{n-1}\rangle$ and input array encoded into the polynomial. This uses "Encoding 2" from above (evaluation points) and uses "Roots of Unity + FFT" from above where $\omega\in\mathbb{G}_\kappa$ is a generator for the x-coordinates of the points.

$\bot$ is an arbitrary non-zero integer.

| Operation          | Input Array                 | Input Polynomial | Output Array                            | Output Polynomial                                     |
| ------------------ | --------------------------- | ---------------- | --------------------------------------- | ----------------------------------------------------- |
| Zero all           | $\langle 3,1,3,3,7 \rangle$ | $P(X)$           | $\langle 0,0,0,0,0 \rangle$             | $P(X)\cdot(X^\kappa-1)$                               |
| Zero first         | $\langle 3,1,3,3,7 \rangle$ | $P(X)$           | $\langle 0,\bot,\bot,\bot,\bot \rangle$ | $P(X)\cdot(X-\omega^0)$                               |
| Zero last          | $\langle 3,1,3,3,7 \rangle$ | $P(X)$           | $\langle \bot,\bot,\bot,\bot,0 \rangle$ | $P(X)\cdot(X-\omega^{\kappa-1})$                      |
| Zero all but first | $\langle 3,1,3,3,7 \rangle$ | $P(X)$           | $\langle \bot,0,0,0,0 \rangle$          | $P(X)\cdot\frac{(X^\kappa-1)}{(X-\omega^0)}$          |
| Zero all but last  | $\langle 3,1,3,3,7 \rangle$ | $P(X)$           | $\langle 0,0,0,0,\bot \rangle$          | $P(X)\cdot\frac{(X^\kappa-1)}{(X-\omega^{\kappa-1})}$ |

