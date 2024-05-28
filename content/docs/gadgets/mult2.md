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



## Commitment-level



