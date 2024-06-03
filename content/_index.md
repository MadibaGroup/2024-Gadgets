---
title: Introduction
type: docs
---

# Plonkbook: A Handbook for Poly-IOP Gadgets 


*Contributors: Elizabeth van Oorschot (McGill), Youwei Deng (Concordia), & Jeremy Clark (Concordia)*

```latex
@misc{vODC24,
      author = {Elizabeth van Oorschot and Youwei Deng and Jeremy Clark},
      title = {Plonkbook: A Handbook for Poly-IOP Gadgets},
      howpublished = {GitHub Pages},
      year = {2024},
      url = {https://madibagroup.github.io/2024-Gadgets}
}
```



## Motivation

TBD



## Basic Gadgets

Below is a quick summary of various gadgets you can use in your Poly-IOP systems.



### Selections/Zeroing

| Type  | Description                                          | Recap                                                        |
| ----- | ---------------------------------------------------- | ------------------------------------------------------------ |
| zero  | $\mathsf{Arr}[i]\leftarrow0$                         | Zero-out elements of an array $\mathsf{Arr}$ such as: all, first, last, all-but-first, all-but-last. |
| zero2 | $\mathsf{Arr}[i]\leftarrow0$ iff $\mathsf{Sel}[i]=0$ | Zero-out elements of an array $\mathsf{Arr}$ according to a binary array $\mathsf{Sel}$. |



### Additions

| Type | Description                                                  | Recap                                                        |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| add1 | $\mathsf{Arr}_3=\mathsf{Arr}_1 + \mathsf{Arr}_2$             | Array $\mathsf{Arr}_3$ is the element-wise addition of $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$. |
| add2 | $\mathsf{Sum}_\mathsf{Arr}=\sum_{i = 0}^{n-1} \mathsf{Arr}[i]$ | $\mathsf{Prod}_\mathsf{Arr}$ is the disclosed sum of all the elements in array $\mathsf{Arr}$. |
| add3 | $\sum_{i = 0}^{n-1} \mathsf{Arr}_1[i]=\sum_{i = 0}^{n-1} \mathsf{Arr}_2[i]$ | Arrays $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$ have the same undisclosed sum. |



### Multiplications

| Type  | Description                                                  | Recap                                                        |
| ----- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| mult1 | $\mathsf{Arr}_3=\mathsf{Arr}_1 \cdot \mathsf{Arr}_2$         | Array $\mathsf{Arr}_3$ is the element-wise multiplication of $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$. |
| mult2 | $\mathsf{Prod}_\mathsf{Arr}=\prod_{i = 0}^{n-1} \mathsf{Arr}[i]$ | $\mathsf{Prod}_\mathsf{Arr}$ is the disclosed product of all the elements in array $\mathsf{Arr}$. |
| mult3 | $\prod_{i = 0}^{n-1} \mathsf{Arr}_1[i]=\prod_{i = 0}^{n-1} \mathsf{Arr}_2[i]$ | Arrays $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$ have the same undisclosed product. |



## Intermediate Gadgets



### Encodings

| Type   | Description                                                  | Recap                                                        |
| ------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| encode | $\mathsf{Arr}_3=\mathsf{Encode}(\mathsf{Arr}_1,\mathsf{Arr}_2)$ | Map a set of elements, such as the pair $\{\mathsf{Arr}_1[i],\mathsf{Arr}_1[2]\}$, into a single element $\mathsf{Arr}_3[i]$ without collisions. |



### Permutations/Shuffles

| Type     | Description                                            | Recap                                                        |
| -------- | ------------------------------------------------------ | ------------------------------------------------------------ |
| shuffle1 | $\mathsf{Arr}_2=\mathsf{Permute}(\mathsf{Arr}_1)$      | Array $\mathsf{Arr}_2$ is a shuffle of $\mathsf{Arr}_1$ for some undisclosed permutation $\pi$. |
| shuffle2 | $\mathsf{Arr}_2=\mathsf{Permute}(\mathsf{Arr}_1 ,\pi)$ | Array $\mathsf{Arr}_2$ is a shuffle of $\mathsf{Arr}_1$ under a disclosed permutation $\pi$. |



### Set Membership/Lookups

| Type    | Description                         | Recap                                                        |
| ------- | ----------------------------------- | ------------------------------------------------------------ |
| lookup1 | $\mathsf{Arr}[i]\in \{0,1\}$        | Each element of array $\mathsf{Arr}$ is in $\{0,1\}$ (or another small set). |
| lookup2 | $\mathsf{Arr}[i]\in \mathsf{Table}$ | Each element of array $\mathsf{Arr}$ is in a disclosed table of values $\mathsf{Table}$. |



### Range Tests

| Type  | Description               | Recap                                                        |
| ----- | ------------------------- | ------------------------------------------------------------ |
| range | $\mathsf{Arr}[i]\in[0,r]$ | Each element of array $\mathsf{Arr}$ is in the range $[0,r]$ for some upper-cap $r$. |



# Advanced Gadgets



### Circuit Evaluations

| Type  | Description            | Recap                                                        |
| ----- | ---------------------- | ------------------------------------------------------------ |
| plonk | $z=\mathsf{Circ}(x,y)$ | $z$ is the output of disclosed arithmetic circuit $\mathsf{Circ}$ with disclosed (and/or undisclosed) inputs $x$ and $y$. |

