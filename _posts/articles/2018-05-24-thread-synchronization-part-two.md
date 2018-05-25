---
layout: post
title: "Thread synchronization basics in .NET - part II"
excerpt: "High level overview of some less common cases of thread synchronization"
date: 2018-05-24
tags: [thread, synchronization, lock, monitor, semaphore, deadlock]
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

Signaling is used to synchronize between multiple threads, where they can signal each other to notify that a new thread can do something which was blocked by the signalling thread.

#### Event & Wait Handles

Multiple threads can be synchronized by using _synchronization events_, which are object with two possible states - **`signaled`** (threads are activated & executes), and **`unsignaled`** (threads are suspended, goes to `Wait()`). They can have named instances for system-wide synchronization.

There are two types of synchronization events

* AutoResetEvent - allows exclusive access, goes to _unsignaled_ after activating a thread
* ManualResetEvent - allows to actiavte any number of threads until _Reset()_ is called
* CountDownEvent - 

Autoreset - 

* a thread waits for a signal by calling `WaitOne()`
* if the event currently in unsiganaled state, the thread blocks and waits

Threads "take control" and wait handle goes in unsignaled state. Currently no other thread can "take control". Once work is done, the current thread calls `Set()` to signal, and the wait handle enters signaled state. If there are threads already waiting who called `WaitOne()`, one of them gets the handle. And handle object goes to unsignaled state. If there were more than one thread waiting, the'd form a queue. If no thread was waiting, the handle would stay in signaled state.

```csharp
static EventWaitHandle autoResetEvent = new EventWaitHandle(false, EventResetMode.AutoReset);
//false means initially the handle is unsignaled or reset

internal static void Execute()
{
    Task.Run(() => DoWork()); //another thread to wait for signal
    Thread.Sleep(5000); //do some work
    autoResetEvent.Set(); //signal to opne i.e. state = signaleded
    //autoResetEvent.Reset(); //signal to close i.e. state = unsignaled
}

private static void DoWork()
{
    Console.WriteLine($"Thread {Thread.CurrentThread.ManagedThreadId} waiting for signal");
    autoResetEvent.WaitOne();
    //once it gets signal, wait handle AutoReset making it unsignaled
    Console.WriteLine($"Thread {Thread.CurrentThread.ManagedThreadId} got signal");
    Thread.Sleep(2000); //do some work
    autoResetEvent.Set(); //signal to let other thread work, signaleded
}
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

In general, it is advisable to use `lock` and avoid these low-level memory constructs in normal cicumstances. One needs to understand them at a vey low level, and the platform's memory model at large, to apply them in a correct and benefiting way. For some detailed discussion, check [Eric Lippert's posts](https://blogs.msdn.microsoft.com/ericlippert/2011/06/16/atomicity-volatility-and-immutability-are-different-part-three/).
{: .notice--warning}

## 4.5 Interlocked

The [Interlocked](https://docs.microsoft.com/en-us/dotnet/api/system.threading.interlocked?view=netframework-4.7.2) class provides _atomic_ (as one single unit of work, in a single step) operations on variables that may be accesses by multiple threads. Even simple operations like updating/incrementing a value, or adding two values are not a single-step operation in most platforms. This _may_ cause issues if a thread is time-sliced in-between the process and other thread is trying to access the same variable. Interlocked can prevent those scenarios, by making the work atomic to the OS.

```csharp
//update a value atomically
int sharedVar = 1;
int original = Interlocked.Exchange(ref sharedVar, 2);
//add 2 numbers atomically
int a = 5, b = 10; //a gets the sum value
int sum = Interlocked.Add(ref a, b);
```

**Note:** A simple read or write on a field of 32 bits or less is always atomic. Operations on 64-bit fields are guaranteed to be atomic only in a 64-bit runtime environment, and statements that combine more than one read/write operation are never atomic. For example, writing to a `long` is not atomic on 32 bit machine, and `i++` where `i` is `Int32` is not atomic. - from [Albahari](http://www.albahari.com/threading/part4.aspx)
{: .notice--info}

## Some common issues with multi-threading

**Thread safetly**: A piece of code is thread safe, when the outcome of the code and underlying resources do not create undesirable results (inconsistent data, exception etc.) as a result of multiple threads interacting with the code concurrently.

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

#### References

* [Thread synchronization](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/threading/thread-synchronization)
* [Managed threading best practices](https://docs.microsoft.com/en-us/dotnet/standard/threading/managed-threading-best-practices)
* [Threading in C# by Joseph Albahari](http://www.albahari.com/threading/)
* [Processes, Threads, and Jobs in the Windows](https://www.microsoftpressstore.com/articles/article.aspx?p=2233328&seqNum=7)

* [Runtime object creation in CLR](/assets/downloads/Objects_MSDN_May2005.pdf)