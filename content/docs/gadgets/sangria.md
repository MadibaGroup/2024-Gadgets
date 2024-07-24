# Sangria

## Intuition

Given a PLONK circuit, and two private input/public input pairs, we want to reduce the work of checking each of these individually to the work of checking such a relation a single time. This is called folding. First, though, let's review how a PLONK circuit works.

In [circuit](../circuit) we look at how a circuit with a single gate works, but here we generalize to multi-gate circuits. Here we have vectors $\mathsf{L}$, $\mathsf{R}$, and $\mathsf{O}$ representing the right and left input, and the output, of each gate, and together, $(\mathsf{L}, \mathsf{R}, \mathsf{O})$ is called the computational trace. These three vectors are divided into $X$, the public inputs, and $W$, the private inputs, or witness. A PLONK proof will show that a public input/private input pair $(X, W)$ satisfies a circuit defined by the tuple $(\mathsf{Q}, \mathsf{S})$, where $\mathsf{Q}$ is the set of selector vectors, and $\mathsf{S}$ is the set of copy contraints. A circuit being satisfied means that the copy constraints are satisfied, and that each gate is satisfied. The $i^{th}$ gate of the circuit is defined as ${q_L}_i \cdot l_i + {q_R}_i \cdot r_i + {q_O}_i \cdot o_i + {q_M}_i \cdot l_i \cdot r_i + {q_c}_i$ where ${q_L}_i, {q_R}_i, {q_O}_i, {q_M}_i, {q_c}_i$ are the $i^{th}$ values of each selector vector, and $l_i, r_i, o_i$ are the $i^{th}$ values of  $\mathsf{L}$, $\mathsf{R}$, and $\mathsf{O}$.

Now, we turn to folding. Say we want to combine the checks for two private input/public input pairs, $(X', W')$ and $(X'', W'')$. We start by trying the most direct appraoch to reducing our two checks down to one: let's take a random linear combination of $(X', W')$ and $(X'', W'')$, and perform the check on it. Then our new private input/public input pair would be $(X, W) := (X' + sX'', W' + sW'')$.  We consider this as input to the circuit: $C_{Q,i}(X, W)$ $= C_{Q,i}(X' + sX'', W' + sW'')$

$= {q_L}_i \cdot (l_{1i} + sl_{2i} ) + {q_R}_i \cdot (r_{1i} + sr_{2i}) + {q_O}_i \cdot (o_{1i} + so_{2i}) + {q_M}_i \cdot (l_{1i} + s1_{2i}) (r_{1i} + sr_{2i}) + {q_C}_i$

$= {q_L}_i \cdot l_{1i} + {q_L}_i \cdot sl_{2i}  + {q_R}_i \cdot r_{1i} + {q_R}_i \cdot sr_{2i} + {q_O}_i \cdot o_{1i} + {q_O}_i \cdot so_{2i} + {q_M}_i \cdot (l_{1i} + s1_{2i}) (r_{1i} + sr_{2i}) + {q_C}_i$

$\neq C_{Q,i}(X', W') + C_{Q,i}(X'', W'')$

We note the result is no longer of the correct form for a PLONK gate. We end up with an undesirable crossterms, an $s^2$ in front of at least part of $C_{Q,i}(X'', W'')$, and if we want to claim $C_{Q,i}(X, W) = C_{Q,i}(X', W') + C_{Q,i}(X'', W'')$, it simply isn't true. This motivates us to define relaxed PLONK gate equations, which will allow us to deal with these issues. Copy constraints are defined the same in relaxed PLONK, but we defined our gates somewhat differently. 

Here, the $i^{th}$ gate is defined as $u({q_L}_i \cdot l_i + {q_R}_i \cdot r_i + {q_O}_i \cdot o_i) + {q_M}_i \cdot l_i \cdot r_i + u^2({q_c}_i) + e_i$ where $u$ is a scalar and $e_i$ is the $i^{th}$ entry in the "slack vector" $\mathsf{E}$. We now have $(\mathsf{L}, \mathsf{R}, \mathsf{O}, u, \mathsf{E})$ as the computational trace. We must also define our public input/private input pair differently, namely for the regular PLONK pair $(X, W)$ we define the relaxed PLONK pair $(U, V)$ as:

$U := (X, u, \overline W_l, \overline W_r, \overline W_o, \overline E)$

$V:= (W, \mathsf{E}, r_l, r_r, r_o, r_e)$

