---
layout: post
title: Finding the Number of Edges in a Complete Graph 
author: Dayo
category: Other 
---

### **METHOD 1 (GAUSS' LAW OF ADDITION)**

This method is more of a visual derivation of Gauss' Law of addition.\
Let G be a graph with N vertices and no edges. We then add edges to complete the graph. A naive approach [1] might be to connect every vertex with every other vertex. However, this results in each vertex having a degree of N - 1. The construction results in a multigraph with repeated edges. Treating G as a directed graph, we can see that each vertex shares two edges with every other vertex. Thus, to remove repeated elements in the vertex set, we simply remove every other connection. This results in a new vertex set half the size of the original. Therefore, the formula for the number of edges in a complete graph with N vertices is ***N*(N-1)/2***.

### **METHOD 2 (COMBINATORIAL MATH)***

<This method focuses purely on the set relations and the use of combinatorial mathematics. First we have our vertex set **|V|** of N vertices. Then with every vertex connecting with every other vertex no more than once, what we are effectively doing is creating a set which contains every selection of two items from your vertex set in which order does not matter i.e. {A,B} = {B,A} and so only one is chosen. This is known in mathematics as a combination and is calculated in this manner. Given the size of the collection n and the size of the selection s, a combination nCs is given as ***n!/(n-s)!s!***\
This makes sense because the complete graph is an undirected graph so order does not matter and we are posing the question "Given N vertices, in how many ways can I connect two vertices in which order does not matter?" If it doesn't make sense right now, it will at the end of this section.

#### **What is a factorial?**

Suppose we have N = 5 cards. Imagine 5 empty slots where they would go on the table. In how many ways can we place these cards such that one card occupies exactly one slot.\
The best way to answer this is as follows, you have 5 ways of placing the first card, place it anywhere. Now you have 4 slots for the next, then 3 and so on. That is what a factorial is, the number of arrangements that exists for a set of N items in which order matters and we take it as N!.

#### **What is a combination?**

Still with the same example above, take your 5 cards. You want to find all possible choices of 2 in which order does not matter.\
Another way of rephrasing the question is "In how many ways can you make a choice of 2 items from the set of 5 items in which order does not matter?" i.e. "({A,B}={B,A})". So what do you do? What we are going to do now is basically the reverse of the process above. In this case, we are choosing cards from the empty slots. We have five possible choices for the first choice then 4 for the second.\
While you might think the result will be 5x4 it is not rather it is 5x2 because we have both {A,B} and {B,A} in our set and they both mean the same thing in a combination as order does not matter so we divide by 2 to get an answer which is 5x2. This can be expressed in another way which is n!/(n-r)!r! which is basically the general version of what we are doing which is 5x4x3x2x1/ (3x2x1)(2x1) = 5x2 = 5!/(5-2)!2!

### **METHOD 3 (RECURSION)**

This method involves the use of induction. Understanding and appreciating this method involves a little imagination and is fun to think about.\
We can imagine a method of formation of the complete graph by a process of state evolution by placing additional nodes over time evolving the state from n to n+1 edges and keeping track of the newly added edges to account for these placed nodes and then at the final state summing all these added edges.\
As an example, we shall take out N=5. We first start with the base/recursive case, the complete graph of n=1 vertices. It has 0 edges. Then we add 1 vertex, increasing the vertex count to 2. The number of edges formed is 1. We then do this again, noticing that the number of edges formed of course must equal to the number of vertices excluding the added vertex for the graph to be complete. You can also imagine this applying in the generalized case when N=n. Then, the number of total edges for a completed graph with N vertices must just be a summation from 0 to N-1 which brings us to Gauss's formula. Then, our final formula from this method is N*(N-1)/2.\
I think this method and the first opens your eyes to the relation between Gauss' Law and addition.

#### **A short digression on permutations**

A permutation is a selection in which order does matter => {A, B} != {B, A} and can be expressed from the same above scenario as nPr = n!/(n-r)!= 5x4x3x2x1/ 3x2x1 = 5!/(5-2) = 5x4 which includes the duplicates omitted by the combination set.\
From the above explanations of combinatorial math, we can now see that the problem of counting the number of edges in a complete graph can be phrased in the form

"Given N vertices, in how many ways can 2 items be chosen in which order does not matter?" and we can see that for a complete graph of N vertices, N/(N-2)!2! = N(N-1)/2 edges exist.