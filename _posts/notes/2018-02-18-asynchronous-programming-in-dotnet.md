---
layout: post
title: "Synchronous to asynchronous in .NET"
excerpt: "Simple conversion of synchronous code to asynchronous with Tasks"
date: 2018-02-20
tags: [tech, csharp, dotnet, async, asynchronous, tasks, threading]
categories: notes
modified: 2018-02-23T22:11:53-04:00
---

#### [1] The problem

Before going into solution and available options, lets understand the problem taht we are trying to address. In general form, this is the problem

> I have some work to be done. But that is a slow process and takes time. How do I do other stuffs while that slow operation is running, rather than just sit idle and wait for it to complete.

With very basic idea of `Operating systems` and `Programming`, we can kind of guess that somehow we have to run the operation on a different thread while keeping the original thread free. Well, in any of the modern tech platforms, we do use that technique to do stuffs `asynchronously`. Asynchronous, in general terms, means multiple things can run without depending on or waiting for each other, and can collaborate later if required.

#### [2] Available options in .NET

In .NET (framework version 4.0 and above) we have three high level options

1. Do the work in new `Thread`
2. Do it with a `Task`
3. Use `async-await`

We'll discuss async-await later, which is a much newer concept. Before making our choice, let's understand what are threads and tasks.

A **`thread`** is a low level construct and roughly represents an actual OS-level thread.

A **`task`** is a .NET abstraction that basically represents _a promise_ of a separate work, that'll be completed in future. Generally a task runs on a separate thread (mostly a managed thread-pool one), managed by the .NET framework.
{: .notice--info}

#### [3] Running some code on a separate thread

First we'll look at a very basic example of running a simple code on a new thread.

```cs
public void SlowMethod()
{
    Thread.Sleep(1500);
    Console.WriteLine("Work completed");
}

public void DoWorkOnThread()
{
    var thread = new Thread(SlowMethod);
    thread.Start(); //starts work on new thread
}
```

Here we have a method `SlowMethod()` that takes time. So, we simply create a new thread by attaching the method to it, and start. 

It works, and a `Thread` being a very raw and low level thing, gives the maximum flexibility to control execution and manage some attached resources. But, creating them in code might cause some serious issues as well. 

* Creating a new thread is pretty resource intensive
* If we keep creating huge number of new threads it might become very difficult for the system to handle. Which can result in the application hanging or the whole system crashing
* Managing and synchronizing them can be pretty complex

So, for most purposes in modern .NET programming, it is reccomended to use `Task` instead.

#### Synchronous to Asynchronous with TPL

Converting a `synchronous` method to `asynchronous`. In simple words, making a normal method to run _'out-of-sync'_ in a separate thread. For that we use `Task.Run()` which takes a `thread` from the `ThreadPool` and runs the method/code on that new thread. As an effect, the main or the original thread can continue to do other stuffs without waiting for that method/code to complete. Let's see a simple example

```cs
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

Well, what if I want to do something only after that `SlowMethod()` is done? Then you have to create a **continuation** of that task. The `Task.Run()` actually returns a `Task` which is kind of a context that can give details of the underlying work that is being executed. It can tell whether the work is completed or not, if that failed etc. and can return any result once that is completed in `task.Result`.

Let's create a **continuation** that will return the string result when `SlowMethod()` is completed.

```cs
//Updated DoAsyncWorkMethod
public void DoAsyncWorkMethod()
{
    var task = Task.Run(() => SlowMethod());
    task.ContinueWith(t => UpdateInDB(t.Result));
    //the `t` argument is same as `task`
    //so execution will continue here normally
    //when task completed, it'll be updated in DB
}
```

**Note:** Using `task.Result` directly makes the process synchronous, that the current thread waits for the task to complete.
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

#### Handling exception in asynchronous

How do we handle `Exception` in methods that do asynchronous work? Do we simply wrap them in `try-catch` to handle them? Well, that doesn't work. 

If an snchronous method throws exception, that is being run on another thread, our current thread cannot simply get them. That exception doesn't effect the current thread directly, but the application actually failes internally to do what it was supposed to do. Very unreliable and bad, right? It fails, but user doesn't get to know! 

So, the following method `DoAsyncWork_2()` **DOES NOT work as expected, and it cannot catch the exception** while the program failed internally! Execution never reaches the `catch` block.

```cs
public string SlowBuggyMethod()
{
    Thread.Sleep(1500);
    throw new Exception("It failed!");
}

public void DoAsyncWork_2()
{
    try
    {
        Task.Run(() => SlowBuggyMethod());
    }
    catch (Exception)
    {
        Console.WriteLine("Exception caught in DoAsyncWork_2");
    }            
}
```

<u>The correct way, check fault on continue</u>

If the asynchronous method/code fails, the exception details are attached to the returned Task. To handle the exception, we need to handle the exception by getting a handle back from the task. We can check the `IsFaulted` and `Exception` properties on continuation to see if anything went wrong, or handle on `Wait()` or `WaitAll()`. Alternatively, if we want to execute some code only if it worked/failed (e.g. log the exception), `ContinueWith()` has an overload that accepts a `TaskContinuationOptions` that can specify when to run the continuation code. 

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

//with Wait()
public void DoAsyncWork_5()
{
    var task = Task.Run(() => SlowBuggyMethod());
    try
    {
        task.Wait();
    }
    catch (Exception)
    {
        Console.WriteLine("Exception: " + task.Exception.Message);
    }
}
```

**Note:** The exception returned by `task.Exception` is of type `AggregateException`, that actually wraps one or more errors that occur during the task execution. It has a collection of actual exceptions in `InnerExceptions` property. If we know there can be just one exception (we are very sure what all things that code is doing), we can simply do task.Exception.InnerExceptions[0].Message;
{: .notice--info}

In the `DoAsyncWork_5()` example we use a `task.Wait()` call. This actually halts the current thread and waits for the task to complete. Just like above (sample) code, if we do a `Wait()` immediately after running a task, that'll basically behave as a synchronous work. Ideally, you'll do other stuffs before waiting for the task. Also, if the task throws exception, that can be cauth on the `Wait()`. To wait for multiple tasks to complete, use `WaitAll(tasks)`.

#### [7] Tasks, threads and context

When a new task is run, it is managed and synchronized by the CLR. 

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
    //thread #1 continues
}
```

