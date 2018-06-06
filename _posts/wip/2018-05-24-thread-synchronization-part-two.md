---
layout: post
title: "Thread synchronization basics in .NET - part II"
excerpt: "High level overview of some less common cases of thread synchronization"
date: 2018-05-24
tags: [thread, synchronization, signaling, monitor, semaphore, deadlock]
categories: articles
image:
  feature: posts/threads/pxhere.jpg
  credit: pxhere
  creditlink: https://pxhere.com/en/photo/1032879
comments: true
share: true
published: false
---

**Note:** This article is currently incomplete & in-progress. It'll be updated soon.
{: .notice--danger}

## Signals

Signaling is used to synchronize between multiple threads, where they can signal each other to notify that a new thread can do something which was blocked by the signalling thread. The most common way of signalling is through `WaitHandle` that can coordinate between threads by notifying (signal) them when to go ahead and when to halt.

#### Event & Wait Handles

Multiple threads can be synchronized by using _synchronization events_, which are objects of `EventWaitHandle` or it's child types. These objects are meant to handle thread waiting, and some events to signal them. They can have named instances for system-wide synchronization. EventWaitHandle are like gates, which has 2 possible states

* **`signaled`** - waiting threads are activated & executes (open)
* **`unsignaled`** - waiting threads are suspended, continue to wait in a queue (closed)

There are three important methods in `EventWaitHandle`

* **`Set()`** - signals to say it is open now, threads can proceed. State becomes `signaled`
* **`Reset()`** - signals to say it's closed, other threads have to wait. State becomes `unsignaled`
* **`WaitOne()`** - a thread calls this to wait for a open signal. If current state is `signaled`, it can proceed immediately, else it has to wait

So, how the synchronization works among multiple threads is - all threads who wants to use the shared resources, calls `WaitOne()` and waits for a signal. When some thread calls `Set()` (or if the synchronization object was already `signaled`), the waiting threads proceed to access the resource. Now, when `Reset()` is called from a thread, no more new threads can proceed and have to wait. The object remains in `unsigaled` state until someone calls `Set()` again. At that moment, if no threads are waiting, the synchronization object remains in `signaled` state, or new threads start work.

There are several types of synchronization events

