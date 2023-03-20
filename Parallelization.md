## Parallelization

This is multiprocess parallelization. Not the instruction level parallelism from before.

### DoAll loops

```
for (i = 11 to 20) 
  a[i] = a[i-10} + 3
```
This is a do all too! because you're reading from 1 to 10 and writing from 11 to 20, no dependence.

Does there exists a data dependence edge beteween 2 different iterations? 
* A data dependence edge is _loop-carried_ if it crosses iterations boundaries
* DoAll Loops = loops without loop-carried dependences

Recall data dependences from [Instruction Scheduling](InstructionScheduling.md)

### Affine Array Accesses

When array indexes are affine expressions of surrounding loop indexes. Meaning they are linear expressions of indexes + constants. Example A[2*i+1].

This is the domain we'll focus on. If we introduce `read(n)` for example, we have no way of knowing what _n_ is at runtime.

In fact we only look at indexes with affine relations and ignore the other ones (for example if I have something like: A[i*i]). We ignore them and we assume they may refer to the same location.

When analizing a multi dimensional index all the indexes have to match for it to be a dependence. Therefore if you need to ignore one of the indexes you can still make the safe assumption that it matches and work with the others to see if the others don't.

### Data dependences analysis

Be sure to also check for output dependence! In such cases you add the constrait that i ≠ i'

![Dependence Analysis Example](/images/depAnalysisExample.png)

![Dependence Analysis](/images/depAnalysis.png)

### Parallelizable without Loop Transformation
**1-d parallelizable**: is it ok to assign each outer loop to one single process? Any data dependency between these processes? (different outer array index can access the same array element?)

**2-d Parallelizable**: is it ok to assign each inner loop to one single process? (with same outer loop array index, but different inter loop array index, can they access the same array element?)

### Loop Transformation

A loop transform is legal if all data dependences in the original loop nest are honored with sequential execution in the new loop.

### Relaxing dependencies

![Relaxing dependences](/images/relaxingDependences.png)

To solve the scalar privatization each processor has to have it's own _t_. So no processor would be writing in another one's variable. The dataflow analysis that we need to figure out if it's privatized is an **Reaching def analysis**. The variable t is not live at the end of the iteration.

Reduction operations are parallelizable to. Think Map-Reduce

### Amdahl's Law

> "Suppose you have 10% of the program that is not parallelisable you can only speed up the program by 90%"

Hence we need to parallelise at the coarsest granularity (outer loops) minimizing communication. Or else if you found that the bulk of the computation is in a small core, you fully optimize that core.

:pencil2: //TODO understand intraprocedural parallelization

## Loop transformations for Parallelism and Locality

Parallelism is NOT enough. We also need to improve locality: Operations using the same data are executed on the same processor. 

Locality is _also important in sequential performance_ where operations using the same data are executed close in time, that way you take advantage of the cache.

### Loop permutation

![Loop permutation](/images/loopPermutation.png)

From inner loop parallel code to outer loop parallel code without communication

### Loop fusion

![Loop fusion](/images/loopFusion.png)

### Loop transformations

Unimodular on loop nests:
* Permutation
* Skewing
* Reversal

Cross statement transforms
* Loop fusion
* Loop fission
* Re-indexing

You can combine all of the above.

## Maximum Parallelism & No Communication

![Max parallelism No communication](/images/maxParallelismNoCommunication.png)

**Rank of partitioning = Degree of parallelism**

The constant term is used to move the indexes sideways to make them match. This is used in the cross statement transforms

### Code generation

**Naive:** Each processor visits all the iterations and executes only if it owns that iteration.
**Optimization:** Removes unnecessary looping and condition evaluation.

### Advanced topic: Pipelining

SOR = Successive over-relaxation.

![SOR](/images/SOR.png)

Using affine function will tell us send everything to the same processor.

Since we can order the execution either horizontally or vertically we know that we can advanced as a wave-front.

Now we try to find the legal way of executing the code in different time stages.

![SOR Pipelining](/images/SORPipelining.png)

When you have a choice in time mapping you have (pipelined) parallelism

If you give me a Rank(C) answer then there are Rank(C) - 1 degrees of parallelism
