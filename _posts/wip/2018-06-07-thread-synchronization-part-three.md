---
layout: post
title: "Multithreading with signals | Thread synchronisation Part III"
excerpt: "Thread synchronization in .NET with signal & pulses"
date: 2018-06-07
tags: [thread, synchronization, multithreading, signal, waitevent, pulse]
categories: articles
image:
  feature: posts/threads/ogqcorp.jpg
  credit: ogqcorp.com
  creditlink: http://bgh.ogqcorp.com/share/h/SmP2O
comments: true
share: true
published: true
---

In this post, we'll see how to coordinate among many threads using signals. This is a continuation in the multithreading series, following **[Multithreading with non-exclusive locks &#124; Thread synchronisation Part II](/articles/thread-synchronization-part-two/)**.

## Signals

Signalling is used to synchronize between multiple threads, where they can signal each other to notify that a new thread can do something which was blocked by the another thread. The common ways of signalling are through `WaitHandle` and `Monitor`, that can coordinate between threads by notifying (signal) them when to go ahead and when to halt.

All the `EventWaitHandle` mentioned below are basically `WaitHandle` objects that can signal and wait for signals.

#### Event & Wait Handles

Multiple threads can be synchronized by using _synchronization events_, which are objects of `EventWaitHandle` or it's child types. These objects are meant to handle thread waiting, and some events to signal them. They can have named instances for system-wide synchronization. EventWaitHandle are like gates, which has 2 possible states

* **`signaled`** - waiting threads are activated & executes (open)
* **`nonsignaled`** - waiting threads are suspended, continue to wait in a queue (closed)

There are three important methods in `EventWaitHandle`

* **`Set()`** - signals to say it is open now, threads can proceed. State becomes `signaled`
* **`Reset()`** - signals to say it's closed, other threads have to wait. State becomes `nonsignaled`
* **`WaitOne()`** - a thread calls this to wait for an open signal. If current state is `signaled`, it can proceed immediately, else it has to wait until it gets a signal

So, how the synchronization works among multiple threads is - all threads who wants to use the shared resources, calls `WaitOne()` and waits for a signal. When some thread calls `Set()` (or if the synchronization object was already `signaled`), the waiting threads proceed to access the resource. Now, when `Reset()` is called from a thread, no more new threads can proceed and have to wait. The object remains in `nonsigaled` state until someone calls `Set()` again. When `Set()` is called, if no threads are waiting, the synchronization object remains in `signaled` state, and new threads can start the work immediately.

There are several types of synchronization events