* [AutoResetEvent](https://msdn.microsoft.com/en-us/library/system.threading.autoresetevent.aspx) - allows exclusive **_access to a single thread at a time_**. It maintains an internal queue of all waiting threads, and lets one thread pass when it is _signaled_. The moment one thread with `WaitOne()` gets activated, it **_automatically_** resets to _unsignaled_ state. Hence the name.
* [ManualResetEvent](https://msdn.microsoft.com/en-us/library/system.threading.manualresetevent.aspx) - **_allows access to any number of threads_** in _signaled_ state. When it is in _signaled_ state, it allow all of the waiting threads to get activated, and keeps allowing new threads until some thread calls `Reset()` **_manually_** to put it in _unsignaled_ state. Hence the name.
* There is another very similar synchronization construct [CountdownEvent](https://msdn.microsoft.com/en-us/library/system.threading.countdownevent.aspx) - which is based on a counter. It gets set to _signaled_ state when it's count reaches 0, and allows all waiting threads to proceed. It stays in _unsignaled_ state al long as count is greater than 0. It is instatntiated with an initial count e.g. `new CountdownEvent(3)`, and count can be increased and decreased with `AddCount()` and `Signal()` respectively. **Note:** [1] If an attemp is made to bring the count below 0, it throws exception [2] Once the count reached 0, `AddCount()` cannot be called, but `Reset()` can be called to get the count to the initial value.

We'll look at some very basic examples to see how different `EventWaitHandle` work. First, lets look at **AutoResetEvent**

```csharp
static EventWaitHandle autoResetEvent =
    new EventWaitHandle(false, EventResetMode.AutoReset);
internal static void Execute2()
{
    var tasks = new Task[3];
    for (int i = 0; i < 3; i++)
    {
        tasks[i] = Task.Run(() => DoWork());
    } //other threads wait for signal
    Task.Delay(5000) //do work, signal to opne i.e. state = signaleded
        .ContinueWith((t) => manualResetEvent.Set());
    Task.WaitAll(tasks);
    //autoResetEvent.Reset(); //signal to close i.e. state = unsignaled
}
private static void DoWork()
{
    Console.WriteLine($"Thread {Thread.CurrentThread.ManagedThreadId} waiting for signal");
    autoResetEvent.WaitOne();
    //once it gets signal, wait handle AutoReset making it unsignaled
    Console.WriteLine($"Thread {Thread.CurrentThread.ManagedThreadId} got signal");
    Thread.Sleep(3000); //do some work
    Console.WriteLine($"Thread {Thread.CurrentThread.ManagedThreadId} leaving...");
    autoResetEvent.Set(); //signal to let other thread work
}
```

This is the produced result. Threads are queued and released one at a time.

```cmd
Thread 4 waiting for signal
Thread 3 waiting for signal
Thread 5 waiting for signal
Thread 3 got signal
Thread 3 leaving...
Thread 4 got signal
Thread 4 leaving...
Thread 5 got signal
Thread 5 leaving...
```

We'll now see almost the same exaple with **ManualResetEvent**. Even in this code `Reset()` is never called. Noe since it does not call it automatically, it remains in _signaled_ state once `Set()` is called.

```csharp
static EventWaitHandle manualResetEvent =
    new EventWaitHandle(false, EventResetMode.ManualReset);
internal static void Execute()
{
    var tasks = new Task[3];
    for (int i = 0; i < 3; i++)
    {
        tasks[i] = Task.Run(() => DoWork());
    }
    Task.Delay(5000) //wait & then set the event
        .ContinueWith((t) => manualResetEvent.Set());
    Task.WaitAll(tasks);
    //this code does not call Reset(), it remains signaled
}
private static void DoWork()
{
    Console.WriteLine($"Thread {Thread.CurrentThread.ManagedThreadId} waiting for signal");
    manualResetEvent.WaitOne(); //no Reset() called
    Console.WriteLine($"Thread {Thread.CurrentThread.ManagedThreadId} got signal");
    Thread.Sleep(1000); //do some work
    Console.WriteLine($"Thread {Thread.CurrentThread.ManagedThreadId} leaving...");
}
```

Result from manual reset event. All threads are released together. This code sample does not show a `Reset()` call, but in standard code that'd be necessary to block threads from accessing the shared code/resource, when required. Note that there is no guarantee in order of execution.

```cmd
Thread 3 waiting for signal
Thread 5 waiting for signal
Thread 4 waiting for signal
Thread 5 got signal
Thread 4 got signal
Thread 3 got signal
Thread 5 leaving...
Thread 4 leaving...
Thread 3 leaving...
```

## Non-blocking constructs

.NET Framework provides a bunch of non-blocking synchronization constructs can perform simple operations without ever blocking, pausing, or waiting. They also work across multiple processes.

The **`MemoryBarrier`**

> Synchronizes memory access as follows: The processor executing the current thread cannot reorder instructions in such a way that memory accesses prior to the call to MemoryBarrier execute after memory accesses that follow the call to MemoryBarrier. MemoryBarrier is required only on multiprocessor systems with weak memory ordering (for example, a system employing multiple Intel Itanium processors). - from [MSDN](https://msdn.microsoft.com/en-us/library/system.threading.thread.memorybarrier.aspx)

In _slightly_ simpler terms, it prevents any kind of instruction reordering or caching around that fence due to compiler optimization. So, it works as a wall or barrier preventing instructions or memory access going across it.

```csharp
//vague, not a real example
Thread.MemoryBarrier();
if (_jobCompleted)
{
  Thread.MemoryBarrier();
  log = $"Result = {_resultFromJob}";
}
```

**`Thread.VolatileRead`** & **`Thread.VolatileWrite`** makes sure all processor/threads get the latest value (read), or currently written value (write) is immediately available to all.

> VolatileRead and VolatileWrite are for special cases of synchronization. Under normal circumstances, the C# lock statement, the Visual Basic SyncLock statement, and the Monitor class provide easier alternatives. - [MSDN](https://msdn.microsoft.com/en-us/library/bah54t54(v=vs.110).aspx)

The **`volatile`** keyword, when used against refernce and some other allowed types (enum , pointer, byte, int etc.), JIT prevents any code optimization around it and prevents threads from caching it's value. The main intention is to _always get the latest value_ of the variable in scenarios, where other threads or the OS itself might be updating the value. More details [here](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/volatile).

**In general, it is advisable to use `lock` and avoid these low-level memory constructs in normal cicumstances.** One needs to understand them at a vey low level, and the platform's memory model at large, to apply them in a correct and benefiting way. For some detailed discussion, check [Eric Lippert's posts](https://blogs.msdn.microsoft.com/ericlippert/2011/06/16/atomicity-volatility-and-immutability-are-different-part-three/).
{: .notice--warning}

## Interlocked

The [Interlocked](https://docs.microsoft.com/en-us/dotnet/api/system.threading.interlocked?view=netframework-4.7.2) class provides _atomic_ (as one single unit of work, in a single step) operations on _variables_ that may be accesses by multiple threads. Even simple operations like updating/incrementing a value, or adding two values are not a single-step operation in most platforms. This _may_ cause issues if a thread is time-sliced in-between the process and other thread is trying to access the same variable. Interlocked can prevent those scenarios, by making the work atomic to the OS.

```csharp
//update a value atomically
int sharedVar = 1;
int original = Interlocked.Exchange(ref sharedVar, 2);
//add 2 numbers atomically
int a = 5, b = 10; //a gets the sum value
int sum = Interlocked.Add(ref a, b);
```

Typical scenario where `Interlocked` would be helpful is, when a method in a class modifies a class level variable (e.g. a private field, NOT a variable local to the method), and the method may run on multiple threads concurrently!

**Note:** A simple read or write on a field of 32 bits or less is always atomic. Operations on 64-bit fields are guaranteed to be atomic only in a 64-bit runtime environment, and statements that combine more than one read/write operation are never atomic. For example, writing to a `long` is not atomic on 32 bit machine, and `i++` where `i` is `Int32` is not atomic. - from [Albahari](http://www.albahari.com/threading/part4.aspx)
{: .notice--info}

## Some common issues with multi-threading

**Thread safetly**: A piece of code or data structure is thread safe, when the outcome of the code and underlying resources do not create undesirable results (inconsistent data, exception etc.) as a result of multiple threads interacting with the code concurrently.

**Thread contention**: When a thread has to wait to do some work or use a resource, because that is not readily available (e.g. locked by another thread or other reasons), and slows down because of that.

**Race condition**: In most general form, when two or more threads are trying to update a sahred data at the same time. Based on OS scheduling and time-slicing, the threads may update the value in order (like a race event). Because of that, the final state of data is unpredictable and the program can produce unexpected results.

**Deadlock**: When two or more threads are holding locks (on a _"critical section"_ of code or a resource) and waiting for the other thread(s) to release their resource (so that it can lock/use that), they come to a stand still waiting on each-other. This undesirable situation is known as a deadlock!

```csharp
static readonly object locker1 = new object();
static readonly object locker2 = new object();

new Thread(() => { //worker thread
    lock (locker1)
    {
        Thread.Sleep(1000);
        lock (locker2) { } // Deadlock
    }
}).Start();

lock (locker2) //main thread
{
    Thread.Sleep(1000);
    lock (locker1) { } // Deadlock
}
```

So, what kind of code is at risk of above type of issues? Any code that works on shared resources or data structures, that might be accesessed by multiple threads concurrently, are at risk. Most common example would be `static` methods or properties. Remember that the static methos that are part of .NET framework libraries, are made thread safe. But for user code, it's the programmers' responsibility to make them _thread safe_.

## Thread safe collections

- concurrent collections
- monitor pulse

###### All posts in the series - Tasks, Threads, Asynchronous

* **[Synchronous to asynchronous in .NET](/articles/sync-to-async-in-dotnet/)**
* **[Basic task cancellation demo in C#](/articles/task-cancellation/)**
* **[How does Async-Await work - Part I](/articles/async-await/)**
* **[How does Async-Await work - Part II](/articles/async-await-2/)**
* **[Basic thread synchronization with examples - Part I](/articles/thread-synchronization-part-one/)**
* **[Basic thread synchronization with examples - Part II](/articles/thread-synchronization-part-two/)**

#### References

* [Thread synchronization - MSDN](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/threading/thread-synchronization)
* [Managed threading best practices](https://docs.microsoft.com/en-us/dotnet/standard/threading/managed-threading-best-practices)
* [Threading in C# by Joseph Albahari](http://www.albahari.com/threading/)
* [WUSTL - Patterns for Concurrent, Parallel, and Distributed Systems](http://www.cs.wustl.edu/~schmidt/patterns-ace.html)
* [Async programming - unit testing considerations](https://msdn.microsoft.com/en-us/magazine/dn818494.aspx)