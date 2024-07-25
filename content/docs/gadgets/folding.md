# Folding (Sangria)

The following document is heavily based on the [Sangria techincal note](https://github.com/geometryxyz/technical_notes/blob/main/sangria_folding_plonk.pdf) and its [condensed form](https://geometry.xyz/notebook/sangria-a-folding-scheme-for-plonk).

## Intuition

Given a PLONK circuit, and two private input/public input pairs, we want to reduce the work of checking each of these individually to the work of checking one such relation a single time. This is called folding, and the model we explain here for Plonk, specifically, is called Sangria. Before we get into folding, though, let's review how a PLONK circuit works.

In [circuit](../circuit) we look at how a circuit with a single gate works, but now we generalize to multi-gate circuits. Here we have vectors $\mathsf{L}$, $\mathsf{R}$, $\mathsf{O}$ $\in \mathbb{F}^{s + n +1}$ representing the right and left input, and the output, of each gate, where $n$ is the number of public inputs and $s$ is the number of gates. The $+1$ is an extra row to check the final result, i.e. if the circuit is satisfied. Together, $(\mathsf{L}, \mathsf{R}, \mathsf{O})$ is called the computational trace. These three vectors are divided into $X$, the public inputs, and $W$, the private inputs, or witness. A PLONK proof will show that a public input/private input pair $(X, W)$ satisfies a circuit defined by the tuple $(\mathsf{Q}, \mathsf{S})$, where $\mathsf{Q}$ is the set of selector vectors, and $\mathsf{S}$ is the set of copy contraints. A circuit being satisfied means that the copy constraints are satisfied, and that each gate is satisfied. The $i^{th}$ gate of the circuit is defined as ${q_L}_i \cdot l_i + {q_R}_i \cdot r_i + {q_O}_i \cdot o_i + {q_M}_i \cdot l_i \cdot r_i + {q_c}_i$ where ${q_L}_i, {q_R}_i, {q_O}_i, {q_M}_i, {q_c}_i$ are the $i^{th}$ values of each selector vector, and $l_i, r_i, o_i$ are the $i^{th}$ values of  $\mathsf{L}$, $\mathsf{R}$, and $\mathsf{O}$.

Now, we turn to folding. Say we want to combine the checks for two private input/public input pairs, $(X', W')$ and $(X'', W'')$. We start by trying the most direct appraoch to reducing our two checks down to one: let's take a random linear combination of $(X', W')$ and $(X'', W'')$, and perform the check on it. Then our new private input/public input pair would be $(X, W) := (X' + sX'', W' + sW'')$.  We consider this as input to the circuit: $C_{Q,i}(\mathsf{L}, \mathsf{R}, \mathsf{O})$ $= C_{Q,i}(\mathsf{L}' + s\mathsf{L}'', \mathsf{R}' + s\mathsf{R,}'', \mathsf{O} + s\mathsf{O}'')$

$= {q_L}_i \cdot (l_{i}' + sl_{i}'' ) + {q_R}_i \cdot (r_{i}' + sr_{i}'') + {q_O}_i \cdot (o_{i}' + so_{i}'') + {q_M}_i \cdot (l_{i}' + s1_{i}'') (r_{i}' + sr_{i}'') + {q_C}_i$

$= {q_L}_i \cdot l_{i}' + {q_L}_i \cdot sl_{i}''  + {q_R}_i \cdot r_{i}' + {q_R}_i \cdot sr_{i}'' + {q_O}_i \cdot o_{i}' + {q_O}_i \cdot so_{i}'' + {q_M}_i \cdot (l_{i}'+ s1_{i}'') (r_{i}' + sr_{i}'') + {q_C}_i$

$\neq C_{Q,i}(\mathsf{L}', \mathsf{R}', \mathsf{O}')+ C_{Q,i}(\mathsf{L}'', \mathsf{R}'', \mathsf{O}'')$

We note the result is no longer of the correct form for a PLONK circuit. We end up with an undesirable crossterms, an $s^2$ in front of at least part of $C_{Q,i}(\mathsf{L}'', \mathsf{R}'', \mathsf{O}'')$, and if we want to claim $C_{Q,i}(\mathsf{L}, \mathsf{R}, \mathsf{O}) = C_{Q,i}(\mathsf{L}', \mathsf{R}', \mathsf{O}')+ C_{Q,i}(\mathsf{L}'', \mathsf{R}'', \mathsf{O}'')$, it simply isn't true. This motivates us to define relaxed PLONK gate equations, which will allow us to deal with these issues. Copy constraints are defined the same in relaxed PLONK, but we defined our gates somewhat differently. 

Here, the $i^{th}$ gate is defined as $u({q_L}_i \cdot l_i + {q_R}_i \cdot r_i + {q_O}_i \cdot o_i) + {q_M}_i \cdot l_i \cdot r_i + u^2({q_c}_i) + e_i$ where $u$ is a scalar and $e_i$ is the $i^{th}$ entry in the "slack vector" $\mathsf{E}$. We now have $(\mathsf{L}, \mathsf{R}, \mathsf{O}, u, \mathsf{E})$ as the computational trace. We must also define our public input/private input pair differently, namely for the regular PLONK pair $(X, W)$ we define the relaxed PLONK pair $(U, V)$ as:

$U := (X, u, \overline W_l, \overline W_r, \overline W_o, \overline E)$

$V:= (W, \mathsf{E}, r_l, r_r, r_o, r_e)$

Where $\overline W_l = \mathsf{com}(w_l, r_l)$, $\overline W_r = \mathsf{com}(w_r, r_r)$, $\overline W_o = \mathsf{com}(w_o, r_o)$, $\overline E = \mathsf{com}(\mathsf{E}, r_e)$.  For each of these commitments (and elsewhere in this document) the public parameters are including implicitly for cleaner notation.

We define the relaxed PLONK gate equation as:

$C'_{Q,i}= (u) \cdot [{q_L}_i \cdot l + {q_R}_i \cdot r + {q_O}_i \cdot o] + {q_M}_i \cdot l r + u^2{q_C}_i + e_i$

Note that regular PLONK relation can be represented as a relaxed PLONK relation simply by setting $u=1$ and $\mathsf{E}=\vec{0}$. 

## Protocol Details

Now, we once again try the random linear combination approach, this time with relaxed PLONK gate equations. The verifier is given the verifier key, and the two sets of public inputs $(X', u', \overline W_l', \overline W_r', \overline W_o', \overline E')$ and $(X'', u'', \overline W_l'', \overline W_r'', \overline W_o'', \overline E'')$. The prover has the prover key and both sets of corresponding private inputs: $(W', \mathsf{E}', r_l', r_r', r_o', r_e')$ and $(W'', \mathsf{E}'', r_l'', r_r'', r_o'', r_e'')$. The protocol (which is a public-coin folding scheme) proceeds as follows:

1. $\mathcal{P}$ computes $\mathsf{T}= u''(q_L \circ l' + q_R \circ r' + q_O \circ o') + u'(q_L \circ l'' + q_R \circ r'' + q_O \circ o'') + q_M \circ (l' \circ r'' + l'' \circ r') + 2u'u''q_C$ where $\circ$ denotes element-wise multiplication. This $t$ is used to account for the crossterms.
2. $\mathcal{P}$ samples a random $r_T$ sends $\overline T = \mathsf{com}(\mathsf{T}, r_T)$.
3. $\mathcal{V}$ samples a random challenge $r$ and sends it.
4. $\mathcal{P}$ and $\mathcal{V}$ output the folded public input $(X, u, \overline W_l, \overline W_r, \overline W_o, \overline E)$, computed as:

$\quad X = X' + sX''$

$\quad u = u' + su''$

$\quad \overline W_l = \overline W_l' +s\overline W_l''$

$\quad \overline W_r = \overline W_r' +s\overline W_r''$

$\quad \overline W_o = \overline W_o' +s\overline W_o''$

$\quad \overline E = \overline E' -s\overline T + s^2 \overline E''$

1. $\mathcal{P}$ outputs the folded private input $(W, \mathsf{E}, r_l, r_r, r_o, r_e)$, computed as:

$\quad W = W' + sW''$

$\quad r_l = r_l' +sr_l''$

$\quad r_r = r_r' +sr_r''$

$\quad r_o = r_o' +sr_o''$

$\quad \mathsf{E} = \mathsf{E}' - s\mathsf{T} + s^2 \mathsf{E}''$

$\quad r_e = r_e' - sr_T + s^2r_e''$

The resulting public and private inputs constitute a new PLONK pair $(U, V)$ which is the folding of the two input pairs. The protocol can be made non-interactive via Fiat-Shamir.

## Security Proof

### Completeness

To show completeness, we observe that $C'_{Q,i}(\mathsf{L}, \mathsf{R}, \mathsf{O}, u, \mathsf{E})$

$= C'_{Q,i}(\mathsf{L}' + s\mathsf{L}'', \mathsf{R}' + s\mathsf{R}'', \mathsf{O}' + s\mathsf{O}'', u' + u'', \mathsf{E}' -s\mathsf{T} + s^2\mathsf{E}'')$

$= (u' + su'') \cdot [{q_L}_i \cdot (l_i' + sl_i'') + {q_R}_i \cdot (r_i' + sr_i'') + {q_O}_i \cdot (o_i' +so_i'')] + {q_M}_i \cdot (l_i' + sl_i'') (r_i' + sr_i'') + (u' + su'')^2{q_C}_i + e'_i -st_i + s^2 e''_i$

$= u'(q_{L_i} \cdot l_i' + q_{R_i} \cdot r_i' + q_{O_i} \cdot o_i' + q_{M_i} \cdot l_i' \cdot r_i' + u' \cdot q_{C_i}) + e'_i + u'' \cdot s^2(q_{L_i} \cdot l_i'' + q_{R_i} \cdot r_i'' + q_{O_i} \cdot o_i'' + q_{M_i} \cdot l_i'' \cdot r_i'' + u'' \cdot q_{C_i}) + s^2 e''_i \newline + u''(q_{L_i} \cdot l_i' + q_{R_i} \cdot r_i' + q_{O_i} \cdot oi') + u'(q_{L_i} \cdot li'' + q_{R_i} \cdot r_i'' + q_{O_i} \cdot o_i'') + q_{M_i} \cdot (l_i' \cdot r_i'' + l_i'' \cdot r_i') + 2u'u''q_{C_i} - st_i$

$= u'(q_{L_i} \cdot l_i' + q_{R_i} \cdot r_i' + q_{O_i} \cdot o_i' + q_{M_i} \cdot l_i' \cdot r_i' + u' \cdot q_{C_i}) + e'_i + u'' \cdot s^2(q_{L_i} \cdot l_i'' + q_{R_i} \cdot r_i'' + q_{O_i} \cdot o_i'' + q_{M_i} \cdot l_i'' \cdot r_i'' + u'' \cdot q_{C_i}) + s^2 e''_i$

$ = C'_{Q,i}(\mathsf{L}', \mathsf{R}', \mathsf{O}', u, \mathsf{E}') + s^2 \cdot C'_{Q,i}(\mathsf{L}'', \mathsf{R}'', \mathsf{O}'', u, \mathsf{E}'')$

So following the steps of the protocol above provides a folding that satisfies completeness.

### Soundness

The proof for soundness is rather involved, so we provide the intuition for it here, and the [technical note](https://github.com/geometryxyz/technical_notes/blob/main/sangria_folding_plonk.pdf) for Sangria provides the proof in full. We rely on the fact that a binding commitment is used.

First, we apply the forking lemma for folding ([Nova paper](https://eprint.iacr.org/2021/370.pdf) lemma 1), to obtain three transcripts. We then show that, using all three transcripts, the extractor can interpolate the values of $\mathsf{E}', r_e', \mathsf{E}'', r_e''$. It can also interpolate $(W', r_l', r_r', r_o')$ and $(W'', r_l'', r_r'', r_o'')$ using any two of the transcripts. Finally, we show that the traces these values belong to satisfy the copy constraints and the equality for each gate.

### Zero-Knowledge

The proof relies on the fact that the commitment used is hiding, then the only message sent by the prover is a hiding commitment, which means the proof is relatively short and straight forward. We define a simulator, $\mathcal{S}$, which samples a random vector $\mathsf{T}$ from $\mathbb{F}^{n + s +1}$ and a random scalar $r_T$ from $\mathbb{F}$. From these, it produces the commitment $\overline T = \mathsf{com}(\mathsf{T}, r_T)$. It then generates a random challenge $r$, and uses this $r$ and $\overline T$ to output the transcript. Since the commitment is hiding, this is computationally indistinguishable from a real transcript.