Where $\overline W_l = \mathsf{com}(w_l, r_l)$, $\overline W_r = \mathsf{com}(w_r, r_r)$, $\overline W_o = \mathsf{com}(w_o, r_o)$, $\overline E = \mathsf{com}(\mathsf{E}, r_e)$.

Note that regular PLONK relation can be represented as a relaxed PLONK relation simply by setting $u=1$ and $\mathsf{E}=\vec{0}$. 

Now, we once again try the random linear combination approach, this time with relaxed PLONK gate equations. The verifier is given the verifier key, and the two sets of public inputs $(X', u', \overline W_l', \overline W_r', \overline W_o', \overline E')$ and $(X'', u'', \overline W_l'', \overline W_r'', \overline W_o'', \overline E'')$. The prover has the prover key and both sets of corresponding private inputs: $(W', \mathsf{E}', r_l', r_r', r_o', r_e')$ and $(W'', \mathsf{E}'', r_l'', r_r'', r_o'', r_e'')$. The protocol (which is a public-coin folding scheme) proceeds as follows:

## Protocol Details

1. $\mathcal{P}$ computes $t= u''(q_L \circ l' + q_R \circ r' + q_O \circ o') + u'(q_L \circ l'' + q_R \circ r'' + q_O \circ o'') + q_M \circ (l' \circ r'' + l'' \circ r') + 2u'u''q_C$ where $\circ$ denotes element-wise multiplication. This $t$ is used to account for the crossterms.
2. $\mathcal{P}$ samples a random $r_t$ sends $\mathsf{com}(\mathsf{pp_E}, t, r_t)$.
3. $\mathcal{V}$ samples a random challenge $r$.
4. $\mathcal{P}$ and $\mathcal{V}$ output the folded public input $(X, u, \overline W_l, \overline W_r, \overline W_o, \overline E)$, computed as:
5. $\mathcal{P}$ outputs the folded private input $(W, \mathsf{E}, r_l, r_r, r_o, r_e)$, computed as:

The resulting public and private inputs constitute a new PLONK pair $(U, V)$ which is the folding of the two input pairs.

## Security Proof

### Completeness

To show completeness, we observe that $C_{Q,i}(l, r, o, u, \mathsf{E})$

$= C_{Q,i}(l' + sl'', r' + sr'', o' + so'', u' + su'', \mathsf{E}' -sT + s^2\mathsf{E}'')$

$= (u' + su'') \cdot [{q_L}_i \cdot (l' + sl'') + {q_R}_i \cdot (r' + sr'') + {q_O}_i \cdot (o' +so'')] + {q_M}_i \cdot (l' + sl'') (r' + sr'') + (u' + su'')^2{q_C}_i + \mathsf{E}' -st + s^2\mathsf{E}''$

$= u'(q_{L_i} \cdot l' + q_{R_i} \cdot r' + q_{O_i} \cdot o' + q_{M_i} \cdot l' \cdot r' + u' \cdot q_{C_i}) + \mathsf{E}' + u'' \cdot s^2(q_{L_i} \cdot l'' + q_{R_i} \cdot r'' + q_{O_i} \cdot o'' + q_{M_i} \cdot l'' \cdot r'' + u'' \cdot q_{C_i}) + s^2\mathsf{E}'' \newline + u''(q_{L_i} \circ l' + q_{R_i} \circ r' + q_{O_i} \circ o') + u'(q_{L_i} \circ l'' + q_{R_i} \circ r'' + q_{O_i} \circ o'') + q_{M_i} \circ (l' \circ r'' + l'' \circ r') + 2u'u''q_{C_i} - st$

$= u'(q_{L_i} \cdot l' + q_{R_i} \cdot r' + q_{O_i} \cdot o' + q_{M_i} \cdot l' \cdot r' + u' \cdot q_{C_i}) + \mathsf{E}' + u'' \cdot s^2(q_{L_i} \cdot l'' + q_{R_i} \cdot r'' + q_{O_i} \cdot o'' + q_{M_i} \cdot l'' \cdot r'' + u'' \cdot q_{C_i}) + s^2\mathsf{E}''$

$ = C_{Q,i}(r', l', o', u', \mathsf{E}') + s^2 \cdot C_{Q,i}(r'', l'', o'', u'', \mathsf{E}'')$

So following the steps of the protocol above provides a folding that satisfies completeness.

### Soundness

- forking lemma???

### Zero-Knowledge

- all the commitments are hiding etc