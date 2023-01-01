---
layout: post
title: Singleton vs. Static Class
date: 2010-05-16 00:14:14.000000000 -04:00
author: Jason M Penniman
excerpt: <p>I'm often asked “Why create a Singleton when a Static Class does the same thing?”.  Well, while using a Singleton vs a Static Class may seem to be the “same thing”, they are quite different.  There are pros and cons to both, and each has it's place in an application.  To understand the difference, let's first take a look at memory allocation.  Then we'll look at memory implications, state and synchronization, and some caveats as to the use and behavior of the two.</p>
category: Development
---
I'm often asked “Why create a Singleton when a Static Class does the same thing?”.  Well, while using a Singleton vs a Static Class may seem to be the “same thing”, they are quite different.  There are pros and cons to both, and each has it's place in an application.  To understand the difference, let's first take a look at memory allocation.  Then we'll look at memory implications, state and synchronization, and some caveats as to the use and behavior of the two.

## Stack and Heap

There are two major types of memory allocation within the .Net CLR and Java VM--the stack and the heap.  The stack is just that—an organized stack of frames.  Think of it as a stack of blocks and you can only access the block at the top of the stack—the active frame.  Each thread gets it's own stack.  The heap is a disorganized “pile” of “things” that can be quickly accessed from anywhere in the pile.  Under the hood, .Net and Java handle these memory areas differently, but they are similar enough at a high level for general discussion.

In Java (Sun/IBM JVM), class data lives in PermGen (permanent generation memory)--a third memory area completely separate from the stack and heap.

In .Net, each AppDomain gets it's own stack(s) and heap.  In Java, memory is allocated per JVM instance (sort-of... this rule is bent in most EJB containers).

An excellent article on stack vs heap in .Net by Matthew Cochran can be found here:

<a href="http://www.c-sharpcorner.com/UploadFile/rmcochran/csharp_memory01122006130034PM/csharp_memory.aspx">http://www.c-sharpcorner.com/UploadFile/rmcochran/csharp_memory01122006130034PM/csharp_memory.aspx</a>

<a href="http://www.c-sharpcorner.com/UploadFile/rmcochran/csharp_memory01122006130034PM/csharp_memory.aspx"></a>

<a href="http://www.c-sharpcorner.com/UploadFile/rmcochran/csharp_memory01122006130034PM/csharp_memory.aspx"></a>


### What goes on the stack?

<ul>
<li>Instructions</li>
<li>Pointers/references</li>
<li>ValueTypes/Primitives/Structs</li>
<li>Static Variables</li>
<li>Global Variables</li>
<li>Literals</li>
<li>Constants</li>
</ul>

### What goes on the heap?

<ul>
<li>Pointers/references (I'll explain this in just a bit)</li>
<li>Static methods that don't fit on the stack</li>
<li>Object Instances</li>
</ul>

In .Net, pointers, or references, live where they were created.  So, any reference created inside an instance method of an object on the heap will live in the heap and not on the stack.


## Memory Implications
From a memory stand point, there's a difference.  The Singleton, because it's an instance, lives on the heap, always.  A static method will live on the stack if it fits, otherwise, it's allocated on the heap.  Why does that matter?  Well, academically, in an extremely high volume system, it could be a performance/scalability issue.  Remember, each thread gets its own stack.  Therefor, while class data is the same across threads, a copy of the method is loaded into each thread's stack.  While this would happen with a Singleton, it would only be each instruction as needed vs. the entire method plus the instruction frames in the case of the static method.


## State and Synchronization

Both need to synchronize access to global variables. Global member variables will be on the stack in either the static class or the singleton.

## Caveats

### Static Class:

* Loaded when the program starts (at least in .Net.  Java works similarly but depends on classpath and context... ie. Desktop app vs Web Container vs EJB Container)
* Allocated on the stack
* Is a sealed class (cannot be inherited)
* Cannot be a derived (sub) class
* Contains only static members
* Cannot contain instance constructors/initializers

### Singleton:

* Allocated on the heap
* Can be inherited
* Can be a derived (sub) class
* Can be lazily loaded

## Conclusion

It's all academic.  Given the caveats and what we know about how memory is allocated, in practicum, it really doesn't matter from a performance and scalability perspective.  Which one you use really depends on the need.  Will your “single” class need to inherit some other class?  Will it be sub-classed?  Do you require lazy or eager initialization?  In some cases, it just comes down to personal preference.

Look for my follow-up to this post, “Singleton vs Static Class: En Practicum”, for examples on where one might choose one over the other.
