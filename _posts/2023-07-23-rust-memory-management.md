---
layout: post
title: How memory management works in Rust
---

## The Pitfalls of Dynamic Memory Management
Managing dynamic (heap) memory has notoriously been a difficult programming problem. 
Programming languages over the years have taken different approaches to solving this problem. 
In C you must explicitly allocate and free all dynamic memory yourself, 
meanwhile in Java a garbage collector runs in the background which cleans 
up heap memory that is no longer in use.
- The problem with C's approach is that it requires an exact pairing of allocate/free calls,
which is not always trivial to ensure. This is often the cause of many bugs such as memory leaks, 
which can significantly detriment performance
- The problem with Java's approach is simply that it adds runtime overhead to the program, which
we would like to avoid doing
- How can we not impact runtime performance while also ensuring memory safety? A seemingly impossible
task is accomplished in Rust through a third type of memory management system: ownership.
- Before we talk about ownership it will be useful to briefly discuss the difference
between a stack and a heap, and why we need them.

## Stack and Heap
- In programs, many of the variables that we declare and assign values to, can be known 
statically at compile time, primitive types like ints, floats, booleans. These variables we call
automatic variables, because they are automatically cleaned up after the function returns.
- However, very often we have variables whose values cannot be known statically, and change
over the lifetime of a program.
