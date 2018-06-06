---
layout: post
title: "Thread synchronization with non-exclusive locks | Thread synchronisation Part II"
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

#### ReaderWriteLock

The [ReaderWriterLock](https://docs.microsoft.com/en-us/dotnet/api/system.threading.readerwriterlock?view=netframework-4.7.2) provides a locking mechanism that supports single writer and multiple readers to a shared resource. It provides `AcquireReaderLock` and `AcquireWriterLock` methods, where multiple threads can aquire reader lock simultaniously, but only one thread can have writer lock at a time and no thread can read at that time.

> Readers and writers are queued separately. When a thread releases the writer lock, all threads waiting in the reader queue *at that instant* are granted reader locks; when all of those reader locks have been released, the next thread waiting in the writer queue, if any, is granted the writer lock, and so on. In other words, `ReaderWriterLock` alternates between a collection of readers, and one writer. - from MSDN

While read is happening, new read threads queue up rather than joining the current batch of read threads. This prevents the write threads from waiting indefinitely.

`ReaderWriterLock` works much more efficiently than `lock` or `Monitor` when _many more_ reads are happening on a resource compared to writes (e.g. a configuration file), and writes are slow.

```csharp
static ReaderWriterLock rwl = new ReaderWriterLock();
static int resource = 0; //shared resource

static void ReadSharedResource(int timeOutMilliSec) {
    try {
        rwl.AcquireReaderLock(timeOutMilliSec);
        try {
            Console.WriteLine($"Current resource value on read {resource}");
        }
        finally {
            rwl.ReleaseReaderLock();
        }
    }
    catch (ApplicationException) { //timed out before getting lock
        Console.WriteLine("Reader lock timed out");
    }
}
```

**Important:** For all new developments, it's recommended to use `ReaderWriterLockSlim` instead of `ReaderWriterLock`. It has same features and as it has [1] simpler rules for recursion control [2] simplified up/down-grade of locks [3] avoids many potential deadlocks [4] performs better.
{: .notice--warning}

#### ReaderWriterLockSlim

`ReaderWriterLockSlim` has similar features of `ReaderWriterLock` and the additional benefits mentioned above. Also code is more readable with [ReaderWriterLockSLim](https://docs.microsoft.com/en-us/dotnet/api/system.threading.readerwriterlockslim?view=netframework-4.7.2). It's basically a _slim_ or _lightweight_ version of ReaderWriterLock, which should be preferred.

```csharp
public class SafeReadWrite {
  static ReaderWriterLockSlim rwLockSlim = new ReaderWriterLockSlim();
  static int resource = 0; //shared resource
  //gets reader lock without timeout
  static void ReadSharedResource() {
      rwLockSlim.EnterReadLock();
      try {
          Console.WriteLine($"Current resource value on read {resource}");
      }
      finally {
          rwLockSlim.ExitReadLock();
      }
  }
  //tries to get writer lock with a timeout
  static void WriteToSharedResource(int timeOutMilliSec) {
      if (rwLockSlim.TryEnterWriteLock(timeOutMilliSec)) {
          try {
              resource = 100;
              Console.WriteLine($"Current resource value after write {resource}");
          }
          finally {
              rwLockSlim.ExitWriteLock();
          }
      }
      else { //timeOutMilliSec elapsed, could not get lock
          Console.WriteLine("Writer lock timed out");
      }
  }
  //ReaderWriterLockSlim implements IDisposable
  ~SafeReadWrite() { //do not forget to dispose
    if (rwLockSlim != null) rwLockSlim.Dispose();
  }
}
```

It laso provides methods `EnterUpgradeableReadLock` and `ExitUpgradeableReadLock` to upgrade a reader lock to writer, and bring back avoiding possible deadlocks. When this is used, the thread upgrading to writer lock gets placed at writer queue.

#### Semaphore & SemaphoreSlim

`Semaphore` class in .NET is a thin wrapper around the OS level counting semaphores synchronization object, which can be used to controll access to a pool of resources, i.e. limit the number of threads that can access it concurrently. .NET [Semaphore](https://docs.microsoft.com/en-us/dotnet/api/system.threading.semaphore?view=netcore-2.0) also suppport named instances for system-wide or inter-process collaboration.

`SemaphoreSlim` is a lightweight version, for using within a process. Also supports cancellation tokens and async wait. For single application, it is recommended to use the slim version. We'll use the same for our examples.

SemaphoreSlim maintains an internal **count** which says how many threads can access the resource concurrently. It has a  property `CurrentCount` that says how many new threads can still take the semaphore.

Each successful call to `Wait()` decreases the CurrentCount by 1 (every thread should wait-and-enter the semaphore before accessing the shared resource). Every call to `Release()` increases the CurrentCount by 1 (every thread should release it, once they are done using the resource), and a call to `Release(n)` increases the CurrentCount by n. When the CurrentCount reaches 0, no more threads can take the semaphore, and has to wait until the count increases to 1 or more. When there are multiple threads waiting, and CurrentCount goes above 0, there is no pre-defined order in which threads will enter the semaphore.

The constructor of [SemaphoreSlim](https://docs.microsoft.com/en-us/dotnet/api/system.threading.semaphoreslim?view=netframework-4.7.2) takes two integer arguments, `initialCount` and `maxCount`. The initial count indicates how many threads can immediately take the semaphoore (Wait) without getting blocked. In other words, this is the value that will be initially set to count (and to CurrentCount). The `maxCount` is optional, and says what is the maximum allowed value of count. If a thread tries to release the semaphore which will take the count (or CurrentCount) to more than maxCount, it'll throw exception. CurrentCount <= count <= maxCount

In simple cases, if you want to allow concurrent access of 5 threads, use `SemaphoreSlim(5, 5)`. Below code shows a SemaphoreSlim with initial count 1 and max count 3. This is just for demonstration, NOT the right way to write code.

```csharp
//class level semaphore with initialCount = 1, maxCount = 3
static readonly SemaphoreSlim smps = new SemaphoreSlim(1, 3);

internal static void Execute()
{
    var t = new Thread(ReleaseSemaphore);
    t.Start(); //get another thread to release the semaphore
    //count = CurrentCount = 1, as set with initialCount = 1
    smps.Wait(); //take the semaphore immediately, CurrentCount = 0
    //next thread waits for some thread to releases it
    //ReleaseSemaphore() releases semaphore, making CurrentCount = 1
    smps.Wait(2000); //try to take with 2 second timeout, CurrentCount = 0
    smps.Release(3); //CurrentCount = 3
    smps.Wait(); //CurrentCount = 2
    smps.Wait(); //CurrentCount = 1
    smps.Release(3); //=> EXCEPTION (CurrentCount = 4 > maxCount 3)
}

private static void ReleaseSemaphore()
{
    Thread.Sleep(1000);
    smps.Release(); //count++
}
```

**Note:** It does not matter which thread calls the `Wait()` and which one does `Release()`, they just change the shared semaphore CurrentCount value. It's the reponsibility of the program to make sure overall sanity is maintained through the `Wait()` and `Release()` calls from all threads. In general cases, write the code that uses the shared resource pool within a wait-release pair, which has to be the only way to access the resource by any thread.
{: .notice--info}