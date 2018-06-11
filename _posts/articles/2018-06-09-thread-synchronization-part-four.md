---
layout: post
title: "Non-blocking multithreading & concurrent collections | Thread synchronization Part IV"
excerpt: "Non-blocking thread synchronization, multithreading issues and concurrent collections in .NET"
date: 2018-06-09
tags: [thread, synchronization, multithreading, deadlock, interlocked, concurrent]
categories: articles
image:
  feature: posts/threads/pxhere.jpg
  credit: pxhere
  creditlink: https://pxhere.com/en/photo/1032879
comments: true
share: true
published: true
---

In this post, we'll look at some non-blocking constructs provided by the .NET framework for safer access of data in multithreading environments. We'll also look at some potential issues in multithreading and the thread-safe concurrent collections. This is the last post in the multithreading series, following **[Multithreading with signals &#124; Thread synchronization Part III](/articles/thread-synchronization-part-three/)**.

## Non-blocking constructs

.NET Framework provides a bunch of non-blocking synchronization constructs can perform simple operations without ever blocking, pausing, or waiting threads. They also work across multiple processes.

The **`MemoryBarrier`**

> Synchronizes memory access as follows: The processor executing the current thread cannot reorder instructions in such a way that memory accesses prior to the call to MemoryBarrier execute after memory accesses that follow the call to MemoryBarrier. - from [MSDN](https://msdn.microsoft.com/en-us/library/system.threading.thread.memorybarrier.aspx)

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

**Note:** The use of `MemoryBarrier` may lead to complicated, hard to debug pitfalls. Same can happen with `volatile`, `VolatileRead` and `VolatileWrite` explained below. Read it from Albahari's [book](http://www.albahari.com/threading/part4.aspx#_Memory_Barriers_and_Volatility) for detailed explanation and examples. Simply putting a **`lock()`** around those field access is generally much more safe and error free, at a potential cost of a very minor performance loss (in order of 10s of nano seconds).
{: .notice--danger}

The `volatile` declaration on a field, and the `volatileRead` & `VolatileWrite` static methods on `Thread` class basically tries to ensure that all threads get latest values available, when multiple threads (and OS, hardware etc.) are accessing them simultaneously.

**`Thread.VolatileRead`** & **`Thread.VolatileWrite`** makes sure all processor/threads get the latest value (read), or currently written value (write) is immediately available to all.

> VolatileRead and VolatileWrite are for special cases of synchronization. Under normal circumstances, the C# lock statement, the Visual Basic SyncLock statement, and the Monitor class provide easier alternatives. - [MSDN](https://msdn.microsoft.com/en-us/library/bah54t54(v=vs.110).aspx)

The **`volatile`** keyword, when used against reference and some other allowed types (enum , pointer, byte, int etc.), JIT prevents any code optimization around it and prevents threads from caching its value. The main intention is to _always get the latest value_ of the variable in scenarios where other threads or the OS itself might be updating the value. More details [here](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/volatile).

```csharp
//all threads/processors will always get latest value of this
public volatile int _criticalCount1 = 1;

private int _criticalCount2 = 0;
public int UpdateAndRead(int value)
{
    //write and make new value available to all threads/processors
    Thread.VolatileWrite(ref _criticalCount2, value);
    //read latest value, in case some thread/processor has updated it
    int updatedVal = Thread.VolatileRead(ref _criticalCount2);
    return updatedVal;
}
```

**In general, it is advisable to use `lock` and avoid these low-level memory constructs in normal circumstances.** One needs to understand them at a very low level, and the platform's memory model at large, to apply them in a correct and benefiting way. For some detailed discussion, check [Eric Lippert's posts](https://blogs.msdn.microsoft.com/ericlippert/2011/06/16/atomicity-volatility-and-immutability-are-different-part-three/).

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

Typical scenario where `Interlocked` would be helpful is, when a method in a class modifies a `static` variable (e.g. a class-level private field, NOT a variable local to the method), and the method may run on multiple threads concurrently!

**Note:** A simple read or write on a field of 32 bits or less is always atomic. Operations on 64-bit fields are guaranteed to be atomic only in a 64-bit runtime environment, and statements that combine more than one read/write operation are never atomic. For example, writing to a `long` is not atomic on 32 bit machine, and `i++` where `i` is `Int32` is still not atomic. - from [Albahari](http://www.albahari.com/threading/part4.aspx)
{: .notice--info}

## Some common issues with multi-threading

**Thread safety**: A piece of code or data structure is thread safe, when the outcome of the code and underlying resources do not create undesirable results (inconsistent data, exception etc.) as a result of multiple threads interacting with the code concurrently.

**Thread contention**: When a thread has to wait to do some work or use a resource, because that is not readily available (e.g. locked by another thread or other reasons), and slows down because of that.

**Race condition**: In most general form, when two or more threads are trying to update a shared data at the same time. Based on OS scheduling and time-slicing, the threads may update the value in any order (like a race event). Because of that, the final state of data is unpredictable and the program can produce unexpected results.

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

So, what kind of code is at risk of above type of issues? Any code that works on shared resources or data structures, that might be accessed by multiple threads concurrently, are at risk. Most common example would be `static` methods, properties or class level fields. Remember that the static methods that are part of .NET framework libraries, are made thread safe. But for user code, it's the programmers' responsibility to make them _thread safe_.

## Thread safe or concurrent collections

.NET Framework 4.0 introduced a new bunch of collections under the namespace [System.Collections.Concurrent](https://docs.microsoft.com/en-us/dotnet/api/system.collections.concurrent?view=netframework-4.7.2), that are thread-safe by nature. That means, they can be read or be written safely even when multiple threads are accessing them concurrently.

The [thread-safe collections](https://docs.microsoft.com/en-us/dotnet/standard/collections/thread-safe/) use lightweight synchronization mechanisms such as `SpinLock`, `SpinWait`, `SemaphoreSlim`, and `CountdownEvent` making them very efficient under multithreading environment. The `ConcurrentQueue<T>` and `ConcurrentStack<T>` classes do not use locks at all. Instead, they rely on `Interlocked` operations to achieve thread-safety.

Some of the very useful thread-safe collections are (there are more)

* [ConcurrentBag&lt;T&gt;](https://docs.microsoft.com/en-us/dotnet/api/system.collections.concurrent.concurrentbag-1?view=netframework-4.7.2) : thread-safe collection of unordered items
* [ConcurrentQueue&lt;T&gt;](https://docs.microsoft.com/en-us/dotnet/api/system.collections.concurrent.concurrentqueue-1?view=netframework-4.7.2) : thread-safe FIFO collection
* [ConcurrentStack&lt;T&gt;](https://docs.microsoft.com/en-us/dotnet/api/system.collections.concurrent.concurrentstack-1?view=netframework-4.7.2) : thread-safe LIFO collection
* [ConcurrentDictionary&lt;TKey,TValue&gt;](https://docs.microsoft.com/en-us/dotnet/api/system.collections.concurrent.concurrentdictionary-2?view=netframework-4.7.2) : thread-safe collection of key-value dictionary

They all provide safe methods for adding data, and useful properties like `IsEmpty` and `Count`. The `ConcurrentDictionary` provides an additional bunch of practical methods like `AddOrUpdate()`, `GetOrAdd()` etc.

The code below, shows simple demo of `ConcurrentBag` and `ConcurrentStack`.

```csharp
static void ConcurrentBagDemo()
{
    //instantiate with values from IEnumerable<T>
    var safeBag = new ConcurrentBag<string>(new string[] { "a", "b", "c" });
    bool isBagEmpty = safeBag.IsEmpty; //false
    int itemsInBag = safeBag.Count; //3
    safeBag.Add("another item"); //add an item to bag

    //this will get items in any order
    while (safeBag.TryTake(out string result)) //C# 7.0 construct
    {
        Console.WriteLine($"Got item {result}"); //will iterate 4 times
    }
    //now that bag is empty, TryTake will return FALSE
    Console.WriteLine($"More items in bag? {safeBag.TryTake(out string item)}");
}

static void ConcurrentStackDemo()
{
    var safeStack = new ConcurrentStack<string>();
    //add strings "1" - "5" to stack
    foreach (var item in Enumerable.Range(1, 5))
    {
        safeStack.Push(item.ToString());
    }
    bool isStackEmpty = safeStack.IsEmpty; //false
    int itemsInStack = safeStack.Count; //5
    //add multiple items with PushRange()
    safeStack.PushRange(new string[] { "x", "y", "z" });

    var data = new string[2]; //try pop 2 items at a time, into data
    while (safeStack.TryPopRange(data) > 0) //no. of items popped
    {
        Console.WriteLine($"Got items {String.Join(',', data)}");
        data = new string[2]; //no old data in case of partial read
    } //this loop will iterate 4 times, 4 * 2 = 8
}
```

###### All posts in the series - Tasks, Threads, Asynchronous

* **[Synchronous to asynchronous in .NET](/articles/sync-to-async-in-dotnet/)**
* **[Basic task cancellation demo in C#](/articles/task-cancellation/)**
* **[How does Async-Await work - Part I](/articles/async-await/)**
* **[How does Async-Await work - Part II](/articles/async-await-2/)**
* **[Multithreading - lock, Monitor & Mutex &#124; Thread synchronization Part I](/articles/thread-synchronization-part-one/)**
* **[Multithreading with non-exclusive locks &#124; Thread synchronization Part II](/articles/thread-synchronization-part-two/)**
* **[Multithreading with signals &#124; Thread synchronization Part III](/articles/thread-synchronization-part-three/)**
* **Non-blocking multithreading & concurrent collections &#124; Thread synchronization Part IV**