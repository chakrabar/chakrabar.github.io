---
layout: post
title: "Thread synchronisation with signals | Thread synchronisation Part III"
excerpt: "High level overview of some less common cases of thread synchronization continued"
date: 2018-06-07
tags: [thread, synchronization, signaling, monitor, semaphore, deadlock]
categories: articles
image:
  feature: posts/threads/pxhere.jpg
  credit: pxhere..blah
  creditlink: https://pxhere.com/en/photo/1032879
comments: true
share: true
published: false
---

**Note:** This article is currently incomplete & in-progress. It'll be updated soon.
{: .notice--danger}

## Signals

Signaling is used to synchronize between multiple threads, where they can signal each other to notify that a new thread can do something which was blocked by the signalling thread. The most common way of signalling is through `WaitHandle` that can coordinate between threads by notifying (signal) them when to go ahead and when to halt.

`Mutex` and all the `EventWaitHandle` mentioned below are basically `WaitHandle` objects that can signal and wait for signals.

#### Event & Wait Handles

Multiple threads can be synchronized by using _synchronization events_, which are objects of `EventWaitHandle` or it's child types. These objects are meant to handle thread waiting, and some events to signal them. They can have named instances for system-wide synchronization. EventWaitHandle are like gates, which has 2 possible states

* **`signaled`** - waiting threads are activated & executes (open)
* **`nonsignaled`** - waiting threads are suspended, continue to wait in a queue (closed)

There are three important methods in `EventWaitHandle`

* **`Set()`** - signals to say it is open now, threads can proceed. State becomes `signaled`
* **`Reset()`** - signals to say it's closed, other threads have to wait. State becomes `nonsignaled`
* **`WaitOne()`** - a thread calls this to wait for a open signal. If current state is `signaled`, it can proceed immediately, else it has to wait

So, how the synchronization works among multiple threads is - all threads who wants to use the shared resources, calls `WaitOne()` and waits for a signal. When some thread calls `Set()` (or if the synchronization object was already `signaled`), the waiting threads proceed to access the resource. Now, when `Reset()` is called from a thread, no more new threads can proceed and have to wait. The object remains in `unsigaled` state until someone calls `Set()` again. At that moment, if no threads are waiting, the synchronization object remains in `signaled` state, or new threads start work.

There are several types of synchronization events

* [AutoResetEvent](https://msdn.microsoft.com/en-us/library/system.threading.autoresetevent.aspx) - allows exclusive **_access to a single thread at a time_**. It maintains an internal queue of all waiting threads, and lets one thread pass when it is _signaled_. The moment one thread with `WaitOne()` gets activated, it **_automatically_** resets to _nonsignaled_ state. Hence the name.
* [ManualResetEvent](https://msdn.microsoft.com/en-us/library/system.threading.manualresetevent.aspx) - **_allows access to any number of threads_** in _signaled_ state. When it is in _signaled_ state, it allow all of the waiting threads to get activated, and keeps allowing new threads until some thread calls `Reset()` **_manually_** to put it in _nonsignaled_ state. Hence the name.
* There is another very similar synchronization construct [CountdownEvent](https://msdn.microsoft.com/en-us/library/system.threading.countdownevent.aspx) - which is based on a counter.

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
    //autoResetEvent.Reset(); //signal to close i.e. state = nonsignaled
}
private static void DoWork()
{
    Console.WriteLine($"Thread {Thread.CurrentThread.ManagedThreadId} waiting for signal");
    autoResetEvent.WaitOne();
    //once it gets signal, wait handle AutoReset making it nonsignaled
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

###### CountdownEvent

The `CountdownEvent` gets to _signaled_ state when it's count reaches 0, and allows all waiting threads to proceed. It stays in _nonsignaled_ state as long as count is greater than 0. It is instatntiated with an initial count e.g. `new CountdownEvent(3)`, and count can be increased and decreased with `AddCount(n)` and `Signal()` respectively.

**Note:** [1] If an attemp is made to bring the count below 0, it throws exception [2] Once the count reached 0, `AddCount()` cannot be called, but `Reset()` can be called to get the count to the initial value.
{: .notice--warning}

## Signalling with Monitor Wait and Pulses

```csharp
class SimpleProducerConsumerQueue<T>
{
    readonly object _syncLock = new object();
    Queue<T> _prodConsQueue = new Queue<T>();

    public T ConsumeItem()
    {
        lock(_syncLock) //safely get one item
        {
            while(_prodConsQueue.Count == 0)
            {
                //if no item, release lock but block the thread
                //move it to the wait-queue and wait for signal
                //when item is available, it'll get signal
                Monitor.Wait(_syncLock); //it's NOT LOCKED at this point
                //once signalled, it'll go and try to regain the lock
                //all such threads will queue up in ready-queue
                //because of the lock, only one thread will consume at a time
                //whichever thread gets the lock...will continue from here
            }
            return _prodConsQueue.Dequeue();
        }
    }

    public void ProduceItems(params T[] itemsToAdd)
    {
        lock(_syncLock) //exclusively add items - single activity in class
        {
            foreach (var item in itemsToAdd)
            {
                _prodConsQueue.Enqueue(item);
            }
            //once items are added, signal all threads that are waiting
            Monitor.PulseAll(_syncLock);
            //this will move all threads from wait-queue to ready-queue
        }
    }
}
```

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