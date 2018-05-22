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
* Tasks are run on the managed ThreadPool, and _generally_ run as background thread
* Almost always use a `Task` over a `Thread`. If you need a dedicated thread for a long running task, still prefer using a Task with `TaskCreationOptions.LongRunning`, for the sake of consistency. Pure threads might be used for very low-level uses, e.g. custom schedulers etc.
* See the [Synchronous to asynchronous in .NET](/articles/sync-to-async-in-dotnet/) post for more details on Tasks.

# Thread synchronization

In a multi-threaded environment, multiple threads can can speed up works and keep main thread responsive (e.g. UI thread in Windows applications). Insuch scenarios, they can also access different resources like files, network connections, memory etc as per the application needs. If done incorrectly, what may happen is multiple threads trying to use and/or update same resource at the same time, unaware of each other. This can result in unpredictable and incocnsistent results.

So, in multi-hreaded applications, the threads needs to be synchronized, so that they do not work on (most importantly, update) the same resource at the same time. There is [several ways to synchronize thread](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/threading/thread-synchronization) in .NET environment.

1. Blocking - blocking threads do not consume CPU, but consume memory
  1. Sleep - sleep for specified milliseconds
  2. Join - wait for thread to compplete work
  3. Task.Wait - wait for task (generally a background thread from threadpool)
2. Locks - limits number of threads that can enter/access a piece of code
  1. Exclusive locks - allows only one thread
    1. `lock()`, `Monitor.Enter()` - `Monitor.Exit()`
    2. Mutex
  2. Non-exclusive locks - allows a limited number of threads to access code
    1. Semaphore
    2. SemaphoreSlim
    3. Reader/Writer lock
3. Signals - allows thread to pause and wait until a notification is received from another thread
  1. Event wait handles
  2. Monitor's wait/pulse
  3. CountdownEvent, Barrier
4. Nonblocking construcuts - protects access to a common field
  1. Thread.MemoryBarrier
  2. Thread.VolatileRead
  3. Thread.VolatileWrite
  4. Volatile
  5. Interlocked

## 2.1.1 `lock(lockObject)`

```cs
private static readonly object lockObject = new object();
lock(lockObject) {
    //do the actual work
}

//translets to (C# 4, .NET 4.0 and above)
bool acquiredLock = false;
try {
    Monitor.Enter(lockObject, ref acquiredLock);
    //on Enter, acquiredLock is automatically marked true
    //do the actual work
}
finally {
    if (acquiredLock)     {
        Monitor.Exit(lockObject);
    }
}
//Monitor.Wait() releases the lock to let other threads lock the object
//and waits to get the lock back**
```

The [Monitor](https://msdn.microsoft.com/en-us/library/system.threading.monitor.aspx#Lock) class cannot be created, it has only static methods & can be called from any context. It can associate itself with object(s) on demand (i.e. on `Monitor.Enter()` or `Monitor.TryEnter()` or `lock()`). For each lock object, it keeps track of

* A reference to the thread that currently holds the lock (through the object)
* A reference to a ready queue, which contains the threads that are ready to obtain the lock
* A reference to a waiting queue, which contains the threads that are waiting for notification of a change in the state of the locked object

It's suggested to use a `private readonly static` object to lock, as

1. If the object is accessible outside, someone else might acquire a lock on the same putting current thread to sleep, or create dead-lock. Same might happen to a string instance. So, `private`
2. If the object is not static, all instances of the code (e.g. different threads calling it) will not be synchronized, so `static`
3. At the time of locking, the object should not be null. A `null` object cannot be locked, see below. So, initiate it once and make sure it is not null. So, `readonly`

Locking does not work on value types. On `lock`/`Monitor.Enter`, the instance of the lock object gets a `Sync Block` structure added to it's managed memory block (sync block can also get initialized for other reasons, e.g. to store results of GetHashCode() method), and the `ManagedThreadId` of the current thread holding the lock is saved. This is used by the `Monitor` for synchronization. For the same reason, a `null` object cannot be locked.

## 2.1.1 `Mutex`

`Mutex` (from _mutually exclusive_) is similar to `Monitor` in that, it allows only one thread to execute a block of code, but unlike Monitor, it can work across processes. Monitor is more of a .NET construct, whereas Mutex is a wrapper over OS level Win32 construct. Generally a string name is used to identify a [Mutex](https://docs.microsoft.com/en-us/dotnet/api/system.threading.mutex?view=netframework-4.7.2) in inter-process thread synchronization. It can be used in intra-process synchronization as well, but Monitor is prefferd as it makes better use of resources.

## 2.3 ReaderWriteLock

The [ReaderWriterLock](https://docs.microsoft.com/en-us/dotnet/api/system.threading.readerwriterlock?view=netframework-4.7.2) provides a locking mechanism that supports single writer and multiple readers to a shared resource.

## 3.1 Event & Wait Handles

Multiple threads can be synchronized by using _synchronization events_, which are object with two possible states - _signaled_ (threads are activated & executes), and _unsignaled_ (threads are suspended, goes to _Wait()_).

There are two types of synchronization events

* AutoResetEvent - goes to _unsignaled_ after activating a thread
* ManualResetEvent - allows to actiavte any number of threads until _Reset()_ is called

## 4.5 Interlocked

The [Interlocked](https://docs.microsoft.com/en-us/dotnet/api/system.threading.interlocked?view=netframework-4.7.2) class provides _atomic_ (as one single unit of work, in a single step) operations on variables that may be accesses by multiple threads. Even simple operations like updating a value, or adding two values are not a single-step operation in most platforms. This _may_ cause issues if a thread is time-sliced in-between and other thread is trying to access the same variable. Interlocked can prevent those scenarios.

```csharp
//update a value atomically
int sharedVar = 1;
int original = Interlocked.Exchange(ref sharedVar, 2);
//add 2 numbers atomically
int a = 5, b = 10; //a gets the sum value
int sum = Interlocked.Add(ref a, b);
```

#### References

* [Thread synchronization](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/threading/thread-synchronization)
* [Managed threading best practices](https://docs.microsoft.com/en-us/dotnet/standard/threading/managed-threading-best-practices)
* [threading in C# by Albahari](http://www.albahari.com/threading/)

* [Runtime object creation in CLR](/assets/downloads/Objects_MSDN_May2005.pdf)