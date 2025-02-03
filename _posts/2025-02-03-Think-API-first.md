---
layout: post
title: Think API First
author: Dayo
category: Main  
---


In my past couple of months programming web backends and systems projects, working a bit on open source, and reading larger codebases, I have reshaped my premature thinking about coding and programming in general.  

# The Premature Coder Mindset 

As a premature coder, the most important thing to me was **building fast and breaking things**. My entire development process revolved around:  
- Gathering as much information about how a software product should be implemented.  
- Building it in the way that came to mind first, with **utmost velocity** and without much code review.  

However, when working on larger projects, the pitfalls of this approach became glaringly obvious—especially in a team setting. I began to respect and understand:  
- The need for **standardized software development practices**  
- The importance of **unit testing and regression testing**  
- The value of **simplifying the useful surface area of code**, rather than just writing more lines for the sake of it.  

A high line count is a **useless metric** when not accompanied by clarity and maintainability.  

# API-First Thinking  

I have now learned that perhaps the best way to design and write software is to **first think deeply about its interfaces**—how other code, systems, and users will interact with it—and simplify that interface as much as possible.  

This is **what OOP is meant to be**, but in professional settings, it often deviates from that principle.
In fact, when I think about **good API design**, I picture Linux kernel code in C—minimal, efficient, and built with clear purpose—at least from an outsider's perspective.

# Why API-First Thinking Matters  

## 1.  The importance of unit and contract testing


<p>
    This API driven developement really lends itself easily to unit testing and TDD, as writing the tests for these interfaces can 
    further build our understanding of how our implementation should actually be. In the process of writing these tests, we might discover
    new things about the nature of the system as well
</p>
<p>
    Contract testing ensures that our API behaves exactly as specified to our end-users.
</p>

### Error handling is not an afterthought 
<p>
When API design and interaction come first, we learn to think of all the failure modes of the system and guard and test against that in whatever contract we have. 
</p>


## 2. The debt of adding new features / the importance of simplicity
<p>
    When we realize that we can simplify the scope of software requirements into quality software for end-users, we realize just how much
    messy code bloats the system with unnecessary features, increasing complexity without adding real value.
</p>
<p>
    This is why simplicity is and should be at the heart of real software engineering, and these values are typically upheld in open source or performance heavy projects , where code-readability drives adoption, and simplicity of code usually translates to better performance (at least at a high level).  

</p>

<p>

Simplicity is elegance. Simplicity will make you appreciate your code when you come back to it after a long period.
<br/> <br/>
Because what are you even engineering? Software engineering at a high level is an art form. You want to express program objectvies or capabilities in a clear format while hitting all objectives, not produce something bloated and buggy and artless. You need to learn to tame and cull the code, to remove irrelevant bits, to write something truly elegant.
True engineering is learning to prune and simplify things, while still improving on whatever metrics are being tracked. 
<br/> <br/>
I don’t understand why I had to write a shit-ton of bad code before my brain suddenly opened up to why I had no style.
Refactoring and code-review sessions are some of the best ways to retrain ourselves.
Writing a ton of LLM-assisted code does not grant one style or understanding. LOC is useless when you have no sense of what you are doing.
</p>


## 3. The importance of regression testing
<p>
    Software is never finished. This is well known. Even when software is developed with a zen-like approach, the world evolves. Competitive forces emerge,
    they implement new features, users demand more, support breaks, dependencies go out of favour, or update. Everything is constantly changing, and breaking.
    As the codebase expands to support these secondary demands, the core API we designed must still function as expected. This is why regression testing is so important, to ensure that these changes don't affect that performance. 
</p>

## 4. The importance of writing code methodically
<p>
    By writing code in this well thought out manner rather than just hammering out lines to push some feature, only to merge subsequent fixes
    in the following days over hurried code; thinking and implementing in a slow and methodical manner helps you know when a certain piece of work is
    actually finished.
</p>

There are generally two ways to approach a problem. The first approach involves diving headfirst into the problem and learning about the
problem in some messy manner while trying to solve it, while the second approach involves pausing and accumulating all relevant information about
the problem before **"one-shotting"** a solution to it. 

The rational mind knows that the second approach might turn out better in the end, especially if the first approach does not engage in
successive refinement of the messy solution, and still keeps discovering new things about the problem even after having some appaarent solution. 

But that same mind should also be aware that most times gathering information about a problem domain is not straightforward, especially if the
domain is not formal or extensively researched. This is why the build fast break things methodology is still prevalent in software engineering, 
because experimenting can be a rapid way to learn how **not** to do things.

( A side concept supporting the first approach is that of the inattentive and dopamine-driven developer. I myself fall into the pitfall
of being a dopamine-driven developer, straddling the dangerous path of firing off code that has not been fully well thought out, resulting in
me being disappointed at its quality, thinking it to be a sloppy fool's work, irrespective of its functionality ).


<p>
    Build fast and break things, yes, but do it with API oriented design in place. <br/> <br/> There are 2 monsters in you, the chaos monster who is impatient and wants to see how the system is like, and the calm production guru who just wants everything to be alright.
</p>


    
## 5. System design
<p>
    This theory of APIs and thinking about requirements and external interactions is really well suited for system design reasoning.
    This is the precursor to systems-design-level thinking, which is the higher level form of this, dealing with simplified interaction between
    different systems, focusing on scalability and reliability of different services, while simplifying as much as possible.
</p>


## 6. Saves a lot of developer hours  
<p>
It is more than self-explanatory that programming in this approach releases a lot of burden from the mind and allows one to plan accordingly, peacefully implementing fully tested features piecewise, instead of panicking and shipping and iteratively fixing to no end. 
</p>



# Conclusion

At the end of the day, writing software is about more than just making things work—it’s about making things work well. The more I code, the more I realize that software isn’t just about hacking things together, but about designing something that lasts. Thinking API-first forces you to slow down, refine your approach, and build with clarity and intent rather than just brute force.

I’ve written a lot of bad code to get here, and I’ll probably write more bad code in the future. But at least now, I know what I’m aiming for—something simpler, something cleaner, something that actually makes sense when I come back to it months later.


## How I plan to improve in this manner of thinking

### 1. Learn from open source.
<p>
    Contribute to open source code. Walk through their code, documentation, and design decisions.
    <br/>
   
</p>
 Read through [The Architecture of Open-Source Applications](https://aosabook.org/en/)


### 2. Improve algorithmic design skills and system design skills

<p>
Algorithmic design, systems design, systems engineering and performance engineering can all be seen through the lens of API design.
What makes or break a good implementation is how strictly they conform to some API or algorithmic specification, with minimal overhead.
</p>

### 3. Try functional programming
<p>
Some lessons can be learned from functional programming, where functions themselves are first-class citizens, not just the APIs built around them.
</p>












