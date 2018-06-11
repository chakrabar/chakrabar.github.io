---
layout: post
title: "Synchronous to asynchronous in .NET"
excerpt: "Simple conversion of synchronous code to asynchronous with Tasks"
date: 2018-02-18
tags: [tech, csharp, dotnet, async, asynchronous, tasks, threading, TPL]
categories: articles
comments: true
share: true
modified: 2018-02-23T22:11:53-04:00
---

Here, we'll look at how we can make our .NET application more efficient and better responsive to users.

#### [1] The problem

Before going into solution and available options, lets understand the problem that we are trying to address. In general form, this is the problem

> I have some work to be done. But that is a slow process and takes time. How do I do other stuffs while that slow operation is running, rather than just sit idle and wait for it to complete.

With very basic idea of `Operating systems` and `Programming`, we can kind of guess that somehow we have to run the operation on a different thread while keeping the original thread free. Well, in any of the modern tech platforms, we do use that technique to do stuffs `asynchronously`. Asynchronous, in general terms, means multiple things can run without depending on or waiting for each other, and can collaborate later if required.

----

There are three different but very similar and related terms in computing: `concurrent`, `parallel` & `asynchronous`. **Concurrent** basically means multiple works being done in overlapping time (e.g. multiple threads sharing same processor core), which may or may not be parallel. **Parallel** specifically means multiple things are being done actually at the same point of time (e.g. threads in different processor cores running simultaneously). **Asynchronous** means some operation is started out-of-sync in time, which will run on its own and notify when completed. (Which may involve threads running in the same or different core, or even in different systems!)

The concepts discussed below may apply to all three. Here, we'll use the term `asynchronous` only as that'll be the our main intention.

----

#### [2] Available options in .NET

In .NET (framework version 4.0 and above) we have three high level options

1. Do the work in new `Thread`
2. Do it with a `Task`
3. Use `async-await`

We'll discuss async-await later, which is a comparatively newer concept. Before making our choice, let's understand what are threads and tasks.

A **`thread`** is a low level construct and roughly represents an actual OS-level thread.
{: .notice--info}
A **`task`** is a .NET abstraction that basically represents _a promise_ of a separate work, that'll be completed in future.
{: .notice--info}

A `Task<T>` is nothing but a task that comes with _promise_ of returning a value of type `T` when the task completes.

**Note:** Just using a `Task` in code does not mean there are separate _new_ threads involved. Generally when using `Task.Run()` or similar constructs, a task runs on a separate thread (mostly a managed thread-pool one), managed by the .NET CLR. But, that depends on the actual implementation of the task.

#### [3] Running some code on a separate thread

First we'll look at a very basic example of running a simple code on a new thread.

```cs
//using System.Threading;
public void SlowMethod()
{
    Thread.Sleep(1500);
    Console.WriteLine("Work completed");
}

public void DoWorkOnThread()
{
    var thread = new Thread(SlowMethod);
    thread.Start(); //starts work on new thread
    //continue working on current thread
}
```

Just for understanding, a `Thread` is not a method, so it does not return a value. But if we want a result back after a thread has completed its work, there are few ways of doing it. Following code shows an example using a `closure`.

```cs
public void ValueReturningThread()
{
    string result = null;
    Thread thread = new Thread(() => {
        Thread.Sleep(5000);
        result = "Thread work completed";
    });
    thread.Start();
    //do other stuffs
    thread.Join(); //wait for thread to terminate
    Console.WriteLine(result);
}
```

In our example we have a method `SlowMethod()` that takes time. So, we simply create a new thread by attaching the method to it, and start. 

It works, and a `Thread` being a very raw and low level thing, gives the maximum flexibility to control execution and manage some attached resources etc. Also the work on the newly created thread starts immediately. But, creating them in code might cause some serious issues as well. 

* Creating a new thread is pretty resource intensive. Creating, starting & stopping threads take time & consumes resources (on the other hand, ThreadPool threads are not created or terminated once done. Rather they are kept to be re-used again)
* If we keep creating huge number of new threads it might become very difficult for the system to handle. If there are much more threads than CPU cores, the OS needs to do frequent context switches, which is heavy. That can result in the application hanging or even the whole system crashing
* Managing and synchronizing them can be pretty complex, from correct coding to actual execution stand point

