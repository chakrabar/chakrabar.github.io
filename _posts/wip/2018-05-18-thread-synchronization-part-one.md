---
layout: post
title: "Thread synchronization basics in .NET - part I"
excerpt: "Some common cases of bare basic thread synchronization"
date: 2018-05-18
tags: [thread, synchronization, lock, monitor, semaphore, deadlock]
categories: articles
image:
  feature: posts/threads/ogqcorp.jpg
  credit: ogqcorp.com
  creditlink: http://bgh.ogqcorp.com/share/h/SmP2O
comments: true
share: true
published: false
---

**Note:** This article is currently incomplete & in-progress. It'll be updated soon.
{: .notice--danger}

# What are threads

* Basic unit of execution
* Has it's own memory and gets alloted processor time
* Sequence of instructions managed independently by scheduler
* Thread schedulers are part of OS
* A process (executing program) are run on one or more threads - which executes independently, but can share resources within or across the process

* CLR delegates the thread scheduling to the OS
* It gives all threads it's own stack (threads get own copy of all their local variables) - but static variables are not, they are shared across the whole application
* Threads from same process also share the same heap memory, while different processes are completely isolated

```cs
//using System.Threading;
public void ASlowMethod()
{
    Thread.Sleep(1500);
    Console.WriteLine("Work completed");
}

public void DoWorkOnThread()
{
    var thread = new Thread(ASlowMethod);
    thread.Start(); //starts work on new worker thread
    //continue working on current, main thread
}
```

#### Multi-threading

OS does timeslicing and allocates CPU time to each thread in turn
The slice time (on Windows) is generally close to 10 milli seconds
The overhead of this context switching is in the order of micro seconds
When a thread is suspended from execution (e.g. due to timeslicing), it's called _"preempteed"_ thread

#### Thread overheads

* Creating a new thread takes about a few hundred milliseconds (roughly), for the new stack allocation, alloting resources for spawning etc.
* Also uses around 1MB of memory per thread ([default](https://msdn.microsoft.com/en-us/library/windows/desktop/ms686774%28v=vs.85%29.aspx?f=255&MSPPError=-2147217396) in Windows)

#### ThreadPool

Maintains a pool of worker threads, which are assigned to do _tasks_ on demand. Once the task is completed, the thread is taken back to the pool and used for other tasks as required.

* Keeps a pool of worker threads
* All threads are [background](https://msdn.microsoft.com/en-us/library/system.threading.threadpool.aspx) (stops abruptly if main thread terminates)
* New work is allocated to idle threads, and are returned to the pool once done
* If there is more work than maximum thread limit, they are queued
* Can be used via a `Task` or `QueueUserWorkItem()`

**Note:** Threads do not throw exceptions across other threads. To handle exception, it has to be caught on the same context.
{: .notice--warning}

#### Tasks

* Designed to return value
* Tasks are run on the managed ThreadPool, and _generally_ run as background thread
* Almost always use a `Task` over a `Thread`. If you need a dedicated thread for a long running task, still prefer using a Task with `LongRunning` of `TaskCreationOptions`, for the sake of consistency. Pure threads might be used for very low-level uses, e.g. custom schedulers etc.
* See the [Synchronous to asynchronous in .NET](/articles/sync-to-async-in-dotnet/) post for more details on Tasks.
* Can easliy `Wait` for one or more Tasks and `Continue` with other Tasks
* Supports parent-child relationships and aggregated exceptions as `AggregateException`
* Tasks cancellation is easier with cancellation tokens, see [task cancellation example](/articles/task-cancellation/)

#### Thread synchronization

In a multi-threaded environment, multiple threads can can speed up works and keep main thread responsive (e.g. UI thread in Windows applications). Insuch scenarios, they can also access different resources like files, network connections, memory etc as per the application needs. If done incorrectly, what may happen is multiple threads trying to use and/or update same resource at the same time, unaware of each other. This can result in unpredictable and incocnsistent results.

So, in multi-hreaded applications, the threads needs to be synchronized, so that they do not work on (most importantly, update) the same resource at the same time. There are [several ways to synchronize thread](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/threading/thread-synchronization) in .NET environment.

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
        3. ReaderWriterLock
        4. ReaderWriterLockSlim
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

#### Monitor & lock

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

#### Mutex

`Mutex` (from _mutually exclusive_) is similar to `Monitor` in that, it allows only one thread to execute a block of code, but unlike Monitor, it can work across processes. Monitor is more of a .NET construct, whereas Mutex is a wrapper over OS level Win32 construct.

Generally a string name is used to identify a [Mutex](https://docs.microsoft.com/en-us/dotnet/api/system.threading.mutex?view=netframework-4.7.2) in inter-process thread synchronization, and that can be across the machine. It can be used in intra-process synchronization as well, but Monitor is prefferd as it makes better use of resources.

```csharp
static Mutex mutex = new Mutex();

if (mutex.WaitOne(2000)) //wait to get mutext with timeout of 2 seconds
{
    Console.WriteLine("Entered mutex");
    //do the work
    Console.WriteLine("Releasing mutex");
}
else
    Console.WriteLine("Timeout expired");
```

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

Continue to **[next module](/articles/thread-synchronization-part-two/)**.

###### Other posts in the series - Tasks, Threads, Asynchronous

* **[Synchronous to asynchronous in .NET](/articles/sync-to-async-in-dotnet/)**
* **[Basic task cancellation demo in C#](/articles/task-cancellation/)**
* **[How does Async-Await work - Part I](/articles/async-await/)**
* **[How does Async-Await work - Part II](/articles/async-await-2/)**
* **[Basic thread synchronization with examples - Part I](/articles/thread-synchronization-part-one/)**
* **[Basic thread synchronization with examples - Part II](/articles/thread-synchronization-part-two/)**

#### References

* [Synchronization in Windows](https://msdn.microsoft.com/en-us/library/ms686353.aspx)
* [Processes, Threads, and Jobs in the Windows](https://www.microsoftpressstore.com/articles/article.aspx?p=2233328&seqNum=7)
* [Runtime object creation in CLR](/assets/downloads/Objects_MSDN_May2005.pdf)