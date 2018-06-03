---
layout: post
title: "How does Async-Await work - Part I"
excerpt: "A simple explanation of Async-Await in .NET"
date: 2018-05-28
tags: [async, await, asynchronous, threading, synchronization, tasks, concurrency]
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

#### Few basic scenarios

#### Async-await in `ASP.NET`

Like any other code, async-await can be used in `ASP.NET` web programming as well. Let's look at an example with `MVC controller`. Consider we have the follwoing synchronous code, which makes a call to an external website and returns the content.

```cs
public IActionResult GetData()
{
    using (var client = new HttpClient())
    {
        var response = client.GetAsync("https://arghya.xyz").Result;
        return Content(response.Content.ReadAsStringAsync().Result, "text/html");
    }
}
```

We can simply convert it into asynchronous making the basic changes - add `async` in method signature, return a `Task<T>`, use `await` to get results from asynchronous calls, and ideally change method name to add `Async` suffix.

```cs
public async Task<IActionResult> GetDataAsync()
{
    using (var client = new HttpClient())
    {
        var response = await client.GetAsync("https://arghya.xyz");
        var content = await response.Content.ReadAsStringAsync();

        return Content(content, "text/html");
    }
}
```

Like any other `async` method, there won't be much of a difference in total execution time, BUT it helps in keeping the threads free.

#### Multiple await

Can you use multiple await inside an `async` method? Absolutely. The compiler will create multiple states in the state-machine based on them, and at runtime the execution will suspend and resume at all `await` points. Look at the simple code below, which uses two `await` to get two different results, then combines them and returns the desired final result.

```cs
public static async Task<string> GetPersonDetailsAsync()
{
    int id = await GetIdAsync();
    string name = await GetNameAsync();
    return $"Person Id:{id}, Name:{name}";
}
```

This works fine. But we can improve it. In the above code, it executes the tasks one-by-one. We can simply start them both (when they are independent) and then wait for them to complete, speeding up the overall process.

```cs
public static async Task<string> GetPersonDetails2()
{
    Task<int> idTask = GetIdAsync();
    Task<string> nameTask = GetNameAsync();
    await Task.WhenAll(idTask, nameTask); //wait for both to complete
    return $"Person Id:{idTask.Result}, Name:{nameTask.Result}";
}
```

#### Exception handling

As discussed in [Synchronous to asynchronous in .NET](/articles/sync-to-async-in-dotnet/), if a task fails to run successfully, it terminates the task and attaches the exception in the returned `Task`. This exception can be handled when doing a `task.Wait()` or `task.Result` in a synchronous style code.

In case of `async` code, we can simply wrap the `await task` call inside a `try-catch` block.

```cs
public async static void Execute()
{
    try
    {
        var message = await GetMessageAsync(); //.Result for sync
    }
    catch (Exception ex)
    {
        Console.WriteLine("Oh snap! " + ex.Message);
    }
}
```

Things are different if the async method (GetNameAsync here) is void. So, as discussed before, return a `Task` or `Task<T>` and avoid the void return.

> Async void methods have different error-handling semantics. When an exception is thrown out of an async Task or async Task&lt;T&ht; method, that exception is captured and placed on the Task object. With async void methods, there is no Task object, so any exceptions thrown out of an async void method will be raised directly on the SynchronizationContext that was active when the async void method started. - [MSDN](https://msdn.microsoft.com/en-us/magazine/jj991977.aspx)

#### Synchronous to asynchronous

The synchronous

```csharp
internal static int GetMessageLength()
{
    Console.WriteLine($"#1 Starting GetMessageLength Simple on thread {Thread.CurrentThread.ManagedThreadId}");
    var message = DoTimeTakingWork();
    DoIndependentWork();
    var length = message.Length; //does some work on result
    Console.WriteLine($"#1 GetMessageLength Simple completed on thread {Thread.CurrentThread.ManagedThreadId}");
    return length; //then returns the final result
}

internal static void DoIndependentWork()
{
    Console.WriteLine($"#3 Starting independent work on thread {Thread.CurrentThread.ManagedThreadId}");
    Thread.Sleep(2000); //works for 5 seconds
    Console.WriteLine($"#3 Independent work completed on thread {Thread.CurrentThread.ManagedThreadId}");
}

internal static string DoTimeTakingWork()
{
    Console.WriteLine($"#2 Started time taking work on thread {Thread.CurrentThread.ManagedThreadId}");
    Thread.Sleep(5000); //works for 5 seconds
    Console.WriteLine($"#2 Time taking work completed on thread {Thread.CurrentThread.ManagedThreadId}");
    return $"Current time : {DateTime.Now.ToLongDateString()}";
}
```

And the calling method, with timer & console writes

```csharp
//The calling method
Console.WriteLine($"STARTING PROGRAM on thread {Thread.CurrentThread.ManagedThreadId}");
var sw = new Stopwatch();
sw.Start();
var messageLength = GetMessageLength();
DoSomeOtherWork(); //independently works for 1000 milli seconds
Console.WriteLine($"TERMINATING PROGRAM on thread {Thread.CurrentThread.ManagedThreadId}");
sw.Stop();
Console.WriteLine($"Total Time taken = {sw.ElapsedMilliseconds} ms");
```

With `Task`

```csharp
internal static int GetMessageLength()
{
    Task<string> task = Task.Run(() => DoTimeTakingWork());
    DoIndependentWork();
    task.Wait();
    var message = task.Result; //the thread blocks here <<=
    var length = message.Length; //does some work on result
    return length; //then returns the final result
}
```

With `async-await`

```csharp
internal async static Task<int> GetMessageLengthAsync()
{
    var stringTask = DoTimeTakingWorkAsync();
    DoIndependentWork();
    var message = await stringTask; //awaits the result
    var length = message.Length; //does some work on awaited result
    return length; //then returns the final result
}

internal async static Task<string> DoTimeTakingWorkAsync()
{
    Console.WriteLine($"#2 Started time taking work on thread {Thread.CurrentThread.ManagedThreadId}");
    var messageTask = Task.Run(() =>
    {
        Thread.Sleep(5000); //works for 5 seconds
        Console.WriteLine($"#2 Time taking work completed on thread {Thread.CurrentThread.ManagedThreadId}");
    });
    await messageTask;
    return $"Current time : {DateTime.Now.ToLongDateString()}";
}
```

With `Task.ContinueWith`

```csharp
internal static Task<int> GetMessageLength()
{
    Task<string> messageTask = DoTimeTakingWorkAsync();
    var lengthTask = messageTask.ContinueWith((stringTask) =>
    {
        var message = stringTask.Result; //awaits the result
        var length = message.Length; //does some work on result
        return length; //then returns the final result
    });
    DoIndependentWork();
    return lengthTask; //it has to return, cannot resume work after awaited work is done
}
```

Remove individual console writes. Discuss about performance & threads on high level.

###### Other posts in the series - Tasks, Threads, Asynchronous

* **[Synchronous to asynchronous in .NET](/articles/sync-to-async-in-dotnet/)**
* **[Basic task cancellation demo in C#](/articles/task-cancellation/)**
* **[How does Async-Await work - Part I](/articles/async-await/)**
* **[How does Async-Await work - Part II](#)**
* **[Basic thread synchronization with examples - Part I](/articles/thread-synchronization-part-one/)**
* **[Basic thread synchronization with examples - Part II](/articles/thread-synchronization-part-two/)**