So, for most purposes in modern .NET programming, it is recommended to use `Task` instead (which does not necessarily mean "new" threads). Tasks also inherently support multi-core.

#### [4] Synchronous to Asynchronous with Task

Converting a `synchronous` method to `asynchronous` in simple words, is making a normal method to run _'out-of-sync'_ in. For that we can use `Task.Run()` which gives the work item to the `ThreadPool`, which takes a `thread` from the `ThreadPool` and runs the method/code on that thread as per schedule and availability. As an effect, the main or the original thread can continue to do other stuffs without waiting for that method/code to complete. 

Now as we discussed the problems of manually creating threads above, it is much better option to run those code as tasks instead. So when a task is started, it grabs an available thread from the `thread-pool` and runs the operation on that. Once completed, the thread is released back to the thread-pool. 

----

**Note:** This differs if a task is marked to be a **`LongRunning`** task. For a long-running task, a new thread is used. A long running (0.5 seconds or more) operation should actually be run as `LongRunning` as that'll not block thread pool threads, which can efficiently run smaller tasks and rotate. There are some good QA on StackOverflow about when a Task should be marked LongRunning [here](https://stackoverflow.com/questions/37607911/when-to-use-taskcreationoptions-longrunning) and [here](https://stackoverflow.com/questions/25833054/what-does-long-running-tasks-mean). To create a long running task

```cs
var cancellationTokenSource = new CancellationTokenSource();
Task.Factory.StartNew(() => SomeSlowMethod(),
    cancellationTokenSource.Token,
    TaskCreationOptions.LongRunning, //this
    TaskScheduler.Default);
```

----

Let's see a simple example of creating a task out of a simple method. This was we can run any code asynchronously.

```cs
//System.Threading.Tasks

//a dummy method that simply wastes 1.5 sec before doing work
public string SlowMethod()
{
    Thread.Sleep(1500); //sleep current thread for 1.5 sec
    return "Done";
}

//Simply making it asynchronously, on a separate thread
public void DoAsyncWorkMethod()
{
    Task.Run(() => SlowMethod()); //will run on separate thread
    //execution can continue from here on original thread
}
```

**Note:** Doing a `new Thread(...).Start()` actually creates and starts a new `Thread`. But doing a `Task.Run(...)` simply _queues_ the work on `ThreadPool`. Then ThreadPool manages the work, assigns the work to a thread from the pool when available.
{: .notice--info}

**Note:** Important thing to remember here is, the way `Task` and modern `ThreadPool` are created, they are pretty much _multi-core aware_. That means, if there are multiple CPU available in the system, tasks will try to utilize them in an efficient way.
{: .notice--info}

#### [5] Starting new Task choices

Now that we are mostly convinced that we want to use Tasks to do stuffs asynchronously, we are again presented with multiple options on how to start a new Task. There are basically 3 options

1. new Task(Action).Start();
2. Task.Factory.StartNew(Action);
3. Task.Run(Action);

They all seem to do pretty much the same thing! Which one to use? 

**TL;DR** Stick to `Task.Run()` unless you need specific customization like `LongRunning` process or a non-default `TaskScheduler` or synchronizing child tasks with parent etc. If those are required, use `Task.Factory.StartNew()`.
{: .notice--success}

First, let's understand one thing, **a Task only executes once** and should be scheduled only once. This needs synchronization to avoid race condition where multiple threads try to start a task (think `new Task()`). Now, as `Task.Factory.StartNew` first starts the task then returns a reference, this is safe and saves the synchronization cost (internal to CLR). The reason is, once a task has been started, any call to that task again to `Start()` will simply fail. So, use `Task.Factory.StartNew()` over `new Task().Start()`.

