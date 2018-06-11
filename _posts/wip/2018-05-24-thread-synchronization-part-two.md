---
layout: post
title: "Multithreading with non-exclusive locks | Thread synchronisation Part II"
excerpt: "Synchronizing multiple threads with non-exclusive locks in .NET"
date: 2018-05-24
tags: [thread, synchronization, multithreading, readerwriterlock, semaphore, non-exclusive-locks]
categories: articles
image:
  feature: posts/threads/pxhere.jpg
  credit: pxhere
  creditlink: https://pxhere.com/en/photo/1032879
comments: true
share: true
published: true
---

In this post, we'll look at some of the non-exclusive locks that allow more than one threads to access a _critical section_ of code at a time, based on some algorithms. This is a continuation of **[Multithreading - lock, Monitor & Mutex &#124; Thread synchronization Part I](/articles/thread-synchronization-part-one/)**.

#### ReaderWriteLock

The [ReaderWriterLock](https://docs.microsoft.com/en-us/dotnet/api/system.threading.readerwriterlock?view=netframework-4.7.2) provides a locking mechanism that supports single writer and multiple readers to access a shared resource. It provides `AcquireReaderLock()` and `AcquireWriterLock()` methods, where multiple threads can acquire reader lock simultaneously, but only one thread can have writer lock at a time and no thread can read at that time.

> Readers and writers are queued separately. When a thread releases the writer lock, all threads waiting in the `reader-queue` *at that instant* are granted reader locks; when all of those reader locks have been released, the next thread waiting in the `writer-queue`, if any, is granted the writer lock, and so on. In other words, `ReaderWriterLock` alternates between a collection of readers, and one writer. - from MSDN

While read is happening, new read threads queue up rather than joining the current batch of read threads. This prevents the write threads from waiting indefinitely.

`ReaderWriterLock` works much more efficiently than `lock` or `Monitor` when _many more_ reads are happening on a resource compared to writes (e.g. a configuration file), and writes are slow.
{: .notice--success}

Sample code showing a `ReaderWriterLock` used only for a reader lock. We'll see a more complete example in the `ReaderWriterLockSlim` section later.

```csharp
static ReaderWriterLock rwl = new ReaderWriterLock();

static void ReadSharedResource(int timeOutMilliSec)
{
    try
    {
        rwl.AcquireReaderLock(timeOutMilliSec);
        try
        {
            Console.WriteLine($"Current resource value on read {resource}");
        }
        finally
        {
            rwl.ReleaseReaderLock();
        }
    }
    catch (ApplicationException)
    { //timed out before getting lock
        Console.WriteLine("Reader lock timed out");
    }
}
```

**Important:** For all new developments, it's recommended to use `ReaderWriterLockSlim` instead of `ReaderWriterLock`. It has same features and as it has [1] simpler rules for recursion control [2] simplified up/down-grade of locks [3] avoids many potential deadlocks [4] performs better.
{: .notice--warning}

#### ReaderWriterLockSlim

`ReaderWriterLockSlim` has similar features of `ReaderWriterLock` and the additional benefits mentioned above. Also code is more readable with [ReaderWriterLockSLim](https://docs.microsoft.com/en-us/dotnet/api/system.threading.readerwriterlockslim?view=netframework-4.7.2). It's basically a _slim_ or _lightweight_ version of ReaderWriterLock, which should be preferred over `ReaderWriterLock`.

```csharp
public class SafeReadWrite
{
  static ReaderWriterLockSlim rwLockSlim = new ReaderWriterLockSlim();
  static int resource = 0; //shared resource
  
  //gets reader lock (without any timeout limit)
  static void ReadSharedResource()
  {
      rwLockSlim.EnterReadLock();
      try
      {
          Console.WriteLine($"Current resource value on read {resource}");
      }
      finally
      {
          rwLockSlim.ExitReadLock();
      }
  }

  //tries to get writer lock with a timeout
  static void WriteToSharedResource(int timeOutMilliSec)
  {
      if (rwLockSlim.TryEnterWriteLock(timeOutMilliSec))
      {
          try
          {
              resource = 100;
              Console.WriteLine($"Current resource value after write {resource}");
          }
          finally
          {
              rwLockSlim.ExitWriteLock();
          }
      }
      else
      { //timeOutMilliSec elapsed, could not get lock
          Console.WriteLine("Writer lock timed out");
      }
  }
  
  ~SafeReadWrite() //ReaderWriterLockSlim implements IDisposable
  { //do not forget to dispose
    if (rwLockSlim != null)
        rwLockSlim.Dispose();
  }
}
```

It also provides methods `EnterUpgradeableReadLock` and `ExitUpgradeableReadLock` to upgrade a reader lock to writer, and bring back avoiding possible deadlocks. When this is used, the thread upgrading to writer lock gets placed at writer queue.

#### Semaphore & SemaphoreSlim

`Semaphore` class in .NET is a thin wrapper around the OS level _**counting semaphores**_ synchronization object, which can be used to control access to a pool of resources, i.e. limit the number of threads that can access it concurrently. The .NET [Semaphore](https://docs.microsoft.com/en-us/dotnet/api/system.threading.semaphore?view=netcore-2.0) also support named instances for system-wide or inter-process collaboration.

`SemaphoreSlim` is a lightweight version, for using within a single process. It also supports cancellation tokens and async wait. For single application, it is recommended to use the slim version. We'll use the same for our examples.

Semaphore maintains an internal **count** which says how many threads can access the resource concurrently. the .NET `SemaphoreSlim` has a  property `CurrentCount` that says how many new threads can still acquire the semaphore.

Each successful call to `Wait()` decreases the CurrentCount by 1 (every thread should wait-and-enter the semaphore before accessing the shared resource). Every call to `Release()` increases the CurrentCount by 1 (every thread should release it, once they are done using the resource), and a call to `Release(n)` increases the CurrentCount by `n`. When the CurrentCount reaches 0, no more threads can acquire the semaphore, and has to wait until the count increases to 1 or more. When there are multiple threads waiting, and CurrentCount goes above 0, there is no pre-defined order in which threads will enter the semaphore.

The constructor of [SemaphoreSlim](https://docs.microsoft.com/en-us/dotnet/api/system.threading.semaphoreslim?view=netframework-4.7.2) takes two integer arguments, `initialCount` and `maxCount`. The initial count indicates how many threads can immediately acquire the semaphore (Wait) without getting blocked. In other words, this is the value that will be initially set to count (and to CurrentCount). The `maxCount` is optional, and says what is the maximum allowed value of count. If a thread tries to release the semaphore which will take the count (or CurrentCount) to more than maxCount, it'll throw exception. The simple relationship is CurrentCount <= count <= maxCount.

Below code shows a SemaphoreSlim with initial count 1 and max count 3. This is just for demonstration, NOT the right way to write code.

```csharp
//class level semaphore with initialCount = 1, maxCount = 3
static readonly SemaphoreSlim smps = new SemaphoreSlim(1, 3);

internal static void Execute()
{
    var t = new Thread(ReleaseSemaphore);
    t.Start(); //this will release the semaphore in another thread
    //count = CurrentCount = 1, as set with initialCount = 1
    smps.Wait(); //acquire the semaphore immediately, CurrentCount = 0
    //next thread waits for some thread to releases it
    //when ReleaseSemaphore() releases the semaphore, makes CurrentCount = 1
    smps.Wait(2000); //try to acquire with 2 second timeout, CurrentCount = 0
    smps.Release(3); //CurrentCount = 0 + 3 = 3
    smps.Wait(); //CurrentCount = 3 - 1 = 2
    smps.Wait(); //CurrentCount = 2 - 1 = 1
    smps.Release(3); //=> EXCEPTION (CurrentCount = 1 + 3 = 4 > maxCount 3)
}
//just a sample showing a separate thread releasing the semaphore
private static void ReleaseSemaphore()
{
    Thread.Sleep(1000);
    smps.Release(); //count++
}
```

In simple cases, if you want to allow concurrent access of 5 threads, use `SemaphoreSlim(5, 5)`.
{: .notice--success}

Let's look at a standard sample code using semaphore that allows simultaneous access to a _critical section_ of code for 5 threads.

```csharp
static readonly SemaphoreSlim semaphore = new SemaphoreSlim(5, 5);
internal static void AccessSharedResource()
{
    try
    {
        semaphore.Wait();
        //access the shared resource pool
    }
    finally
    {
        semaphore.Release();
    }
}
```

**Important:** It does not matter which thread calls the `Wait()` and which one does `Release()`, they just change the shared semaphore CurrentCount value. It's the responsibility of the program to make sure overall sanity is maintained through the `Wait()` and `Release()` calls from all threads. In general cases, write the code that uses the shared resource pool within a wait-release pair, which must be the only way to access the resource by any thread.
{: .notice--danger}

In the next article, we'll look at **[multithreading with signals](/articles/thread-synchronization-part-three/)**.

#### All posts in the series - Tasks, Threads, Asynchronous

* **[Synchronous to asynchronous in .NET](/articles/sync-to-async-in-dotnet/)**
* **[Basic task cancellation demo in C#](/articles/task-cancellation/)**
* **[How does Async-Await work - Part I](/articles/async-await/)**
* **[How does Async-Await work - Part II](/articles/async-await-2/)**
* **[Multithreading - lock, Monitor & Mutex &#124; Thread synchronization Part I](/articles/thread-synchronization-part-one/)**
* **Multithreading with non-exclusive locks &#124; Thread synchronisation Part II**
* **[Multithreading with signals &#124; Thread synchronisation Part III](/articles/thread-synchronization-part-three/)**