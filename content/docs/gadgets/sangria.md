# Sangria

## Intuition

Given a PLONK circuit, and two private input/public input pairs, we want to reduce the work of checking each of these individually to the work of checking such a relation a single time. This is called folding. First, though, let's review how a PLONK circuit works.

In [circuit](../circuit) we look at how a circuit with a single gate works, but here we generalize to multi-gate circuits. Here we have vectors $\mathsf{L}$, $\mathsf{R}$, and $\mathsf{O}$ representing the right and left input, and the output, of each gate, and together, $(\mathsf{L}, \mathsf{R}, \mathsf{O})$ is called the computational trace. These three vectors are divided into $X$, the public inputs, and $W$, the private inputs, or witness. A PLONK proof will show that a public input/private input pair $(X, W)$ satisfies a circuit defined by the tuple $(\mathsf{Q}, \mathsf{S})$, where $\mathsf{Q}$ is the set of selector vectors, and $\mathsf{S}$ is the set of copy contraints. A circuit being satisfied means that the copy constraints are satisfied, and that each gate is satisfied. The $i^{th}$ gate of the circuit is defined as ${q_L}_i \cdot l_i + {q_R}_i \cdot r_i + {q_O}_i \cdot o_i + {q_M}_i \cdot l_i \cdot r_i + {q_c}_i$ where ${q_L}_i, {q_R}_i, {q_O}_i, {q_M}_i, {q_c}_i$ are the $i^{th}$ values of each selector vector, and $l_i, r_i, o_i$ are the $i^{th}$ values of  $\mathsf{L}$, $\mathsf{R}$, and $\mathsf{O}$.

Now, we turn to folding. Say we want to combine the checks for two private input/public input pairs, $(X', W')$ and $(X'', W'')$. We start by trying the most direct appraoch to reducing our two checks down to one: let's take a random linear combination of $(X', W')$ and $(X'', W'')$, and perform the check on it. Then our new private input/public input pair would be $(X, W) := (X' + sX'', W' + sW'')$.  We consider this as input to the circuit: $C_{Q,i}(X, W)$ $= C_{Q,i}(X' + sX'', W' + sW'')$

$= {q_L}_i \cdot (l_{1i} + sl_{2i} ) + {q_R}_i \cdot (r_{1i} + sr_{2i}) + {q_O}_i \cdot (o_{1i} + so_{2i}) + {q_M}_i \cdot (l_{1i} + s1_{2i}) (r_{1i} + sr_{2i}) + {q_c}_i$

$= {q_L}_i \cdot l_{1i} + {q_L}_i \cdot sl_{2i}  + {q_R}_i \cdot r_{1i} + {q_R}_i \cdot sr_{2i} + {q_O}_i \cdot o_{1i} + {q_O}_i \cdot so_{2i} + {q_M}_i \cdot (l_{1i} + s1_{2i}) (r_{1i} + sr_{2i}) + {q_c}_i$

$= C_{Q,i}(X', W') + s^2C_{Q,i}(X'', W'') + XXX$

As we can see, we end up with undesirable cross terms XXX our equation no longer has the right format, also our equation just isn't true now XXX. This motivates us to define relaxed PLONK gate equations. Copy constraints are defined the same in relaxed PLONK, but we defined our gates somewhat differently. 

Here, the $i^{th}$ gate is defined as $u({q_L}_i \cdot l_i + {q_R}_i \cdot r_i + {q_O}_i \cdot o_i) + {q_M}_i \cdot l_i \cdot r_i + u^2({q_c}_i) + e_i$ where $u$ is a scalar and $e_i$ is the $i^{th}$ entry in the "slack vector" $\mathsf{E}$. We now have $(\mathsf{L}, \mathsf{R}, \mathsf{O}, u, \mathsf{E})$ as the computational trace. We must also define our public input/private input pair differently, namely for the regular PLONK pair $(X, W)$ we define the relaxed PLONK pair $(U, V)$ as:

$U := (X, u, \overline W_a, \overline W_b, \overline W_c, \overline E)$

$V:= (W, \mathsf{E}, r_a, r_b, r_c, r_e)$

Where $\overline W_a = \mathsf{com}(w_a, r_a)$, $\overline W_b = \mathsf{com}(w_b, r_b)$, $\overline W_c = \mathsf{com}(w_c, r_c)$, $\overline E = \mathsf{com}(\mathsf{E}, r_e)$.

Note that regular PLONK relation can be represented as a relaxed PLONK relation simply by setting $u=1$ and $\mathsf{E}=\vec{0}$. 

Now, we once again try the random linear combination approach, this time with relaxed PLONK gate equations. The verifier is given the verifier key, and the two sets of public inputs XXX and XXX. The prover has the prover key and both sets of corresponding private inputs. The protocol proceeds as follows:

1. 

## Security Proof

### Completeness

- explicitly write out the algebra here

### Soundness

- forking lemma???

### Zero-Knowledge

- all the commitments are hiding etc