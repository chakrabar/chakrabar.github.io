---
layout: post
title: "Basic task cancellation demo in C#"
excerpt: "A very basic console application that allows user to cancel a task"
date: 2018-02-19
tags: [tech, csharp, dotnet, async, asynchronous, tasks, TPL]
categories: articles
comments: true
share: true
---

Here we'll look at a very basic exmaple of a cancellable `Task`. This is an addition to the post [Synchronous to asynchronous in .NET](/notes/sync-to-async-in-dotnet/). This is a complete runnable code demo of a simple `Console` application that runs a slow process asynchronously. User has an option to _"Press C"_ to cancel the operation.

We'll first have the actual synchronous method `CancellableWork()` that is invoked asynchronously. It accepts a `CancellationToken` and periodically checks if cancellation is requested on the token. If requested, it aborts the operation and throws a `TaskCanceledException`.

The method `CancellableTask()` starts the operation asynchronously and passes a cancellation token. It then retuns the resulting `Task` to the calling method. So, the calling method can pass a `CancellationToken` while invoking this method and request a cancellation later.

**Note:** Just passing a `CancellationToken` and requesting a `Cancel()` doesn't do much by itself. It's up to the actual working logic to check and respond to `IsCancellationRequested`. Also, the code decides whether to throw a `TaskCanceledException` by convention, or not.
{: .notice--warning}

```cs
private void CancellableWork(CancellationToken cancellationToken)
{
    if (cancellationToken.IsCancellationRequested)
    {
        Console.WriteLine("Cancelled work before start");
        cancellationToken.ThrowIfCancellationRequested();
    }

    for (int i = 0; i < 10; i++)
    {
        Thread.Sleep(1000);
        if (cancellationToken.IsCancellationRequested)
        {
            Console.WriteLine($"Cancelled on iteration # {i + 1}");
            //the following lien alone is enough to check and throw
            cancellationToken.ThrowIfCancellationRequested();
        }
        Console.WriteLine($"Iteration # {i + 1} completed");
    }
}

public Task CancellableTask(CancellationToken ct)
{
    return Task.Factory.StartNew(() => CancellableWork(ct), ct);            
}
```

Now we look at the `Main()` method of the application that invokes our `CancellableTask()` method with a cancellation token. If user presses `C`, cancellation is requested to the method.

Then we `Wait()` for the Task to complete. We also do `try-catch` to look for any exception. If we get a `TaskCanceledException` in the `AggregateException` thrown by the task, we know (by convention only), that the task was aborted because of the requested cancellation.

```cs
static void Main(string[] args)
{
    Console.WriteLine("Starting application...");

    CancellationTokenSource source = new CancellationTokenSource();
    var task = new TplTest().CancellableTask(source.Token);

    Console.WriteLine("Heavy process invoked");

    Console.WriteLine("Press C to cancel");
    Console.WriteLine("");
    char ch = Console.ReadKey().KeyChar;
    if (ch == 'c' || ch == 'C')
    {
        source.Cancel();
        Console.WriteLine("\nTask cancellation requested.");
    }

    try
    {
        task.Wait();
    }
    catch (AggregateException ae)
    {
        foreach (var e in ae.InnerExceptions)
        {
            if (e is TaskCanceledException)
                Console.WriteLine("Task canceled exception detected");
            else
                throw;
        }
    }

    Console.WriteLine("Process completed");
    Console.ReadKey();
}
```

This program generates the following output.

```
Starting application...
Heavy process invoked
Press C to cancel

Iteration # 1 completed
Iteration # 2 completed
Iteration # 3 completed
c
Task cancellation requested.
Cancelled on iteration # 4
Task canceled exception detected
Process completed
```