* [AutoResetEvent](https://msdn.microsoft.com/en-us/library/system.threading.autoresetevent.aspx) - allows **_exclusive access to a single thread at a time_**. It maintains an internal queue of all waiting threads, and lets one thread pass when it is _signaled_. The moment one thread with `WaitOne()` gets activated, it **_automatically resets_** to _nonsignaled_ state. Hence the name.
* [ManualResetEvent](https://msdn.microsoft.com/en-us/library/system.threading.manualresetevent.aspx) - **_allows access to any number of threads_** in _signaled_ state. When it is in _signaled_ state, it allows all of the waiting threads to get activated, and keeps allowing new threads until some thread calls `Reset()` **_manually_** to put it in _nonsignaled_ state. That's why it is called manual-reset event.
* There is another very similar synchronization construct [CountdownEvent](https://msdn.microsoft.com/en-us/library/system.threading.countdownevent.aspx) - which is based on a counter. This does not derive from `EventWaitHandle`, but has an instance of `WaitHandle` as composition.

We'll look at some very basic examples to see how different `EventWaitHandle` work. Both AutoResetEvent & ManualresetEvent can be instantiated as an instance of `EventWaitHandle` with a `EventResetMode` passed into it, or they can be instantiated directly as the child type, e.g. `new AutoResetEvent(false)`. The boolean value indicates the initial state, so `false` will make it _nonsignaled_ initially.

###### AutoResetEvent example

All the threads coordinating through a `AutoResetEvent` wait with `WaitOne()` on the same instance of the `EventWaitHandle`. When any thread signals with a `Set()` event, one thread from the waiting queue is allowed to proceed, and it `Reset()` back to `nonsignaled` state, making the rest of the threads to wait for next signal.

```csharp
static EventWaitHandle autoResetEvent =
    new EventWaitHandle(false, EventResetMode.AutoReset);
internal static void Execute()
{
    var tasks = new Task[3]; //get 3 threads to do some work
    for (int i = 0; i < 3; i++)
    {
        tasks[i] = Task.Run(() => DoWork());
    }
    Thread.Sleep(5000); //indicative, it's doing some other work
    autoResetEvent.Set(); //first signal to open i.e. state = signaled
    Task.WaitAll(tasks); //wait for all 3 tasks to complete
}
private static void DoWork() //critical section that works based on signal
{
    Console.WriteLine($"Thread {Thread.CurrentThread.ManagedThreadId} waiting for signal");
    autoResetEvent.WaitOne();
    //once it gets signal, wait handle AutoReset making it nonsignaled again
    //so only one thread will proceed, others keep waiting for signal
    Console.WriteLine($"Thread {Thread.CurrentThread.ManagedThreadId} got signal");
    Thread.Sleep(3000); //do some work
    Console.WriteLine($"Thread {Thread.CurrentThread.ManagedThreadId} leaving...");
    autoResetEvent.Set(); //signal to make it open, i.e. state = signaled
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

Note that the order of execution of threads is nondeterministic. `AutoResetEvent` only makes sure that a single thread is allowed at a time among all the waiting threads.

###### ManualResetEvent example

For `ManualResetEvent`, again, all threads that are coordinating must wait with `WaitOne()` on the same instance of the `EventWaitHandle`. The main difference here is, it does not `Reset()` automatically, and needs to be `Reset()` manually by thread(s) with access to the instance. It does not have a limit on how many threads can proceed when it is `signaled`, but all threads can go ahead as long as it is in `signaled` state.

We'll now see almost the same example with `ManualResetEvent`. Even in this code `Reset()` is never called, for the sake of demonstration. Now, since it does not call it automatically, it just remains in _signaled_ state once `Set()` is called. In real world code, you'd need to call `Reset()` to stop new threads entering the critical section of code, as needed.

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
    //since Reset() is never called, it does not need to call Set()
}
```

Below is the result from manual reset event example. All threads are released together. Note that there is no guarantee in order of execution.

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

The `CountdownEvent` gets to _signaled_ state when it's count reaches 0, and allows all waiting threads to proceed. It stays in _nonsignaled_ state as long as count is greater than 0. It is instantiated with an initial count e.g. `new CountdownEvent(3)`, and count can be increased and decreased with `AddCount(n)` and `Signal()` respectively.

**Note:** [1] If an attempt is made to bring the count below 0, it throws exception [2] Once the count reached 0, `AddCount()` cannot be called, but `Reset()` can be called to get the count to the initial value.
{: .notice--warning}

#### Signalling with Monitor Wait and Pulses

Another, more flexible and advanced way to coordinate among many threads is to signal each-other with the static methods of the `Monitor` class. This is more flexible because it does not follow a _set-reset pattern_, or a _signaled state_. It can simply _signal_ threads just as a notification. The whole synchronization can be orchestrated with any set of custom variables, counters or flags.

Threads can wait for a signal with `Monitor.Wait(obj)`, where the `obj` is any object used for the purpose of identification of the context. So, threads coordinating in the same context (i.e. same scope of synchronization) will use the same `object`. A thread can notify other threads with `Pulse` signal. While `Monitor.Pulse(obj)` signals just one thread from the _waiting queue_, the `Monitor.PulseAll(obj)` method signals all threads in the _waiting queue_.

**Important:** The `Monitor.Wait()` is designed to be used within a `lock(obj)` statement, it throws exception if called outside a lock. Same for `Pulse()` and `PulseAll()`. And for correct synchronization, all threads in the scope must use the same `obj` for locking.
{: .notice--success}

Let's take a moment to understand what happens at **`Monitor.Wait(obj)`**. When a thread hits a `Monitor.Wait(obj)`, it releases the lock (remember, it must be within a `lock(obj)` statement), and allows other threads to get the lock. But it blocks the current thread until it gets back the lock. When this thread is signaled by another thread with `pulse` or `PulseAll` from another thread, it regains the lock the starts executing from the `Wait()` statement again.

Below code shows a classic example of signalling with Monitor, in a simple **Producer-Consumer Queue**. Consumer can take one item at a time from the queue as long as items are available. If no item available, it waits for new items to arrive. The producer adds items to the queue, and uses `PulseAll` to signal that new items have been added. So, if there are threads waiting to consume items, can go ahead once new items have been produced. See the inline comments for more details.

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
                //when item is available, it'll get signal & proceed
                Monitor.Wait(_syncLock); //it's NOT LOCKED at this point
                //once signalled, it'll go and try to regain the lock
                //all such threads will queue up in ready-queue
                //because of the lock, only one thread will consume at a time
                //whichever thread gets the lock...will continue from here
            }
            return _prodConsQueue.Dequeue(); //return one item from queue
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

The `Monitor` based signalling, being more flexible, is also easier to get wrong. Read Joseph Albahari's [book](http://www.albahari.com/threading/part4.aspx) for more detailed explanations and the correct & safe pattern for using Monitor based signalling.

The last article in this _thread synchronization_ series is currently in-progress, and not published yet. Please come back soon for more on multi-threading & thread synchronization.
{: .notice--info}

###### All posts in the series - Tasks, Threads, Asynchronous

* **[Synchronous to asynchronous in .NET](/articles/sync-to-async-in-dotnet/)**
* **[Basic task cancellation demo in C#](/articles/task-cancellation/)**
* **[How does Async-Await work - Part I](/articles/async-await/)**
* **[How does Async-Await work - Part II](/articles/async-await-2/)**
* **[Multithreading - lock, Monitor & Mutex &#124; Thread synchronization Part I](/articles/thread-synchronization-part-one/)**
* **[Multithreading with non-exclusive locks &#124; Thread synchronisation Part II](/articles/thread-synchronization-part-two/)**
* **Multithreading with signals &#124; Thread synchronisation Part III**

#### References

* [Thread synchronization - MSDN](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/threading/thread-synchronization)
* [Managed threading best practices](https://docs.microsoft.com/en-us/dotnet/standard/threading/managed-threading-best-practices)
* [Advanced threading - Joseph Albahari](http://www.albahari.com/threading/part4.aspx)
* [WUSTL - Patterns for Concurrent, Parallel, and Distributed Systems](http://www.cs.wustl.edu/~schmidt/patterns-ace.html)
* [Async programming - unit testing considerations](https://msdn.microsoft.com/en-us/magazine/dn818494.aspx)