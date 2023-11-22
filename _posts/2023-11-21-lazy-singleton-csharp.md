---
layout: post
title: Lazy Singletons in C# - Double Check and Lock Misconceptions
image: '/blog/images/cpu.jpg'
date: 2023-11-21
author: Jason M Penniman
excerpt: In this article, I want to take a look at two misconceptions relating to the implementation of a lazy singleton in C#.
tags:
- dotnet
- C#
- design patterns
- singleton
- volatile
---
In this article, I want to take a look at two misconceptions relating to the implementation of a lazy singleton in C#
(well, .Net really).

- Double-Check and Lock for thread-safety during initialization.
- The `volatile` keyword to "fix" Double-Check and Lock.

The idea for this article came about from a conversation I had on Twitter with [Stefan
Dokic](https://twitter.com/TheCodeMan__), when discussing lazy singletons. I urge you to check out his book, [_"Design
Patterns Simplified"_](https://stefandjokic.tech/design-patterns-simplified). He uses real-word examples for using each pattern, so you'll learn how to use a factory for something besides cats and dogs.

## Singleton Pattern - A quick refresher

The Singleton Pattern is a software design pattern which says there must only be a single instance of a given object
within the process (in the case of .Net, within a single AppDomain).

A lazy singleton is initialized when it is first accessed. In contrast, an eager singleton is loaded before the first
access, usually when the process starts (or AppDomain is loaded).

## Double-Check and Lock

Countless books, blogs, and forum posts tout Double-Check and Lock as a thread-safe way of implementing lazy
singletons. In some cases this is true, but in most modern systems, it is not. In large multi-processor systems, it
definitely is not. The implementation looks like this:

```csharp
class Foo
{
    public static _instance;
    static readonly object _syncRoot = new object();

    public static Foo GetInstance()
    {
        if (_instance == null)
        {
            lock(_syncRoot)
            {
                if (_instance == null)
                {
                    _instance = new Foo();
                }
            }
        }

        return _instance;
    }
}
```

Looks safe at first glance. The instance is first checked for null. If it is null, a lock is taken to prevent other
threads from entering, and is checked for null a second time (the double check) in case another thread grabbed the lock
first and we had to wait for ours. If the instance is still null, we create a new one, otherwise we fall out of the
block and return the instance.

So, what's the problem? The CPU. Specifically, two things:

- Cache memory consistency in multi-core and multi-processor systems
- Reordering and other optimizations

### Let's look at cache first

There are three levels of cache in the CPU -- Level 1 (L1), Level 2 (L2), and Level 3 (L3). Each core gets its own L1
and L2. L3 is shared across all cores. It's important to note that this is per physical CPU. If the computer has
multiple CPUs, each CPU has its own set of caches, as these caches are on-die.

The CPU reads and writes data to its registers. That write is then, eventually, propagated to L1. Then synchronized as
well and eventually published to main memory and other processors. This is not an atomic operation by default! The cache
write happens later. In the example above, while the lock prevents other threads on other cores or other processors from
entering concurrently, there is no guarantee once inside, that the write (`_instance = new Foo()`) will be seen
immediately by other the cores and other processors. This is where atomic instructions (like those on
`System.Threading.Interlocked`) and `volatile` come into play. More on that later.

### Reordering

Even if the writes were atomic, there is nothing stopping the CPU (or even the compiler or JIT) from "optimizing"
this code and reordering the writes. If the write is reordered, the assumption of when threads see each other's writes
breaks down, because the code you wrote above may not be the order in which the CPU is actually executing those instructions
and performing those reads and writes.

## Volatile

Many articles and forum posts assert that `volatile` forces a read/write to main memory always, thereby avoiding
the any consistency issues we may see in cache and memory updates.

This is false!

Microsoft's documentation clearly states exactly what `volatile` does and does not do. It prevents the reordering of
writes, but does not change how the CPU synchronizes cache writes across its cores or other processors. It will ensure
that the data is flushed from the CPU register to cache immediately, which the CPU will eventually write to main memory.
The CPU's cache coherence mechanism ensures caches get updated across cores in the same physical CPU, but there is no
guarantee when the data will be available to other CPUs. Unless you are 100% sure that your server does not have
multiple physical processors, don't rely on `volatile` to save the day here.

## A Safe Way

So what is a safe way? Static initialization. The CLR does ensure that types only exist once in a process. So, we can
use some .Net static type initialization tricks to implement a lazy singleton:

```csharp
class Foo
{
    static class Singleton
    {
        static Singleton() {}

        internal static readonly Instance = new Foo();
    }

    public static Foo GetInstance()
    {
        return Singleton.Instance;
    }
}
```

Because statics are not initialized till they are accessed, `internal static readonly Instance = new Foo();` will not
execute till `Foo.GetInstance()` is called. Completely Lazy. Completely Thread-Safe.

A much simpler way in .Net is to just use `System.Lazy<T>`:

```csharp
class Foo
{
    static readonly Lazy<Foo> _instance = new Lazy<Foo>();

    public static Foo GetInstance()
    {
        return _instance.Value;
    }
}
```

## A Note on Virtual CPUs

Often, server-based software these days run on virtual CPUs, such as VMware or Hyper-V on-prem or a compute instance in
the cloud, or even in containers, not physical ones. Virtual CPUs (vCPU) are _not_ 1:1 with physical cores. They are CPU
time managed by the hypervisor and presented to the VM as a CPU core. This means that multiple vCPUs may not be on the
same physical die.

## Conclusion

Double-Check and Lock will always be "broken" for multi-core and multi-processor environments, especially as presented
in most texts. Yes, on a single core, single processor system (do they even make those anymore?), double-check and lock
with volatile will work just fine.

Volatile is not the fix in multi-core or multi-processor systems. It does prevent reorder and does force an immediate
flush from the register to cache to be snooped by other cores. It does not help at all in multi-processor systems.

Use `System.Lazy<T>` instead. If you look at the source code, there is a double-check and lock in there, but also a lot
of code ensuring as much atomicity as possible and that only one instance can ever be created. The class is over 500
lines of code; far more complex than the textbook double-check and lock examples.

If you don't want to take the performance hit of the locking and volatile memory boundary (especially with ARM), the
lazy static initialization is a safe way to go, and more performant when compared to `lock` and `volatile` and therefor,
more performant than `System.Lazy<T>`.

I like to play it safe and assume my apps will be deployed to a multi-processor system. Most of what I'm building is in
the cloud with no visibility into what I am actually running on. My personal choice when writing singletons? Static
Initializers, unless I can let the IoC container control lifetime.

Oh, one more thing, this holds true for x86 and ARM.

Cheers!

## References

[_The "Double-Checked Locking is Broken"
Declaration_](https://www.cs.umd.edu/~pugh/java/memoryModel/DoubleCheckedLocking.html) by Cliff Click and others.

[_Implementing the Singleton Pattern in C#_](https://csharpindepth.com/articles/Singleton) from C# In Depth by Jon
Skeet.

[Volatile](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/volatile) keyword in C#.

[_Memory barriers in ARM64_](https://kunalspathak.github.io/2020-07-25-ARM64-Memory-Barriers/) by Kunal Pathak, Microsoft.

[_Cache coherency Fundamentals
(ARM)_](https://community.arm.com/arm-community-blogs/b/architectures-and-processors-blog/posts/extended-system-coherency---part-1---cache-coherency-fundamentals)
by Neil Parris.

[_Cache Coherency_](https://inst.eecs.berkeley.edu/~cs152/sp22/lectures/L17-Coherence.pdf) by John Wawrzynek, University
of California, Berkeley
