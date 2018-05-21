---
layout: post
title: "Threading in .NET"
excerpt: "Basic level threads, synchronization etc."
date: 2018-05-18
tags: [tech, database, neo4j, graph, graphdatabase, cypher, query]
categories: articles
image:
  feature: posts/neo4j/graph_1.jpg
  credit: neo4j
  creditlink: https://neo4j.com/blog/other-graph-database-technologies/
comments: true
share: true
published: false
---

**Note:** This article is currently incomplete & in-progress. It'll be updated soon.
{: .notice--danger}

# What are threads

* Basic unit of execution
* Has it's own memory and alloted processor time
* Sequence of instructions managed independently by scheduler
* Thread schedulers are part of OS
* A process (executing program) are run on one or more threads - which executes independently, but shares resources within the process

* CLR delegates the thread scheduling to the OS
* It gives all threads it's own stack (threads get own copy of all their local variables) - but static variables are not, they are shared across the whole application
* Threads from same process also share the same heap memory, while different processes are completely isolated

Multi-threading

OS does timeslicing and allocates CPU time to each thread in turn
The slice time (on Windows) is generally close to 10 milli seconds
The overhead of this context switching is in the order of micro seconds
When a thread is suspended from execution (e.g. due to timeslicing), it's called _"preempteed"_ thread

## `lock(lockObject)`

```cs
lock(lockObject) {
    //dowork
}

//translets to (C# 4, .NET 4.0 and above)
bool acquiredLock = false;
try {
    Monitor.Enter(lockObject, ref acquiredLock);
    //on Enter, acquiredLock is automatically marked true
    //dowork
}
finally {
    if (acquiredLock)     {
        Monitor.Exit(lockObject);
    }
}
//Monitor.Wait() releases the lock to let other threads lock the object
//and waits to get the lock back**
```

The [Monitor](https://msdn.microsoft.com/en-us/library/system.threading.monitor.aspx#Lock) class cannot be created, it has only static methods & can be called from any context. It can associate itself with object(s) on demand (i.e. on `Monitor.Enter()` or `Monitor.TryEnter()` or `lock()`). It does not work for value types. For each lock object, it keeps track of

* A reference to the thread that currently holds the lock
* A reference to a ready queue, which contains the threads that are ready to obtain the lock
* A reference to a waiting queue, which contains the threads that are waiting for notification of a change in the state of the locked object

It's suggested to use a `private static` object to lock, as

1. If the object is accessible outside, someone else might acquire a lock on the same putting current thread to sleep, or create dead-lock. Same might happen to a string instance.
1. If the object is not static, all instances of the code (e.g. different threads calling it) will not be synchronized

## Thread overheads

* Creating a new thread takes about a few hundred milliseconds (roughly), for the new stack and spawning etc.
* Also uses around 1MB of memory per thread ([default](https://msdn.microsoft.com/en-us/library/windows/desktop/ms686774%28v=vs.85%29.aspx?f=255&MSPPError=-2147217396) in Windows)

* Thredpool - keeps a pool of background threads
* All threads are background (stops abruptly if main thread terminates), and there is a maximum limit
* New work is allocated to idle threads, and are returned to the pool once done
* If there is more work than maximum thread limit, they are queued
* Use it via a `Task` or `ThreadPool.QueueUserWorkItem`

**Note:** Threads do not throw exceptions across other threads. To handle exception, it has to be caught on the same context.
{. notice--warning}

## Tasks

* Designed to return value

# Thread synchronization

* Blocking - blocking threads do not consume CPU, but consume memory
  * Sleep
  * Join
  * Task.Wait
* Locks - limits number of threads that can enter/access a piece of code
  * Exclusive locks - allows only one thread
    * `lock()`, `Monitor.Enter()` - `Monitor.Exit()`
    * Mutex
  * Non-exclusive locks - allows a limited number of threads to access code
    * Semaphore
    * SemaphoreSlim
    * Reader/Writer lock
* Signals - allows thread to pause and wait until a notification is received from another thread
  * Event wait handles
  * Monitor's wait/pulse
  * CountdownEvent, Barrier
* Nonblocking construcuts - protects access to a common field
  * Thread.MemoryBarrier
  * Thread.VolatileRead
  * Thread.VolatileWrite
  * Volatile
  * Interlocked