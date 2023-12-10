---
layout: post
title: Why Does fib(40) Elude My Sh*tty Interpreter??  
author: Dayo
---


Some time ago, I followed Part II of [Crafting Interpreters](https://craftinginterpreters.com/) by Rob Nystrom and implemented a tree walk interpreter in C++ [link](https://github.com/owolabioromidayo/cpplox). I didn’t put any effort into performance or memory management as I just wanted it to work, which it did for all of the tests. 

Except one. 

During the introduction to Part III of the book, which involves writing a bytecode virtual machine, Rob uses the example of fib(40) to show how a tree-walk interpreter is fundamentally the wrong design to tackle this sort of problem. And I understand that. But what I didn’t understand was how his Java implementation took 72 seconds to run (took *79.4 seconds* on my system) and my C++ implementation couldn’t even execute it because it kept getting killed by the OS.


### > A Quick Note

The interpreter pipeline is staged as **Scanner -> Parser -> Resolver -> Interpreter **. Only the interpreter was having performance issues.

### > Yes, some things come to mind

Before I go down this rabbit hole, I know that these are simple and reasonable solutions.

1. Just write an iterative solution, dude! **That's no fun.** 
2. Write the bytecode compiler part of the project.
I already tested this. clox performs this task in *10 seconds*.
How? Mark and sweep garbage collection. And flattening out the recursive calls. 

3. **Memory pooling:**  Reimplementing my own allocator to allow for chunk-based alloc/dealloc. Won't work for our pain point (recursive calls) due to its stack-based nature. For a tree like fibonacci it might help after we've gone through the worst of it (fib (39)). It seems a stupid solution because I would've just memoized at that point.



### > Of course, in pure C++ it computes.

<figure>
  <img src="/images/fib/purecppcode.png" alt="my alt text"/>
</figure>

<br /> 

<figure>
  <img src="/images/fib/purecppoutput.png" alt="my alt text"/>
</figure>

The real difference is the overhead of execution and the hidden runtime nature of the program from the compiler.

<br /> 


### > Investigating the problem

### > Why am I interested?

 This is probably my first time experiencing a notable perf limitation. Not just low FPS or slow progression, but actual refusal to compute. The only other thing like this I have experienced is Python's max recursion depth.

Fundamentally, I knew it was a memory error, because why else would it get killed in that manner? If it was just a CPU performance issue, it would have eventually computed. If it was an implementation bug, I would have gotten a result. The OS was killing my process because it was hogging resources.

### > My Ignorance
Of course, the first thing I did was find out just how badly my program was performing memory-wise. So I fired up Valgrind and did a leak check. It was abysmal, to say the least. But I was expecting it. To cut it short, I had neglected the use of smart pointers throughout and went with manual memory management while avoiding freeing any memory at all.

There were reasons—though not entirely justifiable, they were reasons nonetheless. The first had to do with my code structure. I was using derived classes throughout my implementation for my expressions and statements. This was to allow a unified manner of access for my ExprVisitor and StmtVisitor classes. Well, C++ does not like this too much. I was fine using dynamic casts explicitly when I knew what the derived class was i.e.



<figure>
  <img src="/images/fib/dyncast.png" alt="my alt text"/>
</figure>

I had a lot of virtual classes littered through my code. And at the time, when I tried to create a shared pointer for a virtual class, like so: 

```std::make_shared<Base>() ```

It wasn't allowed because you cannot instantiate an abstract object. But recently I found that there might be a way around this : 

```std::shared_ptr<Base>()```

I looked at some other implementations and the ones I saw made me believe I would have to scatter ```std::any``` everywhere to make it work. Which I strongly didn't want. 

But this was not true.

I got a clean C++ implementation of the lox tree-walk interpreter to test alongside [here](https://github.com/zanders3/lox).


### > Trying a proper implementation

I thought the issue was purely with my horrible implementation. I found a well-written C++ implementation of the lox interpreter and gave it fib(40) to chew on. But alas, it did not work either


### > Finding out why

Well, this made me neglect any hope of improving my C++ implementation, as I was more drawn to why this was happening. Was Java better than C++? I couldn’t bring myself to believe it. I had originally thought implementing the lox interpreter in C++ would give me better performance. I had been too naive.

I started investigating why this was happening.

### > Java vs C++

The two important differences between Java and C++ in this situation would be the JIT and GC. Since C++ uses Ahead-of-Time Compilation, and the only code presented to it doesn't hint anything about the nature of the programs that will be executed, it cannot outperform Java's Just In Time compilation system, which selectively compiles hot functions during execution. (more detail)

The second important difference between Java and C++ would be garbage collection. In short, Java uses a mark and sweep algorithm for garbage collection during GC pauses and this fares well for mass alloc/dealloc of small objects while C++ requires manual management and temporary allocs/deallocs, especially on the heap are expensive, even more so for hundreds of objects. 

While this is all nice and good, I tried to find more information on how exactly this JIT magic works for any JVM, and while there are some open-source JVMs, I didn't go too deep. I read some books hoping to gain more insight into the JIT's workings, but I didn't find what I was looking for. I learnt about the tradeoffs between running on the VM and compilation, hotspots and running JIT in tandem, but it wasn't enough. Still magic to me.

### > The real culprit (temp allocs/deallocs)



<figure>
  <img src="/images/fib/allocs.png" alt="my alt text"/>
  <figcaption>Heaptrack memory perf for 3 implementations </figcaption>
</figure>

Dumping the clox, jlox and my c++ implementation into [Heaptrack](https://github.com/KDE/Heaptrack), I saw that the number of allocations made by the program for fibonacci(30) was orders of magnitude higher than for jlox and clox.


<figure>
  <img src="/images/fib/Heaptrack_ui_cpp.png" alt="my alt text"/>
  <figcaption>Heaptrack C++ impl UI view </figcaption>
</figure>

Looking inside the UI for more information, this is what the path of memory usage looks like. The bulk of new memory creation obviously comes from the recursive hot path. No single function can really be pinned as the source of
all memory problems (from what I'm seeing), as the call chain is extremely long.

<figure>
  <img src="/images/fib/clean_allocs.png" alt="my alt text"/>
  <figcaption>Heaptrack  on fib(20) and fib(30) respectively for clean c++ impl</figcaption>
</figure>

This was also the case for the clean C++ implementation I found with much fewer leaks than my C++ implementation but still orders of magnitude higher allocations than the jlox/clox binaries. It couldn't even run fib(35). 


### > Trying Rust

Well, I was interested in how the new favourite child of systems programming (Rust) would perform. Maybe they were trying to hide their tracks but every reasonably completed implementation I saw did not stop at the interpreter. So I simply found a good enough one [link](https://github.com/jeschkies/lox-rs) and checked out the last interpreter branch.  And surprise!


<figure>
  <img src="/images/fib/test-rs.png" alt="my alt text"/>
  <figcaption>Testing lox-rs</figcaption>
</figure>
<figure>
  <img src="/images/fib/top-rs.png" alt="my alt text"/>
  <figcaption>Me waiting for fib(40) to compute</figcaption>
</figure>

The ```ClockCallable``` of the implementation was broken but that was fine. It computed fib(30) in less than 30 seconds. I almost gave up on fib(40) being computed but it finished after **12 minutes**.  
To be fair I was expecting this a bit, as both Rust and C++ were AOT compiled and had the same GC methods (smart pointers vs lifetimes). But it was fun to experience.


### > More analysis 

 I got into C++ because I wanted to know more about performance computing and limitations, and this was the perfect opportunity to try all the tools I hadn't before. So I fired up Heaptrack and FlameGraph for my shitty C++ interpreter, jlox and clox, to compare heap memory usage and performance. But in all honesty, I just wanted to see some nice visuals.


<h5> FlameGraphs </h5>

I decide to compare the FlameGraphs of my cpp implementation to the java implementation to learn more about the nature of the problem.


<figure>
  <img src="/images/fib/flamecpp1.png" alt="my alt text"/>
  <figcaption>C++ FlameGraph</figcaption>
</figure>

<figure>
  <img src="/images/fib/flamecpp2.png" alt="my alt text"/>
  <figcaption>C++ FlameGraph Zoomed In</figcaption>
</figure>

<figure>
  <img src="/images/fib/flamejava.png" alt="my alt text"/>
  <figcaption>Java FlameGraph</figcaption>
</figure>

<figure>
  <img src="/images/fib/flamejava2.png" alt="my alt text"/>
  <figcaption>Java FlameGraph Zoomed In</figcaption>
</figure>

I couldn't really make sense of the stark differences between the two. From C++ I could easily see the recursive function calls and tree-walking
as a result of the fibonacci calls, but for java it was completely flat and the labels weren't helping either. What I did find to be weird were the 
extremely thin graph stacks externing upwards and coming downwards immediately. Yet the only information waiting there for me wasn't at all helpful

It is important to note my peak heap memory usage for my C++ implementation was 117MB on fib(25) while the clean C++ implementation got 99.6kB on fib(30). Yet both still managed to fail on fib(40).


### > Trying to not get killed

<h5> 1. Aggressive compilation flags </h5>

![temp](/images/fib/02flag.png)

Just using the ```-O2``` and ```-finline-functions ``` compilation flags  gave me 3x performance on ```fib(30)```. Didn't solve the main problem.



<h5> 2. Profile guided optimization </h5>

The JIT compromise for AOT compilation would be profile guided optimization. I did try this, using 
```g++ -O3 -fprofile-use src/main.cpp -o cpp_prof.out``` to generate the executable for profiling, which I then ran with the preferred scenario ```./cpp_prof.out src/a.lox``` which generated a gcda file.
The final executable was then generated ```g++ -O3 -fprofile-use src/main.cpp -o cpp_prof.out``` and it knew to use the corresponding gcda file for the executable name.

While this worked for optimizing the specific example I gave it which was fib(30) from *21 secs* to *13 secs* , it still failed on fib(40). 

<br />

<h5> 3. Changing allocators </h5>
![temp](/images/fib/tcmalloc.png)

I decided to try out TCmalloc, mainly for the sake of trying out another allocator. After all, TCMalloc was created with the aim of faster multi-threaded memory management, which wasn't the issue at hand. Needless to say, it failed.




### > Measuring stack usage

<figure>
  <img src="/images/fib/rlimits.png" alt="my alt text"/>
</figure>

I was able to change the program stack size using rlimit. I changed the soft limit to a whopping 600MB but it didn't affect my results.


I tried out some stack analysis tools, using data I got from the ``` -fstack-usage -fdump-ipa-cgraph``` compilation flags and this (tool)[https://github.com/sharkfox/stack-usage]. 

<figure>
  <img src="/images/fib/stack_analysis.png" alt="my alt text"/>
</figure>

THe data I got back showed that over 99% of my function calls were not using any stack memory, and the maximum stack size of my program was 784 bytes. 


### > Trying cppcheck / static analysis

<figure>
  <img src="/images/fib/cppcheck.png" alt="my alt text"/>
</figure>

All cppcheck gives me is this, which is not at all useful. The parsing stage is not where the program is failing.


### > Conclusion

I don't believe I've gotten to the root of the problem. There are still a lot of questions I need to investigate further to understand why this is happening. It might be a (seemingly) pointless issue to waste time on since
there are already obvious solutions glaring at me. But my mind cannot reconcile yet. I am at least glad I got to try out all these tools and toys as a C++ performance noob.

I should clean up my C++ implementation and try profiling the effects of memory pooling. I believe memory pooling will have an effect on heap allocations and might be partially suited to the tree-like nature of the
fibonacci problem. This would make it easier to find the size of the pool and realloc when necessary.

Hoping to dig deeper into this.


### > Questions

I would appreciate any answers for the following questions  

1. Why do you think this is happening?
2. Where do you think the root of this issue is from (stack, heap, ...) and why?
3. Do you know of any profiling tools I can use for programs that crash like this (excluding GDB) ?

I would also appreciate any interesting commentary. Thank you!


### > Footnotes (Screenshots)

1. ![something](/images/fib/constexpr.png)
1. ![something](/images/fib/jit_footnote.png)



