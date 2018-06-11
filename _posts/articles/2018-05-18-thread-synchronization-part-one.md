---
layout: post
title: "Multithreading - lock, Monitor & Mutex | Thread synchronization Part I"
excerpt: "Some common practices of bare basic thread synchronization in .NET"
date: 2018-05-18
tags: [thread, synchronization, lock, monitor, multithreading]
categories: articles
image:
  feature: posts/threads/ogqcorp.jpg
  credit: ogqcorp.com
  creditlink: http://bgh.ogqcorp.com/share/h/SmP2O
comments: true
share: true
published: true
---

Modern computer systems make use of `threads` to do multiple things simultaneously, speed up work and keep the system responsive. All processes running on a computer use one or more threads to carry out the work it is doing, like computations, interacting with input-output systems, monitor background works etc. 

Using multiple threads has bunch of benefits, but also comes with overheads and complexities. Part of the complexity comes from that fact that the `OS` has to manage many threads by _time slicing_ and _context switching_. That means, to keep all the threads working, it allocates slices of CPU-time to each thread in turn, according to its scheduling algorithms. In most common terms, this is called `multi-threading`. All these add up to the CPU efforts and time.

The other major type of complexity comes from shared resources. Though each thread gets its own memory to use for the instructions it's executing, many things are shared among the threads. Like file system, other hardware, network connections etc., and also some type of data that are shared. For simple example, `static` properties are shared among all threads in a program.

Whenever some data is being shared, there are chances that more than one thread might try to manipulate the data at the same time, resulting in unexpected and unpredictable state of data. When resources are being shared (e.g. a database connection), one thread has to wait for another one to complete and release the resource. This needs coordination between multiple threads.

## What are threads

* Thread is a basic unit of execution
* A process (executing program) is run on one or more threads
* Thread has its own memory space and gets allotted processor time
* In .NET, all threads has its own stack (threads get own copy of all their local variables). But static variables are not, they are shared across the whole application
* Threads from same process also share the same heap memory, while different processes are completely isolated. System level resources are shared across processes
* Sequence of instructions managed independently by scheduler, schedulers are part of OS. CLR delegates the thread scheduling to the OS

Below `C#` code shows how to spin up a separate thread to do some work (a method). For some more details, see [Synchronous to asynchronous in .NET](/articles/sync-to-async-in-dotnet/).

```cs
//using System.Threading;
public void DoWorkOnThread()
{
    var thread = new Thread(SomeMethod); //instantiate a thread
    thread.Start(); //starts work (SomeMethod) on new worker thread
    //continue working on current, main thread
}
```

###### Multi-threading

Multi-threading is when you have more than one thread being used by a process. In a more generic way, it's when you have more threads than CPU cores in a system, so that the OS has to rotate them and allocate CPU and other resources in turn.

* `OS` does `time-slicing` and allocates CPU time to each thread in turn, following some scheduling algorithm
* The slice of time (on Windows) is generally close to 10 millisecond
* The overhead of this context switching is in the order of micro seconds
* When a thread is suspended from execution (e.g. due to time-slicing), it's called _"preempteed"_ thread

###### Thread overheads

