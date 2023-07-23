---
layout: post
title: Dynamic Memory Management in Rust
---

Managing dynamic (heap) memory has notoriously been a difficult programming problem. 
Programming languages over the years have taken different approaches to solving this problem. 
In C you must explicitly allocate and free all dynamic memory yourself, 
meanwhile in Java a garbage collector runs in the background which cleans 
up heap memory that is no longer in use.

The problem with C's approach is that it requires an exact pairing of allocate/free calls,
which is not always trivial to ensure. This is often the cause of many bugs such as memory leaks, 
which can significantly detriment performance. Explicit memory allocation also makes programming 
less ergonomic and straight-forward.

The problem with Java's approach is simply that it adds runtime overhead to the program, which
also effects performance.

How can we not impact runtime performance while also ensuring memory safety? A seemingly impossible
task is accomplished in Rust through a third type of memory management system: ownership.

Before we talk about ownership it will be useful to briefly discuss the difference
between a stack and a heap, and motivate the intuition as to why we need them.

## Stack and Heap
In programs, some of the variables that we declare and assign values to are placed on the stack.
These include any local primitive variables (ints, floats, bools, pointers), function arguments
(before you call a function you must push the arguments to the stack), and return values. 

Placing variables on the stack is preferable since they are contiguous in memory and require 
less jumping in memory for the CPU.

Why don't we place all variables on the stack since it is so fast and neatly laid out in memory? 
A few problems with that..

The first problem is that some variables need to exist longer than a function call or be shared 
amongst multiple functions. So they need a bigger scope and/or a longer lifetime.

The second problem is that some variables shrink and grow often as a program goes on and hence 
cannot be placed in a contiguous area of memory reliably.

The third problem is that some variables are huge.. think of an array of 1M+ strings. Say you want
to perform multiple preprocessing steps on these strings. It would be neat to organize these
different preprocessing steps as separate functions. However it would be incredibly prohibitive 
to have to pass this huge array as an argument to these functions and then return it back. 
It would result in an incessant amount of data copying, which is unnecessary.

A solution to the first problem is to use a global variable, so a variable declared in the 
global scope, which is stored in another area of memory called the data region. 
Therefore the variable has a global scope and lasts for the entire duration of the program.

This could also be a solution to the third problem if the big variable is static.

However if we need a variable that shrinks/grows overtime then this solution will not work.
The reason it will not work is that the value of all variables stored in the data region 
need to be known at compile time. Once the program starts running, the data region becomes
read-only, and hence cannot be shrunk/grown.

A solution to all three problems is to use heap memory, which is an area of memory whose allocation
mechanisms can be controlled by the programmer or memory management system of the compiler. Unlike
the stack whose memory allocation scheme is straight-forward and non-programmable. Hence, think of 
the heap as a stack with programmable memory allocation. The hard part is figuring out exactly
how to program the memory allocation scheme to maximize ergonomics and minimize runtime overhead.

C says fuck you, hands you malloc() and free() as axioms, and has you figure it out yourself.

Java/Python are nice and have a garbage collector clean up any heap memory once it is no longer in 
use, therefore not requiring the programmer to allocate or free memory explicitly.

Rust uses a different approach called ownership..

# Ownership
Ownership involves a set of rules that must hold before a program can even compile.
