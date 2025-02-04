---
layout: post
title: High Performance Bioinformatics (4) - Parallel Computing
author: Dayo
category: Main  
---

Parallel computing is essential in high-performance bioinformatics, enabling faster processing of large datasets. If you're looking for a deeper dive into parallel computing and heterogenous platforms, I recommend this book:  

**Algorithms and Parallel Computing** – Fayez Gebali  

(Graphs would be helpful, but I’d rather keep this page clean. Check the book for detailed visuals.)  

---

## Types of Parallel Algorithms

Algorithms can be broadly classified based on task dependencies:  

1. **Serial algorithms** → Some tasks depend on previous tasks.  
2. **Parallel algorithms** → All tasks are independent.  
3. **Serial–parallel algorithms (SPAs)** → Groups of tasks depend on groups of other tasks.  
4. **Nonserial–parallel algorithms (NSPAs)** → Neither serial nor parallel; computation graphs are irregular and hard to dissect.  
5. **Regular iterative algorithms (RIAs)** → Often used in structured computations like matrix multiplication (somewhat out of scope here).  

---

## Understanding the Speedup Factor (S)

The primary goal of parallel computing is to increase the **speedup factor (S)** of a program. This is defined as:  

$$
S(N) = \frac{T_p(1)}{T_p(N)}
$$

where:  
- $ T_p(1) $ = Execution time on a **single** processor  
- $ T_p(N) $ = Execution time on **N** processors  

In an ideal case, **S(N) = N**, assuming perfect parallelization and no overhead (e.g., network delay, memory access bottlenecks).  

### Factors That Affect Speedup:
To maximize parallel efficiency, we need to minimize:  

- **Memory collisions** → Delays from locks when multiple processes access the same memory.  
- **Memory mismatch ratio** → Defined as $T_m / T_p$, where:  
  - $T_m$ = Time taken to fetch data from memory.  
  - $T_p$ = Time taken to process that data.  

Because of memory hierarchy effects (cache → RAM → storage → networked storage in heterogeneous systems), mismatches can **drastically** slow down computation.  

---

## Amdahl’s Law for Multiprocessing Systems  
#### [Section 1.9 , Algorithms and Parallel Computing - Fayez Gebali ]

Assume an algorithm or a task is composed of a parallelizable fraction \( f \) and a serial fraction \( 1 - f \). Assume the time needed to process this task on one single processor is given by:

$$
T_p(1) = N(1 - f)T_p + N f T_p = N T_p
$$

where the first term on the right-hand side (RHS) is the time the processor needs to process the serial part. The second term on RHS is the time the processor needs to process the parallel part. When this task is executed on \( N \) parallel processors, the time taken will be given by:

$$
T_p(N) = N(1 - f)T_p + f T_p, 
$$

where the only speedup is because the parallel part is now distributed over \( N \) processors. Amdahl’s law for speedup \( S(N) \), achieved by using \( N \) processors, is given by:

$$
S(N) = \frac{T_p(1)}{T_p(N)}
$$

$$
= \frac{N}{(1 - f)N + f}
$$

$$
= \frac{1}{(1 - f) + \frac{f}{N}}. 
$$

To get any speedup, we must have:

$$
1 - f \ll \frac{f}{N}. 
$$


**Takeaway:** Even with infinite processors, the serial portion of code limits the maximum speedup. This is why optimizing **serial bottlenecks** is just as important as parallelizing the workload.  

---

## Parallel Hardware Configurations  

(Heterogeneous computing setups can get complicated. Instead of summarizing, I’d suggest checking the book for a more in-depth breakdown.)  

---

## Concurrency Platforms  

Concurrency platforms help abstract a lot of the complexity in parallel programming. Since complete mastery of these platforms isn’t always necessary, I’ll focus on the **most useful primitives** rather than going into exhaustive detail.  

I'll cover **OpenMP** and **MPI**, which handle different types of parallel programming.  

CUDA will be discussed in a [later post](https://owolabioromidayo.github.io/).  

---

## OpenMP  

[OpenMP](https://www.openmp.org/) is a **shared-memory parallel computing** platform for C/C++. It makes writing multi-threaded code easier.  

### Real-World Uses:
- **GROMACS** (Molecular Dynamics Simulations) → [Repo](https://github.com/gromacs/gromacs)  
- **N-body simulator I wrote** → [Repo](https://github.com/owolabioromidayo/nbody_simulation)  

### Best Learning Resources:
- [An excellent overview by bisqwit](https://bisqwit.iki.fi/story/howto/openmp/)  
- [Mastering OpenMP Performance](https://www.openmp.org/wp-content/uploads/openmp-webinar-vanderPas-20210318.pdf)  
- [A “Hands-on” Introduction to OpenMP](https://www.openmp.org/wp-content/uploads/omp-hands-on-SC08.pdf)  

### Most Useful OpenMP Primitives:
- **Parallel for loops** → Great if there are no cross-dependencies. (Can be statically or dynamically scheduled.)  
- **Work sections** → Separate parallelizable and non-parallelizable work.  
- **Synchronization mechanisms** →  
  - `#pragma omp critical` → Protects sections from race conditions.  
  - `#pragma omp barrier` → Ensures all threads reach a point before proceeding.  
  - `#pragma omp atomic` → Lightweight protection for individual memory operations.  
- **Reduction operations** → Useful for aggregating results across multiple threads.  

---

## MPICH (Message Passing Interface)  

[MPICH](https://www.mpich.org/) is a **distributed computing** interface, allowing tasks to run across multiple servers. Think of it as a step up from OpenMP, where instead of parallelizing within a single machine, we're distributing work across a **cluster**.  

### Best Learning Resources:
- [MPI Tutorial](https://mpitutorial.com/tutorials/) → Excellent practical guide.  

### Most Useful MPICH Primitives:
- **Message passing** →  
  - `MPI_Send()` → Send data to a node.  
  - `MPI_Recv()` → Receive data from a node.  
- **Distributed Computing Models:**  
  - **Orchestrator-Processor Model** → A single node (often a weak but memory-heavy machine) controls multiple computing nodes and aggregates results.  
  - **Collective Communication Operations:**  
    - `MPI_Bcast()` → Broadcast data to all nodes.  
    - `MPI_Scatter()` → Distribute chunks of data to multiple nodes.  
    - `MPI_Gather()` → Collect data from each node for processing.  
    - `MPI_Reduce()` → Perform data reduction (e.g., sum, max, min) across all nodes and return the final result.  

---

## Conclusion  

Parallel computing is a powerful tool in bioinformatics, but scaling isn’t just about adding more processors. Memory bottlenecks, synchronization overhead, and serial sections all limit speedup.  

OpenMP is great for **shared-memory, thread-based** parallelism, while MPI scales better for **distributed computing** across multiple machines. 

In a later post, I’ll cover **CUDA** for **GPU-based parallelization**.
Check out **previous posts** as well!  

