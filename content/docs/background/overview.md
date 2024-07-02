# Overview

## Preliminaries

While we may add more background in future iterations, for now we punt on explaining the following concepts and assume you are already familiar with them:

1. Elliptic curve cryptography
   1. Hardness of the discrete logarithm problem and related problems
   2. Pairing-based cryptography at a "black-box" level (what it does, not necessarily how it works)
   3. Resources: [Lecture (Boneh)](https://www.youtube.com/watch?v=8WDOpzxpnTE)
2. Commitment schemes
   1. Properties: hiding and binding
   2. Construction: Pedersen Commitments
   3. Resources: [Lecture (Boneh)](https://www.youtube.com/watch?v=IkNZWJFcfcU)
3. Zero knowledge proofs (high level idea)
   1. Properities: completeness, soundness, zero-knowledge
   2. Resources: [Lecture (Goldwasser)](https://www.youtube.com/watch?v=uchjTIlPzFo)

## zk-SNARKs

Plonk is a zk-SNARK proof system. In such a system, one entity (the prover) executes a function $f$ on a set of inputs (say $x,y$) and produces output $z=f(x,y)$. It then issues a proof $\pi$ that $z$ is correct. It can show $\pi$ to anyone and it will convince them (the verifier) that $z$ is correct. 

What are the assumptions about the function, inputs, and outputs?

**Function**:

* $f$ can be nearly any function.
* $f$ must be deterministic, meaning that running $f$ on the same inputs always produces the same output.  Randomness can be used but it needs to be an input to the function.
* $f$ is disclosed to the verifier, meaning that to check $\pi$ is correct requires a copy of $f$ itself. It is possible to make a private $f$ with a hack, see below (under inputs).
* In Plonk specifically, $f$ takes the form of a circuit with two kinds of gates: addition (takes two inputs and outputs their sum) and multiplication (takes two inputs and outputs their product). It is technically modular addition and multiplication, where the modulus is a large (for example, 256 bit) prime number. It has been proven that using only these two gates is sufficient to emulate any $f$ that can be implemented in any other kind of circuit (it just might result in a larger circuit to capture the same functionality).
* To use Plonk in practice, you do not need to program circuits. You can use a special-purpose programming language to write $f$ and compilation tools will convert $f$ into the circuit format that Plonk expects. 
* $f$ has a (quite large) upper-bound on its "size" (for example, less than 1 trillion gates for efficiency in standard cryptographic settings).

**Inputs:**

* $f$ can have any number of inputs which are typically integers (modulus the large prime).
* Inputs can be disclosed to the verifier (public inputs), meaning that to check $\pi$ is correct, the verifier also checks that these were the inputs that were used to produce $z$.
* Inputs can also be undisclosed (private inputs), meaning the verifier does not see them and does need them to check that $\pi$ is correct. But what does it mean to check $z$ is correct if the inputs are not disclosed? It proves that the verifier knows some input $x$ such that, given function $f$, public input $y$ and public output $z$ that $z=f(x,y)$. This called a "proof of knowledge." The $\pi$ reveals zero information (or "zero knowledge") about $x$ (beyond what you can learn by knowing it is a legal input to $f$ that produces $y$).
* As inferable from the above example, inputs can be a mix of public and private.

**Ouputs:**

* The output $z$ is disclosed to the verifier, meaning that to check $\pi$ is correct requires a copy of $z$. 
* $z$ does need to be a single integer, it can be a data structure. 

**Proof Output:**

* The output proof $\pi$ is succinct, meaning that (for Plonk) it is always the same size regardless of how many gates $f$ has.
* Checking $\pi$ is correct is (for Plonk) constant-time, meaning it is always the same amount of time regardless of how many gates $f$ has.

So why do we need zk-SNARKs? In the case that some of the inputs are undisclosed/private, it is clear zk-SNARKs let us do something "magic," prove things about inputs that we are not disclosing. In the case that all inputs are disclosed/public (we call this a SNARK rather than zk-SNARK), note that the verifier knows all values $x,y,f,z$ in $z=f(x,y)$. If the verifier wants to know if $z$ is correct, it does not need a proof $\pi$, it can just execute $f$ for itself on $x,y$ and see that the output is $z$. 

At first glance, SNARKs (rather than zk-SNARKs) seem useless but remember that proof outputs are succinct and constant-time. For this reason, it might be cheaper (in terms of time and memory) for the verifier to check $\pi$ than to re-execute $f$. Whether it is cheaper or not basically comes down to how complex $f$ is. If $f$ is simple, re-executing it will be fastest. But once $f$ becomes sufficiently complex, it will be cheaper to verify $\pi$. What is sufficiently complex? Roughly any $f$ that would take, say, a millisecond or more to run on a normal device.

Last, it is important to recognize we are not getting something for "free" here. In order for the verifier to enjoy fast verification of $f$, someone else (the prover) had to execute $f$ and generate a proof $\pi$ for $f$ which is substantially more work than just running $f$. So the prover does extra work to save the verifier work. What are some models where this kind of trade-off makes sense for SNARKs (rather than zk-SNARKs)? There are probably many more, but the two big ones are:

* Case 1: there is a powerful (but not trusted) computer like a cloud service and a smaller constrained device (like a phone with battery life). Additionally, the economics need to make sense for the cloud to execute on behalf of the smartphone (fees, subscription, service offered by phone manufacturer, ...).
* Case 2: there are many verifiers that want to verifier the same thing so the system scales better if one party executes and issues a proof $\pi$ and the verifiers all check the proof rather than re-executing the function.

## Plonk & its gadgets

There are numerous ways to implement a zk-SNARK system. Plonk uses a templated called a polynomial interactive oracle proof (or poly-IOP). The next background article will go into more detail about it. Roughly, the prover stores inputs, outputs, gates, and intermediate values in one-dimensional arrays (or vectors). It will also perform a set of operations on the arrays to prove the circuit is executed correctly. Generally proving that array operations are done correctly will require proofs that are proportional (in size and time) to the length of the array. 

The trick to avoiding this is two-fold: first, the array will be encoded into a polynomial (univariate meaning a single variable) and every operation that is done on the array can be "mirrored" on the polynomial representation of the array. Second, the verifier will not receive the polynomials themselves but rather a short commitment to them. Again, every operation being done on the arrays and polynomials need to be mirrored on the commitments to the polynomials so the verifier can follow along.

In order to work, Plonk establishes a set of Poly-IOP building blocks (or gadgets) and combines them into a somewhat involved protocol that proves a circuit is executed correctly. Once this is done, anyone can write any function into a circuit and have Plonk's circuit gadget prove it is executed correctly.

{{< mermaid >}}
flowchart LR
Succinctness --> Poly-IOP
Poly-IOP --> Univariate
Univariate --> Gadgets


subgraph Gadgets
  zero-.->add
  rotate-.->add
  zero-.->mult
  rotate-.->mult
  mult-.->shuffle
  encode-.->circuit
  shuffle-.->circuit
  add-.->lookup
  shuffle-.->lookup
  lookup-.->circuit
  mult-.->range
  circuit
end

Gadgets --Special Purpose--- Protocol1(Function)
circuit --General Purpose--- Protocol2(Function)

classDef color fill:#9f6;
class Protocol1 color
class Protocol2 color

{{< /mermaid >}}

This gadget book is not for writing Plonk circuits. Writing Plonk circuits is one method for proving a function is correctly executed and is a general purpose method that works for any function. The second method is to realize that the building blocks of Plonk can be used directly, if they are verbose enough to capture what you want $f$ to do. It is a special purpose method, meaning it will not work for all $f$ but if it does work, it should be intuitive that this will be considerably faster for the prover. It is also possible to do a hybrid approach where certain parts of a general purpose circuit are replaced with gadgets and then glued back to the circuit.