* Creating a new thread takes about a few hundred milliseconds (roughly) - for the new stack allocation, allotting resources for spawning etc.
* Also uses around 1MB of memory per thread ([default](https://msdn.microsoft.com/en-us/library/windows/desktop/ms686774%28v=vs.85%29.aspx?f=255&MSPPError=-2147217396) in Windows)

###### ThreadPool

We've seen above that spinning up a new thread takes considerable amount of resources. Also maintaining threads has its own overheads. So, most of the times, it makes sense to re-use threads rather than creating new ones. One easy way to do that is to use the .NET managed `ThreadPool`. ThreadPool maintains a pool of worker threads, which are assigned to do _tasks_ on demand. Once the task is completed, the thread is taken back to the pool and used for other tasks as required.

* Keeps a pool of worker threads. And all threads are [background](https://msdn.microsoft.com/en-us/library/system.threading.threadpool.aspx) threads (terminates if main thread terminates)
* New work is allocated to idle threads, and are returned to the pool once done
* If there is more work than maximum thread limit, they are queued
* Can be used via a `Task` or `QueueUserWorkItem()`

###### Tasks

.NET provides a simple clean way of running tasks in separate threads, without creating new threads. That is `Task`. Conceptually, a `Task` is like a promise of a work, that'll be completed in future. See [Synchronous to asynchronous in .NET](/articles/sync-to-async-in-dotnet/) for more details.

* Designed to return value, if required with `Task<T>`
* Tasks are run on the managed ThreadPool, and _generally_ run as background thread
* Can easily `Wait` for one or more Tasks and `Continue` with other Tasks
* Supports parent-child relationships and aggregated exceptions as `AggregateException`
* Tasks cancellation is easier with cancellation tokens, see [task cancellation example](/articles/task-cancellation/)
* It is advised to use a `Task` over a `Thread` almost always. If you need a dedicated thread for a long running task, still prefer using a Task with `LongRunning` of `TaskCreationOptions`, for the sake of consistency. Pure threads might be used for very low-level uses, e.g. custom schedulers etc.

## Thread synchronization

In a multi-threaded environment, multiple threads can speed up works and keep main thread responsive (e.g. UI thread in Windows applications). In such scenarios, they can also access different resources like files, network connections, memory etc. as per the application needs. If done incorrectly, what may happen is multiple threads trying to use and/or update same resource at the same time, unaware of each other. This can result in unpredictable and inconsistent results.

So, in multi-threaded applications, the threads needs to be synchronized, so that they do not work on (most importantly, update) the same resource at the same time. There are [several ways to synchronize threads](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/threading/thread-synchronization) in .NET environment.

1. Blocking constructs - blocking threads and make them wait for something. Blocked threads do not consume CPU, but they still consume memory
    1. Thread.Sleep() - sleep for specified milliseconds
    2. Thread.Join() - wait for thread to complete work
    3. Task.Wait() - wait for task to complete work
2. Locks - limits number of threads that can enter/access a _"critical section"_ of code
    1. Exclusive locks - allows only one thread
        1. lock(), Monitor.Enter() - Monitor.Exit()
        2. Mutex
    2. Non-exclusive locks - allows a limited number of threads
        1. Semaphore
        2. SemaphoreSlim
        3. ReaderWriterLock
        4. ReaderWriterLockSlim
3. Signals - allows thread to pause and wait until a notification is received from another thread
    1. Event wait handles (ManualResetEvent, AutoResetEvent, CountdownEvent)
    2. Monitor.Wait(), Monitor.Pulse(), Monitor.PulseAll()
4. Nonblocking construcuts - protects access to a common field with specific access rules
    1. Thread.MemoryBarrier
    2. Volatile, Thread.VolatileRead, Thread.VolatileWrite
    3. Interlocked

Here, we'll look at the some of the commonly used ways of thread synchronization. First, the exclusive locks. They make sure, that one and only one thread can enter a critical section of code.

#### Monitor & lock

The `lock()` is the most widely used construct for thread synchronization, and it's one of the easiest ones to use. It can lock a section of code that needs to be handled in thread-safe way, or commonly known as a _"critical section"_ of code.

Lock creates something like a room with lock, for which only one key is there and only the person with the key can enter. The next person has to wait until the previous person comes out and gives her the key. So, only one thread can enter the block of code protected with a `lock(lockObject)`.

In .NET, the `lock()` is actually constructed with `Monitor` class internally, i.e. the compiler translates the lock statement into a `Monitor.Enter()` and `Monitor.Exit()` pair. Basically, a `lock()` is safe syntax (because `Monitor.Exit()` is put in `finally`) for using Enter-Exit pair of Monitor class.

```cs
private static readonly object lockObject = new object();
lock(lockObject)
{
    //critical section of code
}

//translates to (C# 4, .NET 4.0 and above)
bool acquiredLock = false;
try
{
    Monitor.Enter(lockObject, ref acquiredLock);
    //on Enter, acquiredLock is automatically marked true
    //do the actual work
}
finally
{
    if (acquiredLock)
    {
        Monitor.Exit(lockObject);
    }
}
```

**Note:** The most widely used way to protect a piece of code (or the resource it accesses) from being used by multiple threads simultaneously, is to use a `lock()`. It's almost always advised to use `lock()` over other synchronization constructs, as it is easier to use and chances of writing error-prone multi-threaded code is much less than the others. Use the other synchronization constructs for specific needs, for example, when you want to allow a specific number (more than one) of threads to access the _critical section_.
{: .notice--success}

The [Monitor](https://msdn.microsoft.com/en-us/library/system.threading.monitor.aspx#Lock) class is a .NET specific class used for exclusive locks. It cannot be instantiated, and has only static methods & can be called from any context. The `Monitor` based locks are uniquely identified with an `object`. So, one specific lock needs to use a specific object, and if two separate locks are needed, two separate objects need to be used. The `Monitor` can associate itself with object(s) on demand (i.e. on `Monitor.Enter(obj)` or `Monitor.TryEnter(obj)` or `lock(obj)`). `Monitor.Enter()` acquires an exclusive lock based on the lock-object, and `Monitor.Exit()` releases the lock. For each `lock-object`, it keeps track of

* A reference to the thread that currently holds the lock (through the specific lock-object)
* A reference to a `ready-queue`, which contains the threads that are ready to obtain the lock
* A reference to a `waiting-queue`, which contains the threads that are waiting for notification of a change in the state of the lock-object

It's suggested to use a `private readonly static` object to lock

1. If the object is accessible outside, someone else might acquire a lock on the same object, putting the current thread to sleep, or create dead-lock. Same might happen to a string instance because of _"string interning"_. So, `private`
2. At the time of locking, the object should not be null. A `null` object cannot be locked, see below. So, initiate it once and make sure it is not null. So, `readonly`
3. Generally, if the shared resource/code is `static`, then the lock object also needs to be `static`. Otherwise, all instances of the code (e.g. different threads calling it) will not be synchronized

Locking does not work on value types. On `lock`/`Monitor.Enter`, the instance of the lock object gets a `Sync Block` structure added to its managed memory block (sync block can also get initialized for other reasons, e.g. to store results of `GetHashCode()` method), and the `ManagedThreadId` of the current thread holding the lock is saved. This is used by the `Monitor` for synchronization. For the same reason, a `null` object cannot be locked.

The `Monitor` class also supports signalling, that we'll see later.

#### Mutex

`Mutex` (from _"**mut**ually **ex**clusive"_) is similar to `Monitor` in that, it allows only one thread to execute a block of code, but unlike Monitor, it can work across processes. Monitor is more of a .NET construct, whereas Mutex is a wrapper over OS level Win32 construct.

Generally a string name is used to identify a [Mutex](https://docs.microsoft.com/en-us/dotnet/api/system.threading.mutex?view=netframework-4.7.2) in inter-process thread synchronization, and that can be across the machine. These mutex objects are called _"named instances"_. Mutex can be used in intra-process synchronization as well, but Monitor is preferred as it makes better use of resources in managed environment, and easier to use with `lock()`.

```csharp
static Mutex mutex = new Mutex();

if (mutex.WaitOne(2000)) //wait to get mutext with timeout of 2 seconds
{
    Console.WriteLine("Entered mutex");
    //execute the critical section of work
    Console.WriteLine("Releasing mutex");
    mutex.ReleaseMutex();
}
else
    Console.WriteLine("Could not acquire Mutex, timeout expired");
```

In the next article, we'll look at **[multithreading with non-exclusive locks](/articles/thread-synchronization-part-two/)**.

#### All posts in the series - Tasks, Threads, Asynchronous

* **[Synchronous to asynchronous in .NET](/articles/sync-to-async-in-dotnet/)**
* **[Basic task cancellation demo in C#](/articles/task-cancellation/)**
* **[How does Async-Await work - Part I](/articles/async-await/)**
* **[How does Async-Await work - Part II](/articles/async-await-2/)**
* **Multithreading - lock, Monitor & Mutex &#124; Thread synchronization Part I**
* **[Multithreading with non-exclusive locks &#124; Thread synchronisation Part II](/articles/thread-synchronization-part-two/)**
* **[Multithreading with signals &#124; Thread synchronisation Part III](/articles/thread-synchronization-part-three/)**
* **[Non-blocking multithreading & concurrent collections &#124; Thread synchronization Part IV](/articles/thread-synchronization-part-four/)**

#### References

* [Synchronization in Windows](https://msdn.microsoft.com/en-us/library/ms686353.aspx)
* [Threading in C# by Joseph Albahari](http://www.albahari.com/threading/)
* [Processes, Threads, and Jobs in the Windows](https://www.microsoftpressstore.com/articles/article.aspx?p=2233328&seqNum=7)
* [Runtime object creation in CLR](/assets/downloads/Objects_MSDN_May2005.pdf)