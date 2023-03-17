# Dataflow Analysis

### Identifying basic blocks

```
1. Find leaders:
  a. The first instruction of the program is a leader
  b. Any instruction that is the target of a conditional or unconditional jump is a leader.
  c. Any instruction that immediately follows a conditional or unconditional jump is a leader.
2. Then, for each leader, its basic block consists of itself and all instructions up to but not including the next leader or the end of the intermediate program
```

### Meet semi-lattice

if x ≤ y ⇒ x ∧ y = x

![semilattice example](/images/semilatticeExample.png)

⊥ ≤ T

**Top element** _T_, suchthat x ∧ T =  x for all x.

**Bottom element** ⊥, if x ∧ ⊥ = ⊥ for all x.

A meet semi-lattice is **bounded** if there exists a **top element**

### Properties of the meet operator

*  **Idempotent:** x ∧ x = x
*  **Commutative:** x ∧ y = y ∧ x
*  **Associative:** x ∧ (y ∧ z) = (x ∧ y) ∧ z

Properties of meet operator guarantee that ≤ is a partial order:
*  **Reflexive:** x ≤ x
*  **Anti-symmetric:** if x ≤ y and y ≤ x then x = y
*  **Transitive:** if x≤ y and y ≤ z then x ≤ z

If semilattice is too large you can do this:
*  define semi-lattice for 1 element
*  product of semi-lattices for all elements

so {x1,x2} ≤ {y1,y2} iff x1 ≤ y1 and x2 ≤ y2

The **height** of a lattice is the largest number of relations that will fit in a descending chain.

### Transfer functions

* Has an **identity function** ∃ f such that f(x) = x, for all x.  
* **Closed under composition:** if f1, f2 ∈ F, f1•f2 ∈ F

### Monotonicity

A framework is monotone iff x ≤ y implies f(x) ≤ f(y)

Which is the same as saying that f(x∧y) ≤ f(x) ∧ f(y)

**Warning!** Monotonicity **doesn't** mean that ~~f(x) ≤ x~~ 

### Distributive

A framework is **distributive** iff f(x ∧ y)= f(x) ∧ f(y). 

In other words doing the meet early yields the same result than doing it late.

//TODO **How to test for distributive?**

### Convergence

Given:
* ∧ and monotone framework
* Finite descending chain

**It converges**

**IDEAL solution:** ∧f<sub>pi</sub> for all possibly executed paths. But determining all possible execution paths **is undecidable.**

**Meet over all possible paths (MOP)** We assume that every edge is traversed. We do the composition for each path and then meet all the paths. 

So MOP = IDEAL ∧ Result (unexecuted paths). Therefore MOP ≤ IDEAL, smaller solution, more conservative, safe.

**Maximun fix point (MFP)**: We meet early after applying the transfer function.

| FP ≤ MFP ≤ MOP ≤ IDEAL |
|------------------------|

All the fix point solutions are safe because they are smaller than the IDEAL

If *distributive* ⇒ MOP = MFP

### About initialization

For forward, distributive and finite descending chain
If OUT[b] ⊥ for all nodes but entry:
* Safe answer (values start ≤ MFP)
* MOP solution for acyclic graphs 

If OUT[Entry] T:
* Safe answer
* May yield MOP solution

## Speed

Convergence speed dependends on the order you visit the nodes.

For _acyclic graphs_ on _forward analysis_ you want to do it using **topological sort**, where you only visit one node after having visit all it's predecessors. 

* **Pre-order**: First visit a node then it's children
* **Post-order**: First visit the children before visiting the parent.

Using **reverse post-order** you'll delay visiting the farthest node

* For some analysis cycles don't make a difference, examples:
  * Reaching definitions   
  * Live variable
  * Available expressions
* For others they do. examples:
  * Constant propagation 
  
If the cycles don't matter, and the framework is monotone, the maximum number of iterations in a data flow algorithm is _Loop depth + 2_. Where **Loop depth** is the max number of retreating edges in any acyclic path.

For other types of analysis the algorithm is guaranteed to converge after a number of rounds no greater than the **product of the height of the lattice and the number of nodes of the  flow graph**
