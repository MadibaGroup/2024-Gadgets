---
title: Plonkbook
type: docs
---

# Plonkbook: A Handbook for Poly-IOP Gadgets 

*A handbook for gadgets in the polynomial interactive oracle proof (Poly-IOP) model used by Plonk.*

*Contributors: Elizabeth van Oorschot (McGill), Youwei Deng (Concordia), Jeremy Clark (Concordia)*

*Some rights reserved: [cc-by 4.0](https://creativecommons.org/licenses/by/4.0/)[^1]*

[^1]: With attribution, you are permitted to use the gadgets or proofs as written in your papers, although conference/journal policies might supersede our permission.

```latex
@misc{vODC24,
      author = {Elizabeth van Oorschot and Youwei Deng and Jeremy Clark},
      title = {Plonkbook: A Handbook for Poly-IOP Gadgets},
      howpublished = {GitHub Pages},
      year = {2024},
      url = {https://www.plonkbook.org}
}
```



## Motivation

The zoo of zk-SNARKs is complex to navigate and always changing. Groth16 (by Jens Groth in 2016) was an early protocol that was elegant, explainable, and very performant. A lot of educational resources concentrated on explaining it over its other contemporaries. However a main drawback of Groth16 is a trusted setup that cannot be shared between other people doing Groth16 proofs (unless if they are proving different inputs to the exact same function). Shareable (universal) trusted setups came in vogue. Plonk (by Ariel Gabizon, Zachary J. Williamson, & Oana Ciobotaru in 2019) has inherited the throne from Groth16 in terms of being one of the most popular zk-SNARKs to explain.

Is Plonk the "best" SNARK in 2024? By a lot of measures, it is not. Faster (both asymptotically and concretely) prover operations can be found elsewhere. Proof size and verifier time can be squeezed more than Plonk. Trusted setups can be abandoned altogether. Yet Plonk is *pretty good*. Importantly, it is also *pretty understandable*. And it is *pretty flexible*. For these reasons, Plonk is a great place to start when you are learning about SNARKs and when designing a custom protocol for proving something directly—not as a circuit on top of Plonk but using the same building blocks (or "gadgets") that Plonk itself uses.



## Gadgets

Below is a quick summary of various gadgets you can use in your Poly-IOP systems.

| Gadget   | Short Description                                            | Recap                                                        |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| zero1    | $\mathsf{Arr}[i]\leftarrow0$                                 | Zero-out elements of an array $\mathsf{Arr}$ such as: all, first, last, all-but-first, all-but-last. |
| zero2    | $\mathsf{Arr}[i]\leftarrow0$ iff $\mathsf{Sel}[i]=0$         | Zero-out elements of an array $\mathsf{Arr}$ according to a binary array $\mathsf{Sel}$. |
| rotate   | $\mathsf{Arr}[i]\leftarrow \mathsf{Arr}[i+\alpha]$           | Rotate array $\mathsf{Arr}$ by $\alpha$ positions.           |
| add1     | $\mathsf{Arr}_3=\mathsf{Arr}_1 + \mathsf{Arr}_2$             | Array $\mathsf{Arr}_3$ is the element-wise addition of $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$. |
| add2     | $\mathsf{Sum}_\mathsf{Arr}=\sum_{i = 0}^{n-1} \mathsf{Arr}[i]$ | $\mathsf{Sum}_\mathsf{Arr}$ is the disclosed sum of all the elements in array $\mathsf{Arr}$. |
| add3     | $\sum_{i = 0}^{n-1} \mathsf{Arr}_1[i]=\sum_{i = 0}^{n-1} \mathsf{Arr}_2[i]$ | Arrays $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$ have the same undisclosed sum. |
| mult1    | $\mathsf{Arr}_3=\mathsf{Arr}_1 \cdot \mathsf{Arr}_2$         | Array $\mathsf{Arr}_3$ is the element-wise multiplication of $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$. |
| mult2    | $\mathsf{Prod}_\mathsf{Arr}=\prod_{i = 0}^{n-1} \mathsf{Arr}[i]$ | $\mathsf{Prod}_\mathsf{Arr}$ is the disclosed product of all the elements in array $\mathsf{Arr}$. |
| mult3    | $\prod_{i = 0}^{n-1} \mathsf{Arr}_1[i]=\prod_{i = 0}^{n-1} \mathsf{Arr}_2[i]$ | Arrays $\mathsf{Arr}_1$ and $\mathsf{Arr}_2$ have the same undisclosed product. |
| encode   | $\mathsf{Arr}_3\leftarrow\mathsf{Encode}(\mathsf{Arr}_1,\mathsf{Arr}_2)$ | Map a set of elements, such as the pair $\{\mathsf{Arr}_1[i],\mathsf{Arr}_2[i]\}$, into a single element $\mathsf{Arr}_3[i]$ without collisions. |
| shuffle1 | $\mathsf{Arr}_2=\mathsf{Permute}(\mathsf{Arr}_1)$            | Array $\mathsf{Arr}_2$ is a shuffle of $\mathsf{Arr}_1$ for some undisclosed permutation $\pi$. |
| shuffle2 | $\mathsf{Arr}_2=\mathsf{Permute}(\mathsf{Arr}_1 ,\pi)$       | Array $\mathsf{Arr}_2$ is a shuffle of $\mathsf{Arr}_1$ under a disclosed permutation $\pi$. |
| lookup1  | $\mathsf{Arr}[i]\in \{0,1\}$                                 | Each element of array $\mathsf{Arr}$ is in $\{0,1\}$ (or another small set). |
| lookup2  | $\mathsf{Arr}[i]\in \mathsf{Table}$                          | Each element of array $\mathsf{Arr}$ is in a disclosed table of values $\mathsf{Table}$. |
| range    | $\mathsf{Arr}[i]\in[0,r]$                                    | Each element of array $\mathsf{Arr}$ is in the range $[0,r]$ for some upper-cap $r$. |
| circuit  | $z=\mathsf{Circ}(x,y)$                                       | $z$ is the output of disclosed arithmetic circuit $\mathsf{Circ}$ with disclosed (and/or undisclosed) inputs $x$ and $y$. |
|          |                                                              |                                                              |
|          |                                                              |                                                              |
|          |                                                              |                                                              |