There can some cases though, where creating new Task might be beneficial. One example being, need to derive from the `Task` type. [This](https://blogs.msdn.microsoft.com/pfxteam/2010/06/13/task-factory-startnew-vs-new-task-start/) MSDN article explains with great examples.

Now,

```cs
Task.Run(SomeAction);
//is equivalent to
Task.Factory.StartNew(SomeAction,
    CancellationToken.None,
    TaskCreationOptions.DenyChildAttach,
    TaskScheduler.Default);
```

So `Task.Run()` is simply a short-hand with default parameters that works fine in most of the cases when we simply want to off load some activity to a background (thread-pool) thread. If you need specific customization like `LongRunning` process or a non-default `TaskScheduler` or synchronizing child tasks with parent etc. then go for `Task.Factory.StartNew()`. There is another very interesting [MSDN article](https://blogs.msdn.microsoft.com/pfxteam/2011/10/24/task-run-vs-task-factory-startnew/) that explains this topic and how this paved way for the newer keyword `await`.

One <u>interesting thing to note</u> is, `Task.Factory.StartNew()` and `Task.Run()` can both accept a `Func<Task<T>>`. And in that case, `Task.Factory.StartNew()` returns a `Task<Task<T>>` whereas the `Task.Run()` returns a `Task<T>`! This is `Task.Run()` internally implements `.Unwrap()` extension method internally! So, the `Run()` method is also easier to use.

```cs
var outTask1 = Task.Factory.StartNew(() => {
    Task<int> innerTask = Task.Run(() => 100);
    return innerTask;
});
//here outTask1 is Task<Task<int>>

var outTask2 = Task.Run(() => {
    Task<int> innerTask = Task.Run(() => 100);
    return innerTask;
});
//outTask2 is Task<int>

var outTask3 = Task.Factory.StartNew(() => {
    Task<int> innerTask = Task.Run(() => 100);
    return innerTask;
}).Unwrap();
//here outTask3 is Task<int>
```

#### [6] Task continuation

Well, what if I want to do something only after that `SlowMethod()` is done? Then you have to create a **continuation** of that task. The `Task.Run()` actually returns a `Task` which is kind of a context that can give details of the underlying work that is being executed. It can tell whether the work is completed or not, if that failed etc. and can return any result once that is completed in `task.Result`.

Let's create a **continuation** that will return the string result when `SlowMethod()` is completed.

```cs
//Updated DoAsyncWorkMethod
public void DoAsyncWorkMethod()
{
    var task = Task.Run(() => SlowMethod());
    task.ContinueWith(t => UpdateInDB(t.Result));
    //the `t` argument is same as `task`
    //method execution will continue here normally
    //when task completed, it'll be updated in DB
}
```

**Note:** Using `task.Result` directly after starting a task makes the process synchronous, that the current thread waits for the task to complete.
{: .notice--warning}

```cs
//INCORRECT DoAsyncWorkMethod
public void DoAsyncWorkMethod()
{
    var task = Task.Run(() => SlowMethod());
    string result = t.Result;
    //execution WILL WAIT here on current thread
    UpdateInDB(result);
}
```

#### [7] Handling exception in Task

How do we handle `Exception` in methods that do asynchronous work? Do we simply wrap them in `try-catch` to handle them? Well, that doesn't work.

If an asynchronous method (e.g. a `Task`) throws exception, that is being run on another thread, our current thread doesn't get to know about that. That exception doesn't effect the current thread directly, but the application actually fails internally to do what it was supposed to do. Very unreliable and bad, right? It fails, but user doesn't get to know why!

So, the following method `DoAsyncWork_2()` **DOES NOT work as expected, and it cannot catch the exception** while the program failed internally! Execution never reaches the `catch` block.

```cs
public string SlowBuggyMethod()
{
    Thread.Sleep(1500);
    throw new Exception("It failed!");
}

public void DoAsyncWork_2()
{
    //thread_1
    try
    {
        Task.Run(() => SlowBuggyMethod()); //will run on thread_2
        //thread_1 continues
    }
    catch (Exception) //catches exception on thread_1
    {
        Console.WriteLine("Exception caught in DoAsyncWork_2");
    }
}
```

**<u>The correct way, check fault on Task wait OR continuation</u>**

If the asynchronous method/code fails, the task terminates & the exception details are attached to the returned `Task`. To handle the exception, we need to get back the exception details by getting a handle back from the task. We can check the `IsFaulted` and `Exception` properties on continuation to see if anything went wrong, or handle on `Wait()` or `WaitAll()` or `.Result`. Alternatively, if we want to execute some code only if it worked/failed (e.g. log the exception), `ContinueWith()` has an overload that accepts a `TaskContinuationOptions` that can specify when to run the continuation code.

```cs
public void DoAsyncWork_3()
{
    var task = Task.Run(() => SlowBuggyMethod());
    task.ContinueWith(t =>
    {
        if (task.IsFaulted)
            Console.WriteLine("Exception: " + t.Exception.Message);
        else
            Console.WriteLine("Completed successfully!");
    });
}

//with TaskContinuationOptions
public void DoAsyncWork_4()
{
    var task = Task.Run(() => SlowBuggyMethod());
    task.ContinueWith(t =>
        Console.WriteLine("Exception: " + t.Exception.Message), 
        TaskContinuationOptions.OnlyOnFaulted);
}

//with Wait() or .Result
public void DoAsyncWork_5()
{
    var task = Task.Run(() => SlowBuggyMethod());
    try
    {
        task.Wait(); //or task.Result;
    }
    catch (AggregateException ae)
    {
        Console.WriteLine("Exception: " + ae.InnerException.Message);
    }
}
```

**Note:** The exception returned by `task.Exception` is of type `AggregateException`, that actually wraps one or more errors that occur during the task execution. It has a collection of actual exceptions in `InnerExceptions` property. If we know there can be just one exception (we are very sure what all things that code is doing), we can simply use `InnerExceptions[0]`.
{: .notice--info}

In the `DoAsyncWork_5()` example we use a `task.Wait()` call. This actually halts the current thread and waits for the task to complete. Just like above (sample) code, if we do a `Wait()` immediately after running a task, that'll basically behave as a synchronous work. Ideally, you'll do other stuffs before waiting for the task. Also, if the task throws exception, that can be caught on the `Wait()`. To wait for multiple tasks to complete, use `WaitAll(tasks)`.

#### [8] Tasks & threads

Task is an abstraction that, most of the times, uses some thread to run the operation asynchronously. So when we are working with Tasks, multiple threads are involved, though we do not generate them on our own. Let's look at a sample code from above and see how threads are involved here.

When a new task is run, it is managed and synchronized by the CLR. The new task (generally) runs on a `ThreadPool` thread. And even the continuation task is assigned to another thread from the thread pool. So, in the following code, 3 threads would get involved.

**Note:** One can use the `ManagedThreadId` property of static `Thread.CurrentThread` to see the CLR thread ID of the currently executing thread. Useful for debugging.
{: .notice--info}

```cs
public void DoAsyncWork_3()
{
    //the original thread #1
    var task = Task.Run(() => SlowBuggyMethod()); //future thread #2
    task.ContinueWith(t =>
    {
        //future thread #3
        if (task.IsFaulted)
            Console.WriteLine("Exception: " + t.Exception.Message);
        else
            Console.WriteLine("Completed successfully!");
    });
    //thread #1 continues asynchronously
}
```

There is a way to request execute the continuation task on the captured context of the initial task. That means, CLR will try to run them on the same thread. This might be useful in some UI-thread scenarios.

```cs
public void DoAsyncWork_6()
{
    //the original thread #1
    var task = Task.Run(() => SlowBuggyMethod()); //future thread #2
    task.ConfigureAwait(true) //try continuation on captured context
        .GetAwaiter()
        .OnCompleted(() =>
        {
            //continue on thread #2
            if (task.IsFaulted)
                Console.WriteLine("Exception: " + task.Exception.Message);
            else
                Console.WriteLine("Final Thread: " + Thread.CurrentThread.ManagedThreadId);
        });
    //thread #1 continues asynchronously
}
```

Setting `ConfigureAwait(false)` explicitly tells to try run the continuation on a separate thread that is available. This promotes higher degree of concurrency. 

#### [9] Cancellation of Tasks

To be able to cancel a task, we need to start a task with a cancellation token. Then cancellation can be requested on the token source later. 

**Note:** Just requesting a cancellation does not guarantee and immediate cancellation of the task, or even any response at all. It is up to the underlying task code to periodically check for cancellation request and respond accordingly.
{: .notice--warning}

```cs
private void CancellableWork(CancellationToken cancellationToken)
{
    for (int i = 0; i < 10; i++)
    {
        Thread.Sleep(1000);
        //check cancellation requested and act
        cancellationToken.ThrowIfCancellationRequested();
        //else, do normal work
        Console.WriteLine($"Iteration # {i + 1} completed");
    }
}
public Task CancellableTask(CancellationToken ct)
{
    return Task.Factory.StartNew(() => CancellableWork(ct), ct);            
}

//The calling method
CancellationTokenSource source = new CancellationTokenSource();
var task = new CancellableTask(source.Token);

Console.WriteLine("Press C to cancel");
char ch = Console.ReadKey().KeyChar;
if (ch == 'c' || ch == 'C')
{
    source.Cancel();
}

try
{
    task.Wait();
}
catch (AggregateException ae)
{
    //handle exception
}
```

See the [Basic task cancellation demo in C#](/articles/task-cancellation/) for a complete runnable code demo of a simple `Console` application that runs a slow process asynchronously, and gives the user an option to cancel the operation.

#### [10] Bundling of Tasks

Sometimes we create multiple tasks to do multiple things, then we need to monitor them collectively. The `Task` comes with some handy ways to bundle multiple tasks into a single `Task` and work on that. Some of those methods are

* `Task.WhenAll(Task[])` -> `Task`
* `Task.WhenAny(Task[])` -> `Task<Task>`

They return a waitable `Task` that completes as per chosen condition. `WhenAll()` completes when all the tasks are complete while `WhenAny()` returns when any of the tasks are complete and gives back the completed task. 

**<u>Parallel - For, Foreach, Invoke, LINQ</u>**

If you just want to do some work in _parallel_, you can use the static methods from `Parallel` class or the parallel `Linq` extension `AsParallel()`. They all let's you run some code - which the CLR will try to run in parallel. Internally they use `Tasks` in optimized way. So, they can pretty much <u>run in parallel in different CPU cores</u> if available.

Some simple examples

```cs
//A slow method that takes a second to write the value
public void SlowTyper(int value)
{
    Thread.Sleep(1000);
    Console.WriteLine($"Value: {value}");
}

//demo methods
public void ParallelFor()
{
    var k = Parallel.For(1, 6,
        (idx) => SlowTyper(idx));
}

public void ParallelForEach()
{
    Parallel.ForEach(Enumerable.Range(1, 5),
        (idx) => SlowTyper(idx));
}

public void ParallelInvoke()
{
    Parallel.Invoke(() => SlowTyper(1), () => SlowTyper(2),
        () => SlowTyper(3), () => SlowTyper(4), () => SlowTyper(5));
}

public void PLinq()
{
    Enumerable.Range(1, 5)
        .AsParallel()
        .ForAll(i => SlowTyper(i));
}
```

They all generally take much lesser time that general sequential approach (e.g. just above 1 second rather than 5 seconds).

----

And since the `C# 5` was released, we have a cleaner way to do asynchronous programming using the [Async-Await](/articles/async-await/). Also see the [code demo of cancellation](/articles/task-cancellation/).

###### All posts in the series - Tasks, Threads, Asynchronous

* **Synchronous to asynchronous in .NET**
* **[Basic task cancellation demo in C#](/articles/task-cancellation/)**
* **[How does Async-Await work - Part I](/articles/async-await/)**
* **[How does Async-Await work - Part II](/articles/async-await-2/)**
* **[Multithreading - lock, Monitor & Mutex &#124; Thread synchronization Part I](/articles/thread-synchronization-part-one/)**
* **[Multithreading with non-exclusive locks &#124; Thread synchronization Part II](/articles/thread-synchronization-part-two/)**
* **[Multithreading with signals &#124; Thread synchronization Part III](/articles/thread-synchronization-part-three/)**
* **[Non-blocking multithreading & concurrent collections &#124; Thread synchronization Part IV](/articles/thread-synchronization-part-four